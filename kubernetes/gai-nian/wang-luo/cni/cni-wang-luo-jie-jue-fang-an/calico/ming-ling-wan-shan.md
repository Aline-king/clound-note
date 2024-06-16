# 命令完善

calico本身是一个复杂的系统，复杂到它自己提供一个非常重要的Restful接口，结合calicoctl命令来管理自身的相关属性信息。

calicoctl可以直接与etcd进行操作，也可以通过kube-apiserver的方式与etcd来进行操作。

> 默认情况下，它与kube-apiserver通信的认证方式与kubectl的命令使用同一个context。但是我们还是推荐，使用手工定制的一个配置文件。

calicoctl 是运行在集群之外的，用于管理集群功能的一个重要的组件。

### calicoctl 的安装方式

[官网](https://projectcalico.docs.tigera.io/getting-started/kubernetes/hardway/the-calico-datastore#install)

1. 单主机方式、
2. kubectl命令插件方式、
3. pod方式、
4. 主机容器方式

获取专用命令

```bash
cd /usr/local/bin/
curl -L https://github.com/projectcalico/calico/releases/download/v3.24.1/calicoctl-linux-amd64 -o calicoctl
chmod +x calicoctl
```

查看帮助

```
# calicoctl --help
Usage:
  calicoctl [options] <command> [<args>...]
```

## 演示

查看ip的管理

`calicoctl ipam --help`

查看ip的信息

```bash
# calicoctl ipam show
+----------+---------------+-----------+------------+--------------+
| GROUPING |     CIDR      | IPS TOTAL | IPS IN USE |   IPS FREE   |
+----------+---------------+-----------+------------+--------------+
| IP Pool  | 10.244.0.0/16 |     65536 | 0 (0%)     | 65536 (100%) |
+----------+---------------+-----------+------------+--------------+
```

查看信息的显式效果

`calicoctl ipam show --help`

显式相关的配置属性

```bash

# calicoctl ipam show --show-configuration
+--------------------+-------+
|      PROPERTY      | VALUE |
+--------------------+-------+
| StrictAffinity     | false |
| AutoAllocateBlocks | true  |
| MaxBlocksPerHost   |     0 |
+--------------------+-------+
```

## calico集成到kubectl

定制kubectl 插件子命令

```bash
# cd /usr/local/bin/
# cp -p calicoctl kubectl-calico
```

测试效果

```bash
# kubectl calico --help
Usage:
  kubectl-calico [options] <command> [<args>...]
```

获取网络节点效果

```bash
[root@kubernetes-master1 /usr/local/bin]# kubectl calico node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+----------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
+--------------+-------------------+-------+----------+-------------+
| 10.0.0.15    | node-to-node mesh | up    | 07:30:48 | Established |
| 10.0.0.17    | node-to-node mesh | up    | 07:30:48 | Established |
| 10.0.0.13    | node-to-node mesh | up    | 07:30:48 | Established |
| 10.0.0.14    | node-to-node mesh | up    | 07:30:51 | Established |
| 10.0.0.16    | node-to-node mesh | up    | 07:31:41 | Established |
+--------------+-------------------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.
```
