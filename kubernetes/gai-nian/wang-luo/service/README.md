# service

它定义了一组Pod的逻辑集合和一个用于访问它们的策略，它可以基于标签的方式自动找到对应的pod应用，而无需关心pod的ip地址变化与否，从而实现了类似负载均衡的效果.&#x20;

这个资源在master端的Controller组件中，由Service Controller 来进行统一管理。

## service在K8s中的关系

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Kubernetes 的 Service定义了一个服务的访问入口地址，前端的应用Pod通过Service访问其背后一组有Pod副本组成的集群示例，

Service通过Label Selector访问指定的后端Pod，RC保证Service的服务能力和服务质量处于预期状态。

## 工作模型

本质上就是工作节点的一些iptables或ipvs规则，这些规则由<mark style="color:orange;">**kube-proxy**</mark>进行实时维护，

站在kubernetes的发展脉络上来说，kube-proxy将请求代理至相应端点的方式有三种：userspace/iptables/ipvs。目前我们主要用的是 iptables/ipvs 两种。

## 详细解释

Service是Kubernetes中最高一级的抽象资源对象，每个Service提供一个独立的服务，集群Service彼此间使用TCP/IP进行通信，将不同的服务组合在一起运行起来，就行了我们所谓的"系统"，效果如下图

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Pod入口

```
    我们知道每个Pod都有一个专用的IP地址，加上Pod内部容器的Port端口，就组成了一个访问Pod专用的EndPoint(Pod IP+Container Port)，从而实现了用户外部资源访问Pod内部应用的效果。这个EndPoint资源在master端的Controller组件中，由EndPoint Controller 来进行统一管理。
```

kube-proxy

```
    Pod是工作在不同的Node节点上，而Node节点上有一个kube-proxy组件，它本身就是一个软件负载均衡器，在内部有一套专有的负载均衡与会话保持机制，可以达到，接收到所有对Service请求，进而转发到后端的某个具体的Pod实例上，相应该请求。
    -- kube-proxy 其实就是 Service Controller位于各节点上的agent。
```

service表现

```
    Kubernetes给Service分配一个全局唯一的虚拟ip地址--cluster IP，它不存在任何网络设备上，Service通过内容的标签选择器，指定相应该Service的Pod资源，这样以来，请求发给cluster IP，后端的Pod资源收到请求后，就会响应请求。
​
这种情况下，每个Service都有一个全局唯一通信地址，整个系统的内部服务间调用就变成了最基础的TCP/IP网络通信问题。如果我们的集群内部的服务想要和外部的网络进行通信，方法很多，比如：
    NodePort类型，通过在所有结点上增加一个对外的端口，用于接入集群外部请求
    ingress类型，通过集群附加服务功能，将外部的域名流量转交到集群内部。
```

service vs endpoint

```
1 当创建 Service资源的时候，最重要的就是为Service指定能够提供服务的标签选择器，
2 Service Controller就会根据标签选择器创建一个同名的Endpoint资源对象。
3 Endpoint Controller开始介入，使用Endpoint的标签选择器(继承自Service标签选择器)，筛选符合条件的pod资源
4 Endpoint Controller 将符合要求的pod资源绑定到Endpoint上，并告知给Service资源，谁可以正常提供服务。
5 Service 根据自身的cluster IP向外提供由Endpoint提供的服务资源。
​
-- 所以Service 其实就是 为动态的一组pod资源对象 提供一个固定的访问入口。
```

