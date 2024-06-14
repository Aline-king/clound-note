# 部署使用

allocate-node-cidrs ：表示，每增加一个新的节点，都从cluster-cidr子网中切分一个新的子网网段分配给对应的节点上。

```yaml
[root@kubernetes-master1 ~]# cat /etc/kubernetes/manifests/kube-controller-manager.yaml
apiVersion: v1
...
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true                
    ...
    - --cluster-cidr=10.244.0.0/16
```

这些相关的网络状态属性信息，会经过 kube-apiserver 存储到etcd中。

{% code title="查看网段配置" %}
```bash
[root@kubernetes-master1 ~]# cat /run/flannel/subnet.env
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24   一般指的是flannel.d 隧道设备
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true 
```
{% endcode %}

1. 集群的 kube-controller-manager 负责分配每个节点的网段
2.  然后etcd 负责存储网络配置，并把信息分配给flannel

    集群的 flannel 负责各个节点的路由表定制及其数据包的拆分和封装

    所以flannel各个节点是平等的，仅负责数据平面的操作。

    网络功能相对来说比较简单。

    另外一种插件 calico相对于flannel来说，多了一个控制节点，来管控所有的网络节点的服务进程。
3. master节点负责获取信息 kubelet使用信息并启动

## 演示效果

```
[root@kubernetes-master1 ~]# kubectl get pod -n kube-flannel  -o wide
NAME                    READY STATUS    ... IP          NODE               ...
kube-flannel-ds-b6hxm   1/1   Running   ... 10.0.0.15   kubernetes-node1   ...
kube-flannel-ds-bx7rq   1/1   Running   ... 10.0.0.12   kubernetes-master1 ...
kube-flannel-ds-hqwrk   1/1   Running   ... 10.0.0.13   kubernetes-master2 ...
kube-flannel-ds-npcw6   1/1   Running   ... 10.0.0.17   kubernetes-node3   ...
kube-flannel-ds-sx427   1/1   Running   ... 10.0.0.14   kubernetes-master3 ...
kube-flannel-ds-v5f4p   1/1   Running   ... 10.0.0.16   kubernetes-node2   ...
```

flannel.1 后面的.1 就是 vxlan的网络标识。便于隧道正常通信

```
[root@kubernetes-master1 ~]# for i in {12..17};do ssh root@10.0.0.$i ifconfig | grep -A1 flannel ;done
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.0.0  netmask 255.255.255.255  broadcast 0.0.0.0
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.4.0  netmask 255.255.255.255  broadcast 0.0.0.0
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.5.0  netmask 255.255.255.255  broadcast 0.0.0.0
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.1.0  netmask 255.255.255.255  broadcast 0.0.0.0
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.2.0  netmask 255.255.255.255  broadcast 0.0.0.0
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.3.0  netmask 255.255.255.255  broadcast 0.0.0.0
```

```
路由效果
[root@kubernetes-master1 ~]# ip route list | grep flannel
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink
10.244.3.0/24 via 10.244.3.0 dev flannel.1 onlink
10.244.4.0/24 via 10.244.4.0 dev flannel.1 onlink
10.244.5.0/24 via 10.244.5.0 dev flannel.1 onlink
```
