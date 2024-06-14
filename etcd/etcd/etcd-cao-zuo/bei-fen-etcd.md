# 备份etcd

## 备份相关命令

备份ETCD集群时，只需要备份一个ETCD就行，因为集群节点间数据会自动同步

```
sh-5.1# etcdctl --help

		...

        snapshot restore        从默认数据目录还原备份数据

        snapshot save           备份etcd数据到默认数据目录

        snapshot status         查看备份状态
```

## 注意事项

```
etcd在进行备份的时候，禁止出现多个数据入口，否则会发生报错
sh-5.1# etcdctl snapshot save /var/lib/etcd/a.db
Error: snapshot must be requested to one selected node, not multiple [10.0.0.12:2379 10.0.0.14:2379 10.0.0.13:2379]

制作命令别名
sh-5.1# alias etcdctl='etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key'
```

## 数据备份

```
sh-5.1# etcdctl  --endpoints=https://10.0.0.12:2379  \
>   snapshot save /var/lib/etcd/snapshot-etcd-1.db
{"level":"info","ts":1659233955.8515584,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"/var/lib/etcd/snapshot-etcd-1.db.part"}
...
Snapshot saved at /var/lib/etcd/snapshot-etcd-1.db

宿主机查看效果
[root@kubernetes-master1 ~]# ll -h /var/lib/etcd/snapshot-etcd-1.db
-rw------- 1 root root 7.1M 7月  31 10:19 /var/lib/etcd/snapshot-etcd-1.db
```
