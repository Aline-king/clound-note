# 安装

`etcd` 基于 `Go` 语言实现，因此，用户可以从 [项目主页](https://github.com/etcd-io/etcd) 下载源代码自行编译，也可以下载编译好的二进制文件，甚至直接使用制作好的 `Docker` 镜像文件来体验。

> 注意：本章节内容基于 etcd `3.4.x` 版本

{% tabs %}
{% tab title="二进制" %}
编译好的二进制文件在 [github.com/etcd-io/etcd/releases](https://github.com/etcd-io/etcd/releases/) ，用户可以选择需要的版本，或通过下载工具下载。

使用 `curl` 工具下载压缩包，并解压

```bash
$ curl -L https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz -o etcd-v3.4.0-linux-amd64.tar.gz

# 国内用户可以使用以下方式加快下载
$ curl -L https://download.fastgit.org/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz -o etcd-v3.4.0-linux-amd64.tar.gz

$ tar xzvf etcd-v3.4.0-linux-amd64.tar.gz
$ cd etcd-v3.4.0-linux-amd64
```

解压后，可以看到文件包括

{% hint style="info" %}
`etcd` 是服务主文件，

`etcdctl` 是提供给用户的命令客户端，

其他文件是支持文档。
{% endhint %}

```bash
$ ls
Documentation README-etcdctl.md README.md READMEv2-etcdctl.md etcd etcdctl
```

下面将 `etcd` `etcdctl` 文件放到系统可执行目录（例如 `/usr/local/bin/`）。

```bash
 sudo cp etcd* /usr/local/bin/
```

{% hint style="info" %}
默认 `2379` 端口处理客户端的请求，

`2380` 端口用于集群各成员间的通信。
{% endhint %}

启动 `etcd` 显示类似如下的信息：

```bash
$ etcd
...
2017-12-03 11:18:34.411579 I | embed: listening for peers on http://localhost:2380
2017-12-03 11:18:34.411938 I | embed: listening for client requests on localhost:2379
```

```bash
$ ETCDCTL_API=3 etcdctl member list
8e9e05c52164694d, started, default, http://localhost:2380, http://localhost:2379

$ ETCDCTL_API=3 etcdctl put testkey "hello world"
OK

$ etcdctl get testkey
testkey
hello world
```

此时，可以使用 `etcdctl` 命令进行测试，设置和获取键值 `testkey: "hello world"`，检查 `etcd` 服务是否启动成功：


{% endtab %}

{% tab title="yum集群安装" %}
#### 安装

```bash
# yum -y install etcd
```

```bash
# yum -y install etcd
```

#### 配置

```bash
# vim /etc/etcd/etcd.conf
[root@node1 ~]# cat /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/node1.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="node1"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.255.154:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.255.154:2379,http://192.168.255.155:4001"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="node1=http://192.168.255.154:2380,node2=http://192.168.255.155:2380"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
```

```bash
# vim /etc/etcd/etcd.conf
[root@node2 ~]# cat /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/node2.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="node2"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.255.155:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.255.155:2379,http://192.168.255.155:4001"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="node1=http://192.168.255.154:2380,node2=http://192.168.255.155:2380"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
```

#### 启动

```bash
# systemctl enable etcd

# systemctl start etcd
```

#### 检查端口状态

```bash
# netstat -tnlp | grep -E  "4001|2380"
```

{% code title="输出结果" %}
```
tcp6       0      0 :::2380                 :::*                    LISTEN      65318/etcd
tcp6       0      0 :::4001                 :::*                    LISTEN      65318/etcd
```
{% endcode %}

#### 检查etcd集群是否健康

```
# etcdctl -C http://192.168.255.154:2379 cluster-health
```

```
# etcdctl member list
```
{% endtab %}

{% tab title="Docker 镜像方式运行" %}
镜像名称为 `quay.io/coreos/etcd`，可以通过下面的命令启动 `etcd` 服务监听到 `2379` 和 `2380` 端口。

```bash
$ docker run \
-p 2379:2379 \
-p 2380:2380 \
--mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
--name etcd-gcr-v3.4.0 \
quay.io/coreos/etcd:v3.4.0 \
/usr/local/bin/etcd \
--name s1 \
--data-dir /etcd-data \
--listen-client-urls http://0.0.0.0:2379 \
--advertise-client-urls http://0.0.0.0:2379 \
--listen-peer-urls http://0.0.0.0:2380 \
--initial-advertise-peer-urls http://0.0.0.0:2380 \
--initial-cluster s1=http://0.0.0.0:2380 \
--initial-cluster-token tkn \
--initial-cluster-state new \
--log-level info \
--logger zap \
--log-outputs stderr
```

打开新的终端按照上一步的方法测试 `etcd` 是否成功启动。
{% endtab %}

{% tab title="MacOS" %}
```
$ brew install etcd

$ etcd

$ etcdctl member list
```
{% endtab %}
{% endtabs %}
