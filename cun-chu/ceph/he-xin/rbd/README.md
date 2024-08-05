# RBD

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Ceph块设备，也称为RADOS块设备（简称RBD），是一种基于RADOS存储系统支持超配（thin-provisioned）、可伸缩的条带化数据存储系统，它通过librbd库与OSD进行交互。

RBD为KVM等虚拟化技术和云OS（如OpenStack和CloudStack）提供高性能和无限可扩展性的存储后端，这些系统依赖于libvirt和QEMU实用程序与RBD进行集成。

## 介绍

RBD，全称RADOS Block Devices，是一种建构在RADOS存储集群之上为客户端提供块设备接口的存储服务中间层.

这类的客户端包括虚拟化程序KVM（结合qemu）和云的计算操作系统OpenStack和CloudStack等

RBD基于RADOS存储集群中的多个OSD进行**条带化**，支持存储空间的**简配**（thinprovisioning）和**动态扩容**等特性，并能够借助于RADOS集群**实现快照**、**副本**和<mark style="color:purple;">一致性</mark>。

同时，RBD自身也是RADOS存储集群的客户端，它通过将存储池提供的存储服务抽象为一到多个image（表现为块设备）向客户端提供块级别的存储接口&#x20;

{% hint style="info" %}
RBD支持两种格式的image，不过v1格式因特性较少等原因已经处于废弃状态 当前默认使用的为v2格式
{% endhint %}

### 访问RBD设备方式

1. 通过内核模块rbd.ko将image映射为节点本地的块设备，相关的设备文件一般为/dev/rdb#（#为设备编号，例如rdb0等）&#x20;
2. 通过librbd提供的API接口，它支持C/C++和Python等编程语言，qemu即是此类接口的客户端

### 操作过程

RBD接口在ceph环境创建完毕后，就在服务端自动提供了; 客户端基于<mark style="color:purple;">**librbd**</mark>库即可将RADOS存储集群用作块设备。

不过，RADOS 集群体用块设备需要经过一系列的"额外操作"才能够为客户端提供正常的存储功能。 这个过程步骤，主要有以下几步：

1. 创建一个专用的存储池：ceph osd pool create {pool-name} {pg-num} {pgp-num}&#x20;
2. 对存储池启用rbd功能：ceph osd pool application enable {pool-name} rbd
3. 对存储池进行环境初始化：rbd pool init -p {pool-name}
4. 基于存储池创建专用的磁盘镜像：rbd create --size --pool

```
存储池中的各image名称需要惟一，“rbd ls”命令能够列出指定存储池中的image 
rbd ls [-p ] [--format json|xml] [--pretty-format]

要获取指定image的详细信息，则通常使用“rbd info”命令
rbd info [--pool <pool>] [--image <image>]  [--format <format>] ...
```
