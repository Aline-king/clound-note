# 使用环境变量

```
sh-5.1# export ETCDCTL_API=3
sh-5.1# etcdctl version
etcdctl version: 3.5.1
API version: 3.5

定制命令别名，自动附加认证信息
sh-5.1# alias etcdctl='etcdctl --endpoints 10.0.0.12:2379,10.0.0.14:2379,10.0.0.13:2379  \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key'
sh-5.1# etcdctl endpoint status
10.0.0.12:2379, f7a9c20602b8532e, 3.5.1, 7.4 MB, false, false, 62, 294909, 294909,
10.0.0.14:2379, 2f4d6688eb4a75f7, 3.5.1, 7.7 MB, false, false, 62, 294909, 294909,
10.0.0.13:2379, 278be81a99993ec, 3.5.1, 7.7 MB, true, false, 62, 294909, 294909,
注意：
	在定制别名的时候，最好让 etcdctl --endpoints 在同一行，否则会出现异常情况
```
