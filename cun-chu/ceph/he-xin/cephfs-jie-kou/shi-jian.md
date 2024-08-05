# 实践

部署mds主机

```
在admin主机上为stor05上部署mds服务
$ ceph-deploy mds create stor05
​
在stor05上，并确认是否开启了守护进程
$ ssh stor05 "ps aux | grep ceph-mds | grep -v grep"
ceph       14471  0.1  1.2 384316 22504 ?        Ssl  11:48   0:00 /usr/bin/ceph-mds -f --cluster ceph --id stor05 --setuser ceph --setgroup ceph
```

在没有为fs准备存储池的前提下，是看不到效果的

```
在admin主机上通过 ceph -s 中查看mds的状态
[cephadm@admin ceph-cluster]$  ceph -s
  cluster:
    ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 6m)
    mgr: mon01(active, since 47h), standbys: mon02
    mds:  1 up:standby      # 一个mds部署成功
    osd: 6 osds: 6 up (since 6m), 6 in (since 43h)
    rgw: 1 daemon active (stor04)
```

```
需要专用的命令来进行查看mds的状态效果
[cephadm@admin ceph-cluster]$ ceph mds stat
 1 up:standby
```

创建专属存储池

```
创建元数据存储池
ceph osd pool create cephfs-metadata 32 32
ceph osd pool create cephfs-data 64 64
```

```
将刚才创建的存储池组合为fs专属存储池
[cephadm@admin ceph-cluster]$ ceph fs new cephfs cephfs-metadata cephfs-data
new fs with metadata pool 14 and data pool 15
```

确认效果

```
确认效果
[cephadm@admin ceph-cluster]$ ceph -s
  cluster:
    ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 11m)
    mgr: mon01(active, since 47h), standbys: mon02
    mds: cephfs:1 {0=stor05=up:active}
    osd: 6 osds: 6 up (since 11m), 6 in (since 43h)
    rgw: 1 daemon active (stor04)
    ...
```

```
查看状态
[cephadm@admin ceph-cluster]$ ceph fs status cephfs
cephfs - 0 clients
======
+------+--------+--------+---------------+-------+-------+
| Rank | State  |  MDS   |    Activity   |  dns  |  inos |
+------+--------+--------+---------------+-------+-------+
|  0   | active | stor05 | Reqs:    0 /s |   10  |   13  |
+------+--------+--------+---------------+-------+-------+
+-----------------+----------+-------+-------+
|       Pool      |   type   |  used | avail |
+-----------------+----------+-------+-------+
| cephfs-metadata | metadata | 1536k | 35.8G |
|   cephfs-data   |   data   |    0  | 35.8G |
+-----------------+----------+-------+-------+
+-------------+
| Standby MDS |
+-------------+
+-------------+
MDS version: ceph version 14.2.22 (ca74598065096e6fcbd8433c8779a2be0c889351) nautilus (stable)
```
