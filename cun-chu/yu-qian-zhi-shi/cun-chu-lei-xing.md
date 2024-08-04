# 存储类型

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="块存储" %}
块存储是将裸磁盘空间整个映射给主机使用的，主机层面操作系统识别出硬盘，与我们服务器内置的硬盘没有什么差异,可以自由的进行磁盘进行分区、格式化，&#x20;

块存储不仅仅是直接使用物理设备，间接使用物理设备的也叫块设备，比如虚机创建虚拟磁盘、虚机磁盘格式包括raw、qcow2等。&#x20;

常见解决方案： ABS(Azure Block Storage)、GBS(Google Block Storage)、EBS(Elastic Block Storag) Cinder、Ceph RBD、sheepdog
{% endtab %}

{% tab title="文件系统存储" %}
最常见、最容易、最经典实现的一种存储接口。它可以直接以文件系统挂载的方式，为主机提供存储空间资源。它有别于块存储的特点就在于，可以实现共享效果，但是有可能受网络因素限制导致速度较慢。

常见解决方案： AFS(Azure FileStorage)、GFS(Google FileStorage)、EFS(Elastic File Storage) Swift、CephFS、HDFS、NFS、CIFS、Samba、FTP
{% endtab %}

{% tab title="对象存储" %}
其核心是将 **数据读或写** 和 **元数据** 分离，并且基于对象存储设备(Object-based Storage Device，OSD)构建存储系统，然后借助于对象存储设备的管理功能，自动管理该设备上的数据分布。 对象存储主要包括四部分：对象、对象存储设备、元数据服务器、对象存储系统的客户端

块存储读写块，不利于共享，

文件存储读写慢，利于共享，

对象存储综合了两者的优点。&#x20;

常见的解决方案： AS(Azure Storage)、GCS(Google Cloud Storage)、S3(Simple Storage Service) Swift、Ceph OSD
{% endtab %}
{% endtabs %}
