# 令牌认证



默认情况下，kubeadm在创建集群的时候，会使用tls的方式传输信息，所有集群node节点在加入集群的时候，也会应用该信息 -- 即token，

默认情况下，该token是有存活时间的，也就是说，当token时间过期后，我们就无法使用相同的kubeadm join命令将新的节点加入到集群了。

{% tabs %}
{% tab title="查看token存活时间" %}
```
查看集群初始化时候设定的token存活时间
[root@kubernetes-master1 ~]# grep ttl /data/kubernetes/cluster_init/kubeadm_init_1.23.8.yml
  ttl: 96h0m0s
```


{% endtab %}

{% tab title="在token没有过期的时查看集群的token列表" %}
```

[root@kubernetes-master1 ~]# kubeadm token list
TOKEN                     TTL     EXPIRES                USAGES                   DESCRIPTION    EXTRA GROUPS
abcdef.0123456789abcdef   2d      2062-07-28T11:25:18Z   authentication,signing   <none>     system:bootstrappers:kubeadm:default-node-token
```


{% endtab %}

{% tab title="过期token演示" %}
token过期后的效果演示&#x20;

\[root@kubernetes-master1 \~]# kubeadm token list

会返回空的结果
{% endtab %}
{% endtabs %}

## 生成token实践

<details>

<summary>准备新节点环境</summary>

```
从当前集群中移除 kubernetes-node3环境
[root@kubernetes-master1 ~]# kubectl  delete node kubernetes-node3
node "kubernetes-node3" deleted
​
确认效果
[root@kubernetes-master1 ~]# kubectl get nodes
NAME                 STATUS   ROLES                  AGE    VERSION
kubernetes-master1   Ready    control-plane,master   4d2h   v1.23.8
kubernetes-master2   Ready    control-plane,master   4d2h   v1.23.8
kubernetes-master3   Ready    control-plane,master   4d2h   v1.23.8
kubernetes-node1     Ready    <none>                 4d2h   v1.23.8
kubernetes-node2     Ready    <none>                 4d2h   v1.23.8
```

```
kubernetes-node3环境清空所有集群环境
[root@kubernetes-node3 ~]# kubeadm reset
[root@kubernetes-node3 ~]# rm -f /etc/cni/net.d/*
[root@kubernetes-node3 ~]# reboot
```



</details>

<details>

<summary>生成令牌</summary>

查看历史token的列表

发现没有token历史记录

```
查看当前的token信息
[root@kubernetes-master1 ~]# kubeadm token list
[root@kubernetes-master1 ~]#
```

```
生成token方法
[root@kubernetes-master1 ~]# kubeadm token create
o4hkeo.yfs5qr4ashdjeic6
​
查看效果 只有24小时的有效期
[root@kubernetes-master1 ~]# kubeadm token list
TOKEN                     TTL     EXPIRES                USAGES                   DESCRIPTION    EXTRA GROUPS
o4hkeo.yfs5qr4ashdjeic6   23h     2062-07-29T14:28:02Z   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
```

这个token就是我们为新节点加入在集群生成的内容。

```
获取ca证书sha256编码hash值
[root@kubernetes-master1 ~]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
d8a77a69fb0b54cd72a692be83fdcd2c39f203bdc6e729b9d7d63cca3030cfcc
        
生成添加结点命令
[root@kubernetes-node3 ~]# kubeadm join 10.0.0.200:6443 --token o4hkeo.yfs5qr4ashdjeic6 --discovery-token-ca-cert-hash sha256:d8a77a69fb0b54cd72a692be83fdcd2c39f203bdc6e729b9d7d63cca3030cfcc
​
主节点查看效果：
[root@kubernetes-master1 ~]# kubectl get nodes
NAME                 STATUS   ROLES                  AGE    VERSION
kubernetes-master1   Ready    control-plane,master   4d3h   v1.23.8
kubernetes-master2   Ready    control-plane,master   4d3h   v1.23.8
kubernetes-master3   Ready    control-plane,master   4d3h   v1.23.8
kubernetes-node1     Ready    <none>                 4d3h   v1.23.8
kubernetes-node2     Ready    <none>                 4d3h   v1.23.8
kubernetes-node3     Ready    <none>                 39s    v1.23.8
```

新的节点已经添加成功了\


使用 --print-join-command 方法可以更快的输出完整的新阶段添加到集群的命令。

```
精简方法
[root@kubernetes-master1 ~]# kubeadm token create --print-join-command
kubeadm join 10.0.0.200:6443 --token e36qre.9nhy8a3qofq2lgaa --discovery-token-ca-cert-hash sha256:d8a77a69fb0b54cd72a692be83fdcd2c39f203bdc6e729b9d7d63cca3030cfcc
```

</details>



基于认证用户的唯一令牌来进行认证，有默认的超时机制，一会儿就失效了

{% tabs %}
{% tab title="集群级别" %}
11

```
资源定义文件方式 
# 04_kubernetes_secure_dashboard_cluster.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dashboard-admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kube-system
​
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    
创建资源对象
# kubectl  apply -f 04_kubernetes_secure_dashboard_cluster.yaml
clusterrolebinding.rbac.authorization.k8s.io/dashboard-admin created
serviceaccount/dashboard-admin created
```

复制token到浏览器查看效果

```
获取token信息
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
Name:         dashboard-admin-token-4cdrd
...
token:      eyJhbG...qU3__b9ITbLHEytrA
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="用户级别" %}
11

```
资源定义文件方式 
[root@kubernetes-master1 /data/kubernetes/secure]# 05_kubernetes_secure_dashboard_namespace.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-ns
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dashboard-ns
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: dashboard-ns
  namespace: default
```

11

```
创建资源对象
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  apply -f 05_kubernetes_secure_dashboard_namespace.yaml serviceaccount/dashboard-ns created
rolebinding.rbac.authorization.k8s.io/dashboard-ns created
```

复制token到浏览器查看效果

```
获取token信息
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl describe secrets $(kubectl get secret | awk '/dashboard-ns/{print $1}')
Name:         dashboard-ns-token-btq6w
...
token:       eyJhbG...qU3__8Kqc0Q
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

