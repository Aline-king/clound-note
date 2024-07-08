# 集群升级

{% tabs %}
{% tab title="master更新原则 " %}
将待更新节点从高可用集群的反向代理中剔除&#x20;

1. 更新指定的集群环境软件&#x20;
2. 确认更新计划&#x20;
3. 执行更新计划&#x20;
4. 将更新后的节点加入到高可用集群&#x20;
5. 对于其他节点执行1-5步骤
{% endtab %}

{% tab title="node更新原则" %}
1. 冻结待删除节点&#x20;
2. 驱离待删除节点资源&#x20;
3. 确认更新计划&#x20;
4. 执行更新计划&#x20;
5. 删除驱离和冻结动作&#x20;
6. 对于其他节点执行1-5步骤
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="剔除待更新节点" %}
```
所有的高可用节点更新haproxy配置
[root@kubernetes-ha1 ~]# vim /etc/haproxy/haproxy.cfg
...
listen kubernetes-master-6443
        bind 10.0.0.200:6443
        mode tcp
        # server kubernetes-master1 10.0.0.12:6443 check inter 3s fall 3 rise 5
        server kubernetes-master2 10.0.0.13:6443 check inter 3s fall 3 rise 5
        server kubernetes-master3 10.0.0.14:6443 check inter 3s fall 3 rise 5
```

111

```bash
重启服务
[root@kubernetes-ha1 ~]# systemctl restart haproxy.service
```

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>


{% endtab %}

{% tab title="更新软件" %}
111

```
安装新版本软件
[root@kubernetes-master1 ~]# yum install -y kubeadm-1.23.9-0 kubectl-1.23.9-0 kubelet-1.23.9-0
​
检查效果
[root@kubernetes-master1 ~]# kubectl version 
[root@kubernetes-master1 ~]# kubeadm version
[root@kubernetes-master1 ~]# kubelet --version
注意：
    kubectl 会同时出现两个版本
```


{% endtab %}

{% tab title="确认更新计划" %}
222

```
查看更新条件
[root@kubernetes-master1 ~]# kubeadm upgrade plan
...
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.23.8
[upgrade/versions] kubeadm version: v1.23.9
I0728 23:14:42.949711   11880 version.go:255] remote version is much newer: v1.24.3; falling back to: stable-1.23
[upgrade/versions] Target version: v1.23.9
[upgrade/versions] Latest version in the v1.23 series: v1.23.9
​
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     6 x v1.23.8   v1.23.9
​
Upgrade to the latest version in the v1.23 series:
​
COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.23.8   v1.23.9
kube-controller-manager   v1.23.8   v1.23.9
kube-scheduler            v1.23.8   v1.23.9
kube-proxy                v1.23.8   v1.23.9
CoreDNS                   v1.8.6    v1.8.6
etcd                      3.5.1-0   3.5.1-0
​
You can now apply the upgrade by executing the following command:
​
        kubeadm upgrade apply v1.23.9
​
_____________________________________________________________________
​
​
The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.
​
API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```


{% endtab %}

{% tab title="执行更新计划" %}
222

```
根据提示命令，更新到指定的软件版本
[root@kubernetes-master1 ~]# kubeadm upgrade apply v1.23.9
...
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.23.9". Enjoy!
...
​
kubernetes集群软件版本更新，kubelet软件文件变动，需要重载后才能重启
[root@kubernetes-master3 ~]# systemctl daemon-reload
[root@kubernetes-master3 ~]# systemctl restart docker kubelet
​
确认效果
[root@kubernetes-master1 ~]# kubectl get nodes
NAME                 STATUS   ROLES                  AGE    VERSION
kubernetes-master1   Ready    control-plane,master   4d3h   v1.23.9
kubernetes-master2   Ready    control-plane,master   4d3h   v1.23.8
kubernetes-master3   Ready    control-plane,master   4d3h   v1.23.8
kubernetes-node1     Ready    <none>                 4d3h   v1.23.8
kubernetes-node2     Ready    <none>                 4d3h   v1.23.8
kubernetes-node4     Ready    <none>                 20m    v1.23.8
```


{% endtab %}

{% tab title="加入集群" %}
将更新后的节点加入到高可用集群

```
更新haproxy配置
[root@kubernetes-ha1 ~]# vim /etc/haproxy/haproxy.cfg
...
listen kubernetes-master-6443
        bind 10.0.0.200:6443
        mode tcp
        server kubernetes-master1 10.0.0.12:6443 check inter 3s fall 3 rise 5
        # server kubernetes-master2 10.0.0.13:6443 check inter 3s fall 3 rise 5
        server kubernetes-master3 10.0.0.14:6443 check inter 3s fall 3 rise 5
```

```
重启服务
[root@kubernetes-ha1 ~]# systemctl restart haproxy.service
```

反复执行1-5步骤

```
针对其他所有节点更新后的集群最终效果  
[root@kubernetes-master3 ~]# kubectl get nodes
NAME                 STATUS   ROLES                  AGE    VERSION
kubernetes-master1   Ready    control-plane,master   4d4h   v1.23.9
kubernetes-master2   Ready    control-plane,master   4d4h   v1.23.9
kubernetes-master3   Ready    control-plane,master   4d4h   v1.23.9
kubernetes-node1     Ready    <none>                 4d4h   v1.23.8
kubernetes-node2     Ready    <none>                 4d4h   v1.23.8
kubernetes-node4     Ready    <none>                 37m    v1.23.8
```


{% endtab %}
{% endtabs %}

