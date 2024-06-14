# 网络管理

K8s的网络模型如下图所示

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

主机网络 -> 本地网络&#x20;

服务网络 -> service网络

应用网络 -> pod网络



## 容器A和容器B在同一个pod里

它会在每个 Pod 里，额外起一个 Infra container 小容器来共享整个 Pod 的 Network Namespace。

> Infra container 是一个非常小的镜像，大概 700KB 左右，是一个C语言写的、永远处于“暂停”状态的容器 ，[官方的pause容器代码](https://github.com/kubernetes/kubernetes/tree/master/build/pause)

由于有了这样一个 Infra container 之后，其他所有容器都会通过 Join Namespace 的方式加入到 Infra container 的 Network Namespace 中。

## Infra container

由于需要有一个相当于说中间的容器存在，所以整个 Pod 里面，必然是 Infra container 第一个启动

并且整个 Pod 的生命周期是等同于 Infra container 的生命周期的，与容器 A 和 B 是无关的

这也是为什么在 Kubernetes 里面，它是允许去单独更新 Pod 里的某一个镜像的，即：做这个操作，整个 Pod 不会重建，也不会重启，这是非常重要的一个设计。

pause容器的示意图

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>
