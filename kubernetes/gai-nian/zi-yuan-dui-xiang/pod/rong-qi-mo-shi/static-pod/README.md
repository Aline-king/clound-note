# static pod



{% hint style="info" %}
对于pod来说，基于控制的特性主要有两类：静态pod和普通pod。

> 普通pod其实就是我们一直实践中用到的pod，也就是可以直接被集群中的kubelet进行增删改查的pod，
>
> 静态pod在本质上与动态pod没有区别，只是在于静态pod只能在特定的节点上运行，由特定节点上的kubelet进程来管理，对于集群kubectl来说，只能看，不能乱动。&#x20;
{% endhint %}

静态pod常常用于某些结点上的一些隐私操作或者核心操作，而且这些操作往往受到特定结点的场景约束，比如某些定向监控、特定操作。

## 实现方式

> [https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/static-pod/](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/static-pod/)

配置文件方式

其实就是在特定的目录下存放我们定制好的资源对象文件，然后结点上的kubelet服务周期性的检查该目录下的所有内容，对静态pod进行增删改查。

1 定制kubelet服务定期检查配置目录&#x20;

2 增删定制资源文件

