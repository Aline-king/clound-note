# 存储原理

Ceph 存储集群从 Ceph 客户端接收数据,无论是通过 Ceph 块设备、Ceph 对象存储、Ceph 文件系统还是您使用 librados 创建的自定义实现， 这些数据存储为 **RADOS** 对象，每个对象都存储在一个对象存储设备上。&#x20;

Ceph OSD 守护进程处理存储驱动器上的读、写和复制操作。它主要基于两种文件方式实现：&#x20;

\- Filestore方式，每个 RADOS 对象都作为一个单独的文件存储在传统文件系统（通常是 XFS）上。&#x20;

\- BlueStore方式，对象以类似整体数据库的方式存储，这是新版本ceph默认的存储方式。

{% hint style="warning" %}
在Ceph中，每一个文件都会被拆分为多个独立的object，然后按照上面的逻辑进行持久化。
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Ceph OSD 守护进程将"数据"作为对象存储在平面命名空间中,该对象包括如下部分：

<table data-header-hidden><thead><tr><th width="153"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>标识符</td><td>在内存中唯一查找的标识 </td><td></td></tr><tr><td>二进制数据</td><td>每个对象的真实数据 </td><td></td></tr><tr><td>属性数据</td><td>由一组名称/值对组成的元数据，语义完全取决于 Ceph 客户端。 </td><td></td></tr></tbody></table>

{% hint style="warning" %}
对象 ID 在整个集群中是唯一的，而不仅仅是本地文件系统.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
