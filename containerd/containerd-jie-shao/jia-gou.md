# 架构

## C/S 架构

Containerd 采用标准的 C/S 架构：

* 服务端通过 GRPC 协议提供稳定的 API；
* 客户端通过调用服务端的 API 进行高级的操作。

## 组件

为了实现解耦，Containerd 将不同的职责划分给不同的组件，每个组件就相当于一个子系统（subsystem）。连接不同子系统的组件被称为模块。

Containerd 被分为三个大块： Storage 、 Metadata 和 Runtime。

Containerd 两大子系统为：&#x20;

1. <mark style="color:orange;">**Bundle**</mark> : 在 Containerd 中，Bundle 包含了配置、元数据和根文件系统数据，你可以理解为 容器的文件系统。而 Bundle 子系统允许用户从镜像中提取和打包 Bundles。&#x20;
2. <mark style="color:orange;">**Runtime**</mark> : Runtime 子系统用来执行 Bundles，比如创建容器。其中，每一个子系统的行为都由一个或多个模块协作完成（架构图中的 Core 部分）。

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

containerd的实现目的 是为了直接嵌入k8s使用，是一个工业级的容器运行时，不直接提供开发人员和终端用户来使用，避免与docker竞争

### 架构缩略图

Containerd 被分为三个大块：`Storage`、`Metadata` 和 `Runtime`

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
