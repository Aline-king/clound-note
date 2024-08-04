# 存储

## 存储类别

1. <mark style="color:purple;">**DAS设备**</mark>：<mark style="color:purple;">**IDE、SATA、SCSI、SAS、USB （**</mark>无论是那种接口，都是存储设备驱动下的磁盘设备，而磁盘设备其实就是一种存储，这种存储是直接接入到主板总线上去的。<mark style="color:purple;">**）**</mark>&#x20;
   1. 基于数据块来进行访问&#x20;
   2. 基于服务器方式实现互联访问，操作简单、成本低。
2. <mark style="color:purple;">**NAS设备**</mark>：<mark style="color:purple;">**NFS**</mark>、<mark style="color:purple;">**CIFS（**</mark>几乎所有的网络存储设备基本上都是以文件系统样式进行使用，无法进一步格式化操作。<mark style="color:purple;">**）**</mark>
   1. 基于文件系统方式访问&#x20;
   2. 没有网络区域限制，支持多种协议操作文件。
3. <mark style="color:purple;">**SAN**</mark>：<mark style="color:purple;">**scsi协议、FC SAN、iSCSI（**</mark>基于SAN方式提供给客户端操作系统的是一种块设备接口，所以这些设备间主要是通过scsi协议来完成正常的通信。<mark style="color:purple;">**）**</mark>
   1. 基于数据块来实现访问
   2. 不受服务器约束，通过存储池实现资源的高效利用，扩展性好

{% hint style="info" %}
scsi的结构类似于TCP/IP协议，也有很多层，但是scsi协议主要是用来进行存储数据操作的。既然是分层方式实现的，那就是说，有部分层可以被替代。

比如将物理层基于FC方式来实现，就形成了FCSAN，

如果基于以太网方式来传递数据，就形成了iSCSI模式。
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:orange;">文件系统</mark>

文件系统的基本数据单位是**文件**，它的目的是对磁盘上的文件进行组织管理，那组织的方式不同，就会形成不同的文件系统。&#x20;

Linux 文件系统会为每个文件分配两个数据结构：**索引节点**(index node)和**目录项**(directory entry)，它们主要用来记录文件的元信息和目录层次结构。

**索引节点**&#x20;

1. 用来记录文件的元信息，比如 inode 编号、文件大小、访问权限、创建时间、等信息。&#x20;
2. 索引节点是文件的唯一标识，它们之间一一对应，也同样都会被存储在硬盘中，所以索引节点同样占用磁盘空间。&#x20;
3. 用户查找的时候，会根据inode的信息，找到对应的数据块，我们可以将inode理解为数据块的路由信息。&#x20;

**目录项**&#x20;

1. 用来记录文件的名字、索引节点指针以及与其他目录项的层级关联关系。多个目录项关联起来，形成目录结构&#x20;
2. 它与索引节点不同，目录项是由内核维护的一个数据结构，不存放于磁盘，而是缓存在内存。
3. 目录项和索引节点的关系是多对一

### 数据存储

数据块&#x20;

磁盘读写的最小单位是**扇区**，扇区的大小只有 **512B** 大小，文件系统把多个扇区组成了一个逻辑块，每次读写的最小单位就是**逻辑块**（数据块），

Linux 中的逻辑块大小为 4KB，也就是一次性读写 8 个扇区，这将大大提高了磁盘的读写的效率。 磁盘想要被文件系统使用，需要进行格式化，此时磁盘会被分成三个存储区域&#x20;

* 超级块，用来存储文件系统的详细信息，比如块个数、块大小、空闲块等等。
* 索引节点区，用来存储索引节点；&#x20;
* 数据块区，用来存储文件或目录数据；

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
为了加速文件的访问，通常会把相关信息加载到内存中，但是考虑到内存的容量限制，它们加载进内存的时机是不同的：&#x20;

超级块：当文件系统<mark style="color:blue;">**挂载时**</mark>进入内存；&#x20;

索引节点区：当文件被<mark style="color:purple;">**访问时**</mark>进入内存；&#x20;

数据块：文件数据<mark style="color:orange;">**使用时**</mark>进入内存；
{% endhint %}

### 文件存储

文件的数据是要存储在硬盘上面的，数据在磁盘上的存放方式，有两种方式：&#x20;

<mark style="color:purple;">**连续空间存放方式**</mark> ：同一个文件存放到一个连续的存储空间 ，一旦文件删除可能导致磁盘空间碎片，文件内容长度扩展不方便，综合效率是非常低的。&#x20;

<mark style="color:purple;">**非连续空间存放方式**</mark> ：同一个文件存放到一个不连续的存储空间，每个空间会关联下一个空间 ，可以消除磁盘碎片，可大大提高磁盘空间的利用率，同时文件的长度可以动态扩展。 查找效率低，需要额外的资源消耗。

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>
