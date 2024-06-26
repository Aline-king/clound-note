# Kubernetes - 开源容器编排引擎

`Kubernetes` 是 Google 团队发起并维护的基于 Docker 的开源容器集群管理系统，它不仅支持常见的云平台，而且支持内部数据中心。

建于 Docker 之上的 `Kubernetes` 可以构建一个容器的调度服务，其目的是让用户透过 `Kubernetes` 集群来进行云端容器集群的管理，而无需用户进行复杂的设置工作。系统会自动选取合适的工作节点来执行具体的容器集群调度处理工作。其核心概念是 `Container Pod`。一个 `Pod` 由一组工作于同一物理工作节点的容器构成。这些组容器拥有相同的网络命名空间、IP以及存储配额，也可以根据实际情况对每一个 `Pod` 进行端口映射。此外，`Kubernetes` 工作节点会由主系统进行管理，节点包含了能够运行 Docker 容器所用到的服务。

k8s设计文档

[https://github.com/kubernetes/design-proposals-archive/tree/main](https://github.com/kubernetes/design-proposals-archive/tree/main)
