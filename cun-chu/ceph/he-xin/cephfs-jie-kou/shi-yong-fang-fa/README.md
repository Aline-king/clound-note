# 使用方法

客户端使用cephFS，主要有两种方式：&#x20;

1. 内核文件系统 - Ceph、libcephfs
2. 用户空间文件系统 - FUSE(Filesystem in USErspace)

<figure><img src="../../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## 前提

1. 至少一个节点运行ceph-mds守护进程&#x20;
2. 创建存储池 ceph-metadata、ceph-data&#x20;
3. 激活文件系统&#x20;
4. 无论那种客户端使用cephFS的方式，都需要有专门的认证机制。

### 准备认证文件

注意：我们只需要对data的存储池进行操作即可，不需要对metadata存储池进行授权。

```bash
在admin节点上，准备认证账号
[cephadm@admin ceph-cluster]$ ceph auth get-or-create client.fsclient mon 'allow r' mds 'allow rw' osd 'allow rwx pool=cephfs-data' -o ceph.client.fsclient.keyring
```

```
查看账号信息
[cephadm@admin ceph-cluster]$ ceph auth get client.fsclient
​
查看认证文件信息
[cephadm@admin ceph-cluster]$ ls *fsclient*
ceph.client.fsclient.keyring
[cephadm@admin ceph-cluster]$ cat ceph.client.fsclient.keyring
[client.fsclient]
        key = AQBgdTJj+LVdFBAAZOjGIsw4t+o1swZlW4CvKQ== 
```

### 秘钥管理

客户端挂载cephFS的时候，需要指定**用户名**和**key**，而这两项是有不同的两个选项来指定的，所以我们应该将这两个内容分开存储 ​

```
获取相关的秘钥信息
[cephadm@admin ceph-cluster]$ ceph-authtool -p -n client.fsclient ceph.client.fsclient.keyring
AQBgdTJj+LVdFBAAZOjGIsw4t+o1swZlW4CvKQ==
[cephadm@admin ceph-cluster]$ ceph auth print-key client.fsclient
AQBgdTJj+LVdFBAAZOjGIsw4t+o1swZlW4CvKQ==
```

```
保存用户账号的密钥信息于secret文件，用于客户端挂载操作认证之用
[cephadm@admin ceph-cluster]$ ceph auth print-key client.fsclient > fsclient.key
[cephadm@admin ceph-cluster]$ cat fsclient.key
AQBgdTJj+LVdFBAAZOjGIsw4t+o1swZlW4CvKQ==
```

## 实践

ceph客户端的需求

{% hint style="info" %}
<mark style="color:purple;">**内核模块ceph.ko：**</mark>yum install ceph

<mark style="color:purple;">**安装ceph-common程序包：**</mark>yum install ceph-common

<mark style="color:purple;">**提供ceph.conf配置文件**</mark> ：ceph-deploy --overwrite-conf config push stor06

<mark style="color:purple;">**获取到用于认证的密钥文件**</mark> ：sudo scp fsclient.key stor06:/etc/ceph/   &#x20;
{% endhint %}

{% tabs %}
{% tab title="cephFS客户端准备" %}
我们准备将 stor06 作为cephFS的客户端主机

```
检查stor06的ceph模块加载情况
[root@stor06 ~]# modinfo ceph
...
​
保证 stor06上有ceph相关的配置文件
[root@stor06 ~]# ls /etc/ceph/
ceph.client.kube.keyring ...  ceph.conf  fsclient.key  rbdmap
```




{% endtab %}

{% tab title="挂载cephFS" %}
内核空间挂载ceph文件系统，要求必须指定ceph文件系统的挂载路径

```
创建专属的数据目录
[root@stor06 ~]# mkdir /cephfs-data/
​
挂载cephFS对象
[root@stor06 ~]# mount -t ceph mon01:6789,mon02:6789,mon03:6789:/ /cephfs-data/ -o name=fsclient,secretfile=/etc/ceph/fsclient.key
    
确认挂载效果
[root@stor06 ~]# mount | grep cephfs
10.0.0.13:6789,10.0.0.14:6789,10.0.0.15:6789:/ on /cephfs-data type ceph (rw,relatime,name=fsclient,secret=<hidden>,acl,wsize=16777216)
```

```
查看挂载点的效果
[root@stor06 ~]# stat -f /cephfs-data/
  文件："/cephfs-data/"
    ID：c931549bd8ae8624 文件名长度：255     类型：ceph
块大小：4194304    基本块大小：4194304
    块：总计：9182       空闲：9182       可用：9182
Inodes: 总计：0          空闲：-1
    
确认文件系统使用效果
[root@stor06 ~]# echo nihao cephfs > /cephfs-data/cephfs.txt
[root@stor06 ~]# cat /cephfs-data/cephfs.txt
nihao cephfs
```


{% endtab %}

{% tab title="开机自启动挂载cephFS" %}
```
卸载刚才挂载的cephFS
[root@stor06 ~]# umount /cephfs-data
[root@stor06 ~]# mount | grep ceph
​
设置开机自启动
[root@stor06 ~]# echo 'mon01:6789,mon02:6789,mon03:6789:/  /cephfs-data  ceph name=fsclient,secretfile=/etc/ceph/fsclient.key,_netdev,noatime 0 0' >> /etc/fstab
```

<mark style="color:purple;">**属性解析：**</mark>

&#x20;\_netdev ：告诉挂载程序，一旦超时，自动跳过去。&#x20;

noatime ：在读文件时不去更改文件的access time属性，提高性能

```
挂载磁盘
[root@stor06 ~]# mount -a
[root@stor06 ~]# mount | grep ceph
10.0.0.13:6789,10.0.0.14:6789,10.0.0.15:6789:/ on /cephfs-data type ceph (rw,noatime,name=fsclient,secret=<hidden>,acl)
​
查看效果
[root@stor06 ~]# cat /cephfs-data/cephfs.txt
nihao cephfs
```


{% endtab %}
{% endtabs %}



\
