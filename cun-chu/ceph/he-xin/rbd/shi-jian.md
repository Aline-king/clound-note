# 实践

## 创建专用存储池

这里面的pg数量，我们定制为64个

```
[cephadm@admin ceph-cluster]$ ceph osd pool create rbddata 64
pool 'rbddata' created
```

## 对存储池启用rbd功能

如果关闭应用的话，使用disable

```bash
[cephadm@admin ceph-cluster]$ ceph osd pool application enable rbddata rbd
enabled application 'rbd' on pool 'rbddata'
	
# 查看rbc的效果
[cephadm@admin ceph-cluster]$ ceph osd pool application get rbddata
{
    "rbd": {}
}
```

## 对存储池进行环境初始化

```
环境初始化
[cephadm@admin ceph-cluster]$ rbd pool init -p rbddata

查看效果
[cephadm@admin ceph-cluster]$ rbd pool stats rbddata
Total Images: 0
Total Snapshots: 0
Provisioned Size: 0 B
```

## 基于存储池创建专用的磁盘镜像

{% hint style="info" %}
这个时候，我们创建出来的磁盘影响文件，就可以在客户端上，通过内核机制，直接导入到内核中，在内核中被当成一个磁盘设备来进行使用，样式就是 /dev/xxx，然后就可以针对这个rdb磁盘设备，进行各种后续分区、格式化等操作。
{% endhint %}

```bash
创建镜像
[cephadm@admin ceph-cluster]$ rbd create img1 --size 1024 --pool rbddata
[cephadm@admin ceph-cluster]$ rbd ls -p rbddata
img1

查看状态
[cephadm@admin ceph-cluster]$ rbd pool stats rbddata
Total Images: 1
Total Snapshots: 0
Provisioned Size: 1 GiB
```

## 查看磁盘信息

```
[cephadm@admin ceph-cluster]$ rbd info rbddata/img1
[cephadm@admin ceph-cluster]$ rbd --image img1 --pool rbddata info
rbd image 'img1':
        size 1 GiB in 256 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: 3817a3738fac
        block_name_prefix: rbd_data.3817a3738fac
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features:
        flags:
        create_timestamp: Mon Sep 26 10:46:25 2062
        access_timestamp: Mon Sep 26 10:46:25 2062
        modify_timestamp: Mon Sep 26 10:46:25 2062
```

<table data-header-hidden><thead><tr><th width="138"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>size</td><td>这个块的大小，即1024MB=1G，1024MB/256 = 4M,共分成了256个对象(object)，每个对象4M。 </td><td></td></tr><tr><td>order</td><td>就是 22： 有效范围为12-25。22是个幂编号，4M是22， 8M是23，也就是2^22 bytes = 4MB, 2^23 bytes = 8MB</td><td></td></tr><tr><td> block_name_prefix</td><td>这个是块的最重要的属性了，这是每个块在ceph中的唯一前缀编号，有了这个前缀，把主机上的OSD都拔下来带回家，就能复活所有的VM了</td><td></td></tr><tr><td> format</td><td>格式有两种，1和2 </td><td></td></tr><tr><td>features</td><td><p>当前image启用的功能特性，其值是一个以逗号分隔的字符串列表，</p><p>例如 layering, exclusive-lock, object-map</p></td><td></td></tr><tr><td> op_features</td><td>可选的功能特性</td><td></td></tr></tbody></table>

## 磁盘设备的删除

```
删除设备
[cephadm@admin ceph-cluster]$ rbd rm rbddata/img1
Removing image: 100% complete...done.

查看效果
[cephadm@admin ceph-cluster]$ rbd ls -p rbddata
[cephadm@admin ceph-cluster]$ rbd pool stats rbddata
Total Images: 0
Total Snapshots: 0
Provisioned Size: 0 B

删除存储池
[cephadm@admin ceph-cluster]$ ceph osd pool rm rbddata rbddata --yes-i-really-really-mean-it
pool 'rbddata' removed
```
