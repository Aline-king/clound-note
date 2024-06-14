# 实践

在启动服务时候，还加载了kubelet.service.d目录下的10-kubeadm.conf文件的内容

{% tabs %}
{% tab title="First Tab" %}
{% code title="软件组成" %}
```bash
[root@kubernetes-master1 ~]# rpm -ql kubelet
/etc/kubernetes/manifests
/etc/sysconfig/kubelet
/usr/bin/kubelet
/usr/lib/systemd/system/kubelet.service
```
{% endcode %}

```bash
服务状态
​[root@kubernetes-master1 ~]# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
     Active: active (running)
   ...
```
{% endtab %}

{% tab title="Second Tab" %}
结果显示：

&#x20;kubelet的配置参数主要是在/var/lib/kubelet/config.yaml文件中定义

```
查看加载文件内容
[root@kubernetes-master1 ~]# cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
...
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
...
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
...
```

结果显示：&#x20;

该文件里面设置了大量的配置参数，其中有一项关键的就是staticPodPath。其实我们还可以直接 在/etc/kubernetes/kubelet.conf文件中使用 --pod-manifest-path=的方式指定静态pod的路径。

```
kubelet配置参数
[root@kubernetes-master1 ~]# grep static /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```


{% endtab %}

{% tab title="Untitled" %}
根据我们的查看资料，对于kubeadm的环境来说，他默认情况下已经配置好了静态pod的专用目录，就是/etc/kubernetes/manifests，而我们没有看到静态pod的效果，主要是该目录中并没有任何资源对象文件。

```
master节点查看效果
[root@kubernetes-master1 ~]# ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
​
node点查看效果
[root@kubernetes-node1 ~]# ls /etc/kubernetes/manifests/
[root@kubernetes-node1 ~]#
```
{% endtab %}
{% endtabs %}

## 静态pod实践

```
master查看集群当前效果
[root@kubernetes-master1 ~]# kubectl  get pod
No resources found in default namespace.
​
查看特制的静态资源文件
[root@kubernetes-master1 /data/kubernetes/pod]# cat 05_kubernetes_pod_static.yaml
apiVersion: v1
kind: Pod
metadata:
  name: superopsmsb-static-pod
spec:
  containers:
    - name: nginx
      image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
```

负载工作节点的静态pod目录，只要配置文件发生变化，立刻都会显示出来

```
传输到指定的node节点
[root@kubernetes-master1 /data/kubernetes/pod]# scp 05_kubernetes_pod_static.yaml root@10.0.0.16:/etc/kubernetes/manifests/
05_kubernetes_pod_static.yaml                                                    100%  180   148.6KB/s   00:00
​
master节点查看效果
[root@kubernetes-master1 /data/kubernetes/pod]# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
superopsmsb-static-pod-kubernetes-node2   1/1     Running   0          12s   10.244.4.7   kubernetes-node2   <none>           <none>
```

由于静态pod的增删主要是基于node节点上的kubelet来实现的，所以master无法完全删除。

```
master节点尝试删除静态pod
[root@kubernetes-master1 /data/kubernetes/pod]# kubectl  delete pod superopsmsb-static-pod-kubernetes-node2
pod "superopsmsb-static-pod-kubernetes-node2" deleted
[root@kubernetes-master1 /data/kubernetes/pod]# kubectl  get pod
NAME                                      READY   STATUS    RESTARTS   AGE
superopsmsb-static-pod-kubernetes-node2   1/1     Running   0          3s
```

静态pod不受kubectl的编辑管理

```
尝试编辑pod对象
[root@kubernetes-master1 /data/kubernetes/pod]# kubectl  set image pod superopsmsb-static-pod-kubernetes-node2 nginx=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.16.0
pod/superopsmsb-static-pod-kubernetes-node2 image updated
[root@kubernetes-master1 /data/kubernetes/pod]# kubectl  get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE     IP           NODE               NOMINATED NODE   READINESS GATES
superopsmsb-static-pod-kubernetes-node2   1/1     Running   0          2m27s   10.244.4.7   kubernetes-node2   <none>           <none>
[root@kubernetes-master1 /data/kubernetes/pod]# curl 10.244.4.7
Hello Nginx, superopsmsb-static-pod-kubernetes-node2-1.23.0
```

移除yaml文件就行

```
移除静态Pod资源
[root@kubernetes-node2 ~]# ls /etc/kubernetes/manifests/
05_kubernetes_pod_static.yaml
[root@kubernetes-node2 ~]# mv /etc/kubernetes/manifests/05_kubernetes_pod_static.yaml  ./
[root@kubernetes-node2 ~]# ls /etc/kubernetes/manifests/
[root@kubernetes-node2 ~]#
​
回到master结点上，查看效果
[root@kubernetes-master1 /data/kubernetes/pod]# kubectl get pod -o wide
No resources found in default namespace.
```
