# 容器发展史

说到Containerd，就必然绕不开Docker。 Docker作为一个完整的容器引擎，其包含三个部分： 计算、存储、网络。在早期的时候，这些功能统一由docker daemon进程提供； 但从docker 1.11版本开始，这些功功能开始做拆分：

1. 计算：由containerd提供&#x20;
2. 存储：由docker-volume提供&#x20;
3. 网络：由docker-network提供

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

当创建容器的请求到达docker api，docker-api会调用containerd执行创建操作，此时containerd会启动一个containerd-shim进程，containerd-shim调用runc执行容器的创建操作。当容器创建完成之后， runc退出，containerd-shim作为容器的父进程收集容器的运行状态，将其上报给containerd，并在容器中 pid 为 1 的进程退出后接管容器中的子进程进行清理, 确保不会出现僵尸进程。

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> runc是OCI的一个具体实现。 OCI（open container initivtive）全称为开放容器标准，其主要用于规范容器镜像的结构以及容器需要接收的操作指令，如create、start、stop、delete等命令。除了runc之外，Kata、gVisor也符合OCI规范。

为什么containerd不直接调用runc，而要启动一个containerd-shim调用runc？因为容器进程是需要一个父进程来做状态收集、维持 stdin 等 fd 打开等工作的，假如这个父进程就是 containerd，那如果 containerd 挂掉的话，整个宿主机上所有的容器都会退出。引入containerd-shim则可解决这个问题。

**所以事实上在早期containerd一直只是作为docker创建容器的子组件而存在。**

## CRI - container runtime interface

在kubernetes早期的时候，由于没法跟docker正面刚，只得通过硬编码的方式在kubelet当中直接调用Docker API创建容器。内置docker-shim

随着容器生态的逐渐发展，市面上出现了更多的容器运行时。 此时kubernetes为了支持更多的容器运行时，Google就和红帽主导了CRI标准，用于将kubernetes平台和特定的容器运行时解耦。&#x20;

{% hint style="info" %}
CRI（Container Runtime Interface 容器运行时接口） 本质上就是 Kubernetes 定义的一组与容器运行时进行交互的接口，所以只要实现了这套接口的容器运行时都可以对接Kubernetes 。

不过 Kubernetes 推出 CRI 这套标准的时候还没有现在的统治地位，所以有一些容器运行时自身没有实现 CRI 接口，于是就有了 shim（垫片）。

一个 shim 的职责就是作为适配器将各种容器运行时本身的接口适配到 Kubernetes 的 CRI 接口上，其中 dockershim 就是 Kubernetes 对接 Docker 到 CRI 接口上的一个垫片实现。
{% endhint %}

Kubelet 通过 gRPC 框架与容器运行时或 shim 进行通信，其中 kubelet 作为客户端，CRI shim（也可能是容器运行时本身）作为服务器。

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

由于当时的Docker仍然处于容器生态的统治地位。kubernetes不得不在kubelet当中内置了docker-shim：

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

再后来，Docker在容器生态大战中败北。在2017年的时候，Docker公司将其容器运行时Containerd捐献给了CNCF。为了将containerd接入到CRI标准中，k8s又搞出了cri-containerd项目，cri-containerd是一个守护进程用来实现kubelet和containerd之间的交互，此时k8s节点上kubelet启动容器就变成了下面这个样子：

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

再往后，在Containerd 1.1时，将cri-containerd改成了Containerd的CRI插件，CRI插件位于containerd内部，这让k8s启动Pod时的通信更加高效：

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

与此同时 Kubernetes 社区也做了一个专门用于 Kubernetes 的 CRI 运行时 CRI-O，直接兼容 CRI 和 OCI 规范：

> CRI
>
> OCI：Open Container Initiative (OCI) 开放容器倡议，是一个由科技公司组成的团体，其目的是围绕容器镜像和运行时创建开放的行业标准。他们维护容器镜像格式的规范，以及容器应该如何运行。

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

无论是CRI-O还是containerd接入kubernetes的方式都比docker使用dockershim的方式接入kubernetes要来的简单。

> 在2020年12月份，kubernetes宣布将在1.22版本当中正式从kubelet当中移除docker-shim代码。也就是说，如果docker仍然不支持CRI，则kubernetes将会放弃对docker的支持。而在2021年8月，kubernetes正式发布了1.22版本。
