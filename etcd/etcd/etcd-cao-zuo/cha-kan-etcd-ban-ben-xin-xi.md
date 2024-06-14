# 查看etcd版本信息

## ETCD操作

```
检查etcd的版本信息
sh-5.1# ETCDCTL_API=3 etcdctl \
  --endpoints 10.0.0.12:2379,10.0.0.14:2379,10.0.0.13:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  version
etcdctl version: 3.5.1
API version: 3.5
```
