# 域名服务DNS

## 场景需求

在传统的系统部署中，服务运行在一个固定的已知的 IP 和端口上，如果一个服务调用另外一个服务，可以通过地址直接调用。

在虚拟化或容器话的环境中，我们以k8s集群为例，如果存在个位数个svc我们可以很快找到对应的clusterip地址，进而找到指定的资源，虽然ip地址不容易记住，因为service在创建的时候会为每个clusterip分配一个名称，我们同样可以根据这个名称找到对应的服务。但是，如果我们的集群中有1000个Service，我们如何找到指定的service呢？

虽然我们可以借助于传统的<mark style="color:yellow;">**DNS机制**</mark>来实现，但在k8s集群中，服务实例的启动和销毁是很频繁，服务地址在动态的变化，所以传统的方式配置DNS解析记录就不太好实现了。

所以针对于这种场景，我们如果需要将请求发送到动态变化的服务实例上，可以通过两个步骤来实现：&#x20;

* <mark style="color:blue;">**服务注册**</mark> — 创建服务实例后，主动将当前服务实例的信息，存储到一个集中式的服务管理中心。&#x20;
* <mark style="color:blue;">**服务发现**</mark> — 当A服务需要找未知的B服务时，先去服务管理中心查找B服务地址，然后根据该地址找到B服务

## DNS方案

kubeDNS自从k8s诞生以来，其方案的具体实现样式前后经历了三代，[coreDNS](../gai-shu/k8s-zu-jian/core-dns/)(现在默认）

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 域名解析记录

Kubelet会为每一个容器在 /etc 中创建配置文件resolv.conf，并生成DNS查询客户端依赖到的必要配置，相关的配置信息源自于kubelet的配置参数，

容器的DNS服务器由clusterDNS参数的值设定，它的取值为kube-system名称空间中的Service对象kube-dns的ClusterIP，默认为10.96.0.10.&#x20;

DNS搜索域的值由clusterDomain参数的值设定，若部署Kubernetes集群时未特别指定，其值将为cluster.local、svc.cluster.local和NAMESPACENAME.svc.cluster.local

### 查看dns

以K8s 1.23.8为例，环境初始化配置文件中与dns相关的检索信息

```bash
[root@kubernetes-master1 ~]# grep -A1 networking /data/kubernetes/cluster_init/kubeadm_init_1.23.8.yml
networking:
  dnsDomain: cluster.local
```

以kubectl的方式查询

资源对象的查看dns的后缀主要有四种：&#x20;

default.svc.cluster.local&#x20;

svc.cluster.local&#x20;

cluster.local&#x20;

localhost

```
查看pod内部的resolv.conf文件
[root@kubernetes-master1 ~]# kubectl  exec -it nginx-6944855df5-8zjdn -- cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local localhost
```

{% hint style="info" %}
dns记录具有标准的名称格式

<mark style="color:yellow;">**资源对象名.命名空间名.svc.cluster.local**</mark>
{% endhint %}

