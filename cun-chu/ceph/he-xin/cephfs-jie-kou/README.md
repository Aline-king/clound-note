# CephFS接口

<mark style="color:purple;">**场景**</mark>

有了 RBD，远程磁盘挂载的问题就解决了，但 RBD 的问题是不能多个主机共享一个磁盘，如果有一份数据很多客户端都要读写该怎么办呢？这时 CephFS 作为文件系统存储解决方案就派上用场了。

## 简介

Ceph 文件系统（Ceph FS）是个 POSIX 兼容的文件系统，它直接使用 Ceph 存储集群来存储数据。

Ceph 文件系统与 Ceph 块设备、同时提供 S3 和 Swift API 的 Ceph 对象存储、或者原生库（ librados ）的实现机制稍显不同。

## 原理

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

CephFS需要至少运行一个元数据服务器（MDS）守护进程（ceph-mds），此进程管理与CephFS上存储的文件相关的元数据，并协调对Ceph存储集群的访问。&#x20;

因此，若要使用CephFS接口，需要在存储集群中至少部署一个MDS实例。

MDS虽然称为元数据服务，但是它却不存储任何元数据信息，它存在的目的仅仅是 让我们rados集群提供的存储接口能够兼容POSIX 的文件系统接口&#x20;

简单来说，它是真正元数据信息的索引服务。 客户端在访问文件接口的时候，首先连接到 MDS上，在MDS的内存里面维持元数据的索引信息，这些信息真正的保存在RADOS集群的metadata pool上面，而对应的数据，保存在对应的 data pool上面。 而且以将元数据服务作为一个非状态的效果，便于扩展。

相较于NFS来说，它主要有以下特点优势：

1. 底层数据冗余的功能，底层的roados 提供了基本的数据冗余功能，因此不存在nfs的单点故障因素。
2. 底层的roados系统有n个存储节点组成，所以数据的存储可以分散IO，吞吐量较高，ceph提供的扩展性要想当的高。&#x20;
