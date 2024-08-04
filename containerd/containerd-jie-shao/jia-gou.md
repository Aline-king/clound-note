# 架构

## C/S 架构

Containerd 采用标准的 C/S 架构：我们提到的 containerd 主要为 Server 端实现

* 服务端通过 GRPC 协议提供稳定的 API；
* 客户端通过调用服务端的 API 进行高级的操作。

## 组件

为了实现解耦，Containerd 将不同的职责划分给不同的组件，每个组件就相当于一个子系统（subsystem）。连接不同子系统的组件被称为模块。

Containerd 被分为三个大块： Storage 、 Metadata 和 Runtime。

Containerd 两大子系统为：&#x20;

1. <mark style="color:orange;">**Bundle**</mark> : 在 Containerd 中，Bundle 包含了配置、元数据和根文件系统数据，你可以理解为 容器的文件系统。而 Bundle 子系统允许用户从镜像中提取和打包 Bundles。&#x20;
2. <mark style="color:orange;">**Runtime**</mark> : Runtime 子系统用来执行 Bundles，比如创建容器。其中，每一个子系统的行为都由一个或多个模块协作完成（架构图中的 Core 部分）。

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

containerd的实现目的 是为了直接嵌入k8s使用，是一个工业级的容器运行时，不直接提供开发人员和终端用户来使用，避免与docker竞争

### 架构缩略图

containerd GRPC API、Service、 MetaData 为典型的 API 层、逻辑层、数据层架构。

* GRPC API 层为暴露服务的接口层，
* Service 层为具体的逻辑处理层，
* Metadata 为数据层，Service、 MetaData 两层为 Core 层。

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Containerd 被分为三个大块：`Storage`、`Metadata` 和 `Runtime`

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### MetaData

containerd 中的各个微服务插件的元数据都保存在 metadata 中，存储采用了 `boltdb` 开源数据库。

#### `boltdb`&#x20;

> `boltdb` 是基于 Golang 实现的 KV 数据库。`boltdb` 提供最基本的存储功能，并不支持网络连接，不支持复杂的SQL查询。单个数据库数据存储在单个文件里，通过API的方式对数据文件读写，达到数据持久化的效果。bolt 通过嵌入在程序中进行使用，我们熟知的 ETCD 就是基于 `boltdb` 实现的。

containerd 通过 boltdb 对相关对象的的元数据进行存储，如 snapshots、image、container 等，同时 containerd 对 metadata 中的数据还会定期执行垃圾收集，用于自动清理过期不使用的资源。

containerd 中的对象在 `boltdb` 中保存格式如下。

{% tabs %}
{% tab title="First Tab" %}

{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}

{% tab title="Untitled" %}

{% endtab %}
{% endtabs %}
