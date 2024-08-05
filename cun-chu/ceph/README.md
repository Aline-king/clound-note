# ceph



> 官方介绍
>
> Ceph is the future of storage; where traditional(传统的) systems fail to deliver, Ceph is designed to excel(出色). Leverage(利用) your data for better business(业务) decisions(决策) and achieve(实现) operational(运营) excellence(卓越) through scalable, intelligent(智能), reliable(可靠) and highly available(可用) storage software. Ceph supports object, block and file storage, all in one unified(统一) storage system.

### 来源

Ceph项目最早起源于Sage就读博士期间的工作（最早的成果于2004年发表，论文发表于2006年），并随后贡献给开源社区。

{% hint style="info" %}
参考资料：&#x20;

官方地址：https://ceph.com/en/&#x20;

官方文档：https://docs.ceph.com/en/latest/#&#x20;

github地址：https://github.com/ceph/ceph&#x20;

最新版本：Quincy-17.2.3(2022-04-19)、Pacific-16.2.10(2021-03-31)、Octopus-15.2.17(2020-03-23)&#x20;

系统支持： 全系列最好的系统支持是Ubuntu&#x20;

Ceph的发展对于Centos系列和Redhat系列不友好，新版本不支持旧版本系统。
{% endhint %}

## 介绍

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ceph 是一个开源的分布式存储系统，同时支持对象存储、块设备、文件系统.

对象数据的底层存储服务是由多个主机（host）组成的存储集群，该集群也被称之为 **RADOS**（Reliable Automatic Distributed Object Store）存储集群，即可靠、自动化、 分布式对象存储系统 librados是RADOS存储集群的API，它支持C、C++、Java、Python、Ruby和PHP等编程语言。

### 为什么很多人用ceph？&#x20;

1. 创新，Ceph能够同时提供**对象存储**、**块存储**和**文件系统存储**三种存储服务的统一存储架构&#x20;
2. 算法，Ceph得以摒弃了传统的集中式存储元数据寻址方案，通过**Crush算法**的寻址操作，有相当强大的扩展性。
3. 可靠，Ceph中的数据副本数量可以由管理员自行定义，并可以通过Crush算法指定副本的物理存储位置以分隔故障域，支持数据强一致性的特性也使Ceph具有了高可靠性，可以忍受多种故障场景并自动尝试并行修复。

#### _基本结构_

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ceph是一个多版本存储系统，它把每一个待管理的数据流（例如一个文件） 切分为一到多个固定大小的对象数据，并以其为原子单元完成数据存取。&#x20;

_**应用场景**_

Ceph uniquely(独特的) delivers object, block, and file storage in one unified(统一) system. 注意：这个介绍在官网这几年基本没有变化

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

RadosGW、RBD和CephFS都是RADOS存储服务的客户端，它们把RADOS的存储服务接口(librados)分别从不同的角度做了进一步抽象，因而各自适用于不同的应用场景。 也就是说，ceph将三种存储类型统一在一个平台中，从而实现了更强大的适用性。

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

LIBRADOS：通过自编程方式实现数据的存储能力&#x20;

RADOSGW：通过标准的RESTful接口，提供一种云存储服务&#x20;

RBD：将ceph提供的空间，模拟出来一个一个的独立块设备使用。 ceph环境部署完毕后，服务端就准备好了rbd接口&#x20;

CFS：通过一个标准的文件系统接口来进行数据的存储

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
