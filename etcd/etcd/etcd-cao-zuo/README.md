# etcd操作

etcd的集群环境中，只能通过sh来连接，这个终端中不支持回退甚至ls、grep命令都不支持

命令内容超出一行内容会出现结构性变形，可以借助于换行符号来实现内容的持续输入

etcdctl 目前主要有两种命令模式 2版本和3版本，通过辩论指定命令的版本

export ETCDCTL\_API=3



## 进入etcd集群

```
登录到任意一个etcd pod中检测集群状态
[root@kubernetes-master1 ~]# kubectl -n kube-system exec -it etcd-kubernetes-master1 -- /bin/sh

查看命令帮助
sh-5.1# etcdctl --help
...
COMMANDS:
        ...
        get                     Gets the key or a range of keys
        help                    Help about any command
        endpoint health         Checks the healthiness of endpoints specified in `--endpoints` flag
        endpoint status         Prints out the status of endpoints specified in `--endpoints` flag
        ...
        member add              Adds a member into the cluster
        member list             Lists all members in the cluster
        ...
        version                 Prints the version of etcdctl

OPTIONS:
      --cacert=""                               verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""                                 identify secure client using this TLS certificate file
      ...
      --endpoints=[127.0.0.1:2379]              gRPC endpoints
  ...
      --key=""                         identify secure client using this TLS key file
      ...
      -w, --write-out="simple"        etcd的信息支持四种格式(fields, json, protobuf, simple, table)  
```

