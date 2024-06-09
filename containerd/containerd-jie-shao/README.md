# containerd介绍

### 轻量级容器管理工具

Kubernetes v1.24 之前的版本直接集成了 Docker Engine 的一个组件，名为 dockershim \[用于调用Docker]。 这种特殊的直接整合不再是 Kubernetes 的一部分 （这次删除被作为 v1.20 发行版本的一部分宣布）。 这意味Kubernetes从版本1.24开始就弃用Docker作为容器运行时，取而代之的是更加轻量级的Containerd。

containerd可用作 Linux 和 Windows 的守护进程。它管理其主机系统的完整容器生命周期，从图像传输和存储到容器执行和监督，再到低级存储到网络附件等等。

## 概念

早在2016年3月，Docker 1.11 的Docker Engine里就包含了containerd，而现在则是把containerd从Docker Engine里彻底剥离出来，作为一个独立的开源项目独立发展，目标是提供一个更加开放、稳定的容器运行基础设施。

和原先包含在Docker Engine里containerd相比，独立的containerd将具有更多的功能，可以涵盖整个容器运行时管理的所有需求。另外独立之后containerd的特性演进可以和Docker Engine分开，专注容器运行时管理，可以更稳定。

Containerd是一个工业标准的<mark style="color:purple;">**容器运行时**</mark>，重点是它简洁，健壮，便携，在Linux和window上可以作为一个守护进程运行，它可以管理主机系统上容器的完整的生命周期：镜像传输和存储，容器的执行和监控，低级别的存储和网络。

每个containerd只负责一台机器，Pull镜像，对容器的操作（启动、停止等），网络，存储都是由containerd完成。具体运行容器由runC负责，实际上只要是符合OCI规范的容器都可以支持。

Containerd和docker不同，containerd重点是集成在大规模的系统中，例如kubernetes、Swarm、Mesos等【对于容器编排服务来说，运行时只需要使用containerd+runC，更加轻量，容易管理。】。Containerd 被设计成嵌入到一个更大的系统中，而不是直接由开发人员或终端用户使用。
