# FUSE实践

## 简介

对于某些操作系统来说，它没有提供对应的ceph内核模块，我们还需要使用cephFS的话，可以通过 FUSE方式来实现，FUSE全称Filesystem in Userspace，用于<mark style="color:orange;">**非特权用户**</mark>能够无需操作内核而创建文件系统。

### 准备工作

```
如果我们在非内核的环境下使用cephFS的话，需要对客户端主机的环境做一些准备工作：
    安装ceph-fuse程序包
        yum install -y ceph-fuse ceph-common
    获取到客户端账号的keyring文件和ceph.conf配置文件
        ceph-deploy --overwrite-conf config push stor06
        sudo scp ceph.client.fsclient.keyring root@stor06:/etc/ceph/
```

**简单实践**

{% tabs %}
{% tab title="客户端环境准备" %}
```
我们准备将 stor06 作为cephFS的客户端主机，安装对应的软件
[root@stor06 ~]# yum install ceph-fuse ceph-common -y
​
保证 stor06上有ceph相关的配置文件
[root@stor06 ~]# ls /etc/ceph/
ceph.client.kube.keyring ...  ceph.conf  fsclient.key  rbdmap
```


{% endtab %}

{% tab title="挂载cephFS" %}
```
创建专属的数据目录
[root@stor06 ~]# mkdir /cephfs-data-user/
​
挂载cephFS对象
[root@stor06 ~]# ceph-fuse -n client.fsclient -m mon01:6789,mon02:6789,mon03:6789 /cephfs-data-user/
ceph-fuse[7963]: starting ceph client2062-09-27 12:23:47.905 7f6e97de4f80 -1 init, newargv = 0x5637c9545540 newargc=9
ceph-fuse[7963]: starting fuse
​
确认挂载效果
[root@stor06 ~]# mount | grep cephfs
ceph-fuse on /cephfs-data-user type fuse.ceph-fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0,allow_other)
```

```
查看挂载点的效果
[root@stor06 ~]# stat -f /cephfs-data-user/
  文件："/cephfs-data-user/"
    ID：0        文件名长度：255     类型：fuseblk
块大小：4194304    基本块大小：4194304
    块：总计：9182       空闲：9182       可用：9182
Inodes: 总计：1          空闲：0
    
确认文件系统使用效果
[root@stor06 ~]# echo nihao cephfs user > /cephfs-data-user/cephfs.txt
[root@stor06 ~]# cat /cephfs-data-user/cephfs.txt
nihao cephfs -user
```


{% endtab %}

{% tab title="开机自启动挂载cephFS" %}
```
卸载刚才挂载的cephFS
[root@stor06 ~]# umount /cephfs-data-user
[root@stor06 ~]# mount | grep ceph
​
设置开机自启动
[root@stor06 ~]# echo 'none  /cephfs-data-user   fuse.ceph   ceph.id=fsclient,ceph.conf=/etc/ceph/ceph.conf,_netdev,noatime 0 0' >> /etc/fstab
属性解析：
    _netdev - 告诉挂载程序，一旦超时，自动跳过去。
    noatime - 在读文件时不去更改文件的access time属性，提高性能
```

```
挂载磁盘
[root@stor06 ~]# mount -a
[root@stor06 ~]# mount | grep ceph
ceph-fuse on /cephfs-data-user type fuse.ceph-fuse (rw,nosuid,nodev,noatime,user_id=0,group_id=0,allow_other)
​
查看效果
[root@stor06 ~]# cat /cephfs-data-user/cephfs.txt
nihao cephfs user
​
专用的卸载命令
[root@stor06 ~]# fusermount -u /cephfs-data-user
[root@stor06 ~]# mount | grep cephfs
```


{% endtab %}
{% endtabs %}
