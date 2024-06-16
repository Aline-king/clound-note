# 原理

<figure><img src="../../../../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Felix 每个节点都有，负责配置路由、ACL、向etcd宣告状态等

BIRD 每个节点都有，负责把 Felix 写入Kernel的路由信息 分发到整个 Calico网络，确保 workload 连通

etcd 存储calico自己的状态数据，可以结合kube-apiserver来工作

官方推荐；

< 50节点,可以结合 kube-apiserver 来实现数据的存储

\> 50节点,推荐使用独立的ETCD集群来进行处理。

参考[ 链接](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico)

Route Reflector 路由反射器，用于集中式的动态生成所有主机的路由表，非必须选项

超过100个节点推荐使用：[链接](https://projectcalico.docs.tigera.io/getting-started/kubernetes/rancher#concepts)

Calico编排系统插件 实现更广范围的虚拟网络解决方案。

<figure><img src="../../../../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>
