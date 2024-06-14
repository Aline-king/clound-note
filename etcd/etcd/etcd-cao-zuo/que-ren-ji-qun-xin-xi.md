# 确认集群信息

```
sh-5.1# ETCDCTL_API=3 etcdctl \
  --endpoints 10.0.0.12:2379,10.0.0.14:2379,10.0.0.13:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  member list
278be81a99993ec, started, kubernetes-master2, https://10.0.0.13:2380, https://10.0.0.13:2379, false
2f4d6688eb4a75f7, started, kubernetes-master3, https://10.0.0.14:2380, https://10.0.0.14:2379, false
f7a9c20602b8532e, started, kubernetes-master1, https://10.0.0.12:2380, https://10.0.0.12:2379, false
```

## 确认集群成员信息

```
sh-5.1# etcdctl member list
278be81a99993ec, started, kubernetes-master2, https://10.0.0.13:2380, https://10.0.0.13:2379, false
2f4d6688eb4a75f7, started, kubernetes-master3, https://10.0.0.14:2380, https://10.0.0.14:2379, false
f7a9c20602b8532e, started, kubernetes-master1, https://10.0.0.12:2380, https://10.0.0.12:2379, false
```

## 以表格方式查看信息

```

sh-5.1# etcdctl -w table endpoint status
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 10.0.0.12:2379 | f7a9c20602b8532e |   3.5.1 |  6.6 MB |     false |      false |        10 |     160474 |             160474 |        |
| 10.0.0.14:2379 | 2f4d6688eb4a75f7 |   3.5.1 |  6.7 MB |     false |      false |        10 |     160474 |             160474 |        |
| 10.0.0.13:2379 |  278be81a99993ec |   3.5.1 |  6.6 MB |      true |      false |        10 |     160474 |             160474 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
结果显示：
	三个etcd，一个为true,二个为false，实现了内部集群状态一主两从
```



```
切换主角色的id
sh-5.1# etcdctl move-leader f7a9c20602b8532e
Leadership transferred from 2f4d6688eb4a75f7 to f7a9c20602b8532e

再次查看效果
sh-5.1# etcdctl endpoint status
10.0.0.12:2379, f7a9c20602b8532e, 3.5.1, 7.4 MB, true, false, 96, 310291, 310291,
10.0.0.14:2379, 2f4d6688eb4a75f7, 3.5.1, 7.7 MB, false, false, 96, 310291, 310291,
10.0.0.13:2379, 278be81a99993ec, 3.5.1, 7.7 MB, false, false, 96, 310291, 310291,
```
