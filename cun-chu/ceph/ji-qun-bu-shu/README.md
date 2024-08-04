# 集群部署

参考资料：[https://docs.ceph.com/en/pacific/install/](https://docs.ceph.com/en/pacific/install/)

## 安装方法

<mark style="color:green;">**推荐方法**</mark>

1.  <mark style="color:purple;">**Cephadm**</mark>：使用容器和 systemd 安装和管理 Ceph 集群，并与 CLI 和仪表板 GUI 紧密集成。&#x20;

    仅支持 Octopus 和更新版本，需要容器和 Python3支持&#x20;

    与新的编排 API 完全集成&#x20;
2.  <mark style="color:purple;">**Rook**</mark>：在 Kubernetes 中运行的 Ceph 集群，同时还支持通过 Kubernetes API 管理存储资源和配置。&#x20;

    仅支持 Nautilus 和较新版本的 Ceph

<mark style="color:red;">**其他方法**</mark>

1. ceph-ansible：使用Ansible部署Ceph集群，对于新的编排器功能、管理功能和仪表板支持不好。
2. ceph-deploy：是一个快速部署集群的工具，不支持Centos8
3. DeepSea：使用Salt 安装 Ceph
4. ceph-mon：使用 Juju 安装 Ceph
5. Puppet-ceph：通过 Puppet 安装 Ceph
6. 二进制源码：手工安装
7. windows图形：在windows主机上，通过鼠标点点点的方式进行部署。

## 版本选择

版本地址：https://docs.ceph.com/en/latest/releases/&#x20;

最新版本：官网版本 ~~v16.2.10 Pacific~~&#x20;

版本特性：x.0.z(开发版)、x.1.z(候选版)、x.2.z(稳定、修正版)

## 环境规划

{% tabs %}
{% tab title="网络规划" %}
<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

公有网络(public)：  用于用户的数据通信 10.0.0.0/16&#x20;

集群网络(cluster)： 用于集群内部的管理通信  192.168.8.0/16
{% endtab %}

{% tab title="磁盘规划" %}
磁盘1 - VM的系统盘&#x20;

磁盘2和磁盘3 - Ceph的OSD
{% endtab %}

{% tab title="主机名规划" %}
<table><thead><tr><th width="113">主机名</th><th width="98">公有网络</th><th>私有网络</th><th>磁盘</th><th>其他角色</th></tr></thead><tbody><tr><td> admin</td><td>10.0.0.12</td><td>192.168.8.12</td><td>sdb、sdc </td><td></td></tr><tr><td>stor01</td><td>10.0.0.13</td><td>192.168.8.13</td><td>sdb、sdc </td><td>mon1</td></tr><tr><td>stor02</td><td>10.0.0.14</td><td>192.168.8.14</td><td>sdb、sdc </td><td>mon2</td></tr><tr><td>stor03</td><td>10.0.0.15</td><td>192.168.8.15</td><td>sdb、sdc </td><td>mon3</td></tr><tr><td>stor04</td><td>10.0.0.16</td><td>192.168.8.16</td><td>sdb、sdc </td><td></td></tr><tr><td>stor05</td><td>10.0.0.17</td><td>192.168.8.17</td><td>sdb、sdc </td><td></td></tr><tr><td>stor06</td><td>10.0.0.18</td><td>192.168.8.18</td><td>sdb、sdc </td><td></td></tr></tbody></table>

由于生产中，ceph的集群角色是非常多的，当我们的主机量少的时候，只能让一台主机节点运行多个角色。 stor01\~03 这三台主机，还同时兼具 mon的角色，视情况兼容mgr角色 主机名的完整格式是： xxx.superopsmsb.com
{% endtab %}

{% tab title="其他准备" %}
不推荐直接使用root用户来管理

时间同步

内网dns管理主机

VM主机准备（如果有的话）

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

所有节点都准备三块盘，两块网卡&#x20;

**虚拟网络设置：**&#x20;

VMnet1 设定为 192.168.8.0 网段，&#x20;

VMnet8 设定为 10.0.0.0 网段&#x20;

**虚拟机设置**&#x20;

额外添加两块盘，每个根据自己的情况设定容量，我这里设定为20G&#x20;

额外更加一块网络适配器，使用仅主机模式 -- VMnet1，mac地址必须要重新生成，避免冲突
{% endtab %}
{% endtabs %}

## 定制软件源

ceph-deploy方式部署Ceph，Ceph的官方仓库路径是http://download.ceph.com/，包括各种ceph版本，比如octopus、pacific、quincy等，它根据不同OS系统环境，分别位于rpm-版本号 或者 debian-版本号 的noarch目录下。

比如pacific版本的软件相关源在 rpm-octopus/el7/noarch/ceph-release-1-1.el7.noarch.rpm。

{% hint style="info" %}
注意：&#x20;

el7代表支持Red Hat 7.x、CentOS 7.x 系统的软件

el8代表支持Red Hat 8.x、CentOS 8.x 系统的软件&#x20;

pacific版本及其更新版本，只支持CentOS 8.x环境
{% endhint %}

Ceph的pacific和Quincy版本，仅仅支持CentOS8.X，Octopus版本虽然有CentOS7的版本，不仅仅软件不全，而且对于底层GCC库和GLIBC库要求比较高，如果升级CentOS7的底层库，会导致其他软件受到影响，无法正常使用，另外没有配套的ceph-deploy。所以对于CentOS7来说，只能部署Nautilus版本和更低版本。&#x20;

对于Ubuntu系统来说，即使多个版本对于底层环境要求有些区别，但是经过测试，问题不大，也就是说Ubuntu系统可以安装Ceph的全系列

1. 安装软件源 （所有主机都要做）

`yum install -y https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch/ceph-release-1-1.el7.noarch.rpm`

2. 更新软件源&#x20;

`yum makecache fast`

#### 部署依赖

3. admin主机安装ceph软件&#x20;

`yum update -y yum install ceph-deploy python-setuptools python2-subprocess32 -y`

4. 测试效果&#x20;

`su - cephadm -c "ceph-deploy --help"`

## Ceph部署

{% tabs %}
{% tab title="集群创建" %}
首先在管理节点上以cephadm用户创建集群相关的配置文件目录：&#x20;

`su - cephadm mkdir ceph-cluster && cd ceph-cluster`

初始化集群解析

`ceph-deploy new --help`

初始化第一个MON节点的命令格式为”ceph-deploy new {initial-monitor-node(s)}“，

mon01即为第一个MON节点名称，其名称必须与节点当前实际使用的主机名称(uname -n)保存一致

可以是短名称，也可以是长名称，但是最终用的仍然是短名称,但是会导致如下报错：

`ceph-deploy new: error: hostname:  xxx is not resolvable`

推荐使用完整写法：格式 hostname:fqdn，比如 mon01:mon01.superopsmsb.com

如果初始化的时候，希望同时部署多个节点的话，使用空格隔开hostname:fqdn即可 如果部署过程出现问题，需要清空&#x20;

ceph-deploy forgetkeys&#x20;

ceph-deploy purge mon01&#x20;

ceph-deploy purgedata mon01&#x20;

rm ceph.\*
{% endtab %}

{% tab title="集群初始化" %}
部署3个mon节点

```bash
[cephadm@admin ceph-cluster]$ ceph-deploy new --public-network 10.0.0.0/24 --cluster-network 192.168.8.0/24 mon01:mon01.superopsmsb.com mon02:mon02.superopsmsb.com mon03:mon03.superopsmsb.com --no-ssh-copykey
[ceph_deploy.new][DEBUG ] Resolving host mon03.superopsmsb.com
[ceph_deploy.new][DEBUG ] Monitor mon03 at 10.0.0.15
[ceph_deploy.new][DEBUG ] Monitor initial members are ['mon01', 'mon02', 'mon03']
[ceph_deploy.new][DEBUG ] Monitor addrs are [u'10.0.0.13', u'10.0.0.14', u'10.0.0.15']
[ceph_deploy.new][DEBUG ] Creating a random mon key...
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...
```

如果出现如下报错： `[ceph_deploy][ERROR ] AttributeError: 'module' object has no attribute 'needs_ssh'`&#x20;

在执行命令的时候，添加一个 --no-ssh-copykey 参数即可 这主要是因为免密认证的时候，没有进行 ssh cephadm@主机名 导致的

```bash
# 查看初始化后的文件内容
[cephadm@admin ceph-cluster]$ ls
ceph.conf  ceph.log  ceph.mon.keyring

# 查看集群的配置文件
[cephadm@admin ceph-cluster]$ cat ceph.conf
[global]
fsid = 7738ce65-2d87-481c-8253-9d0d4b29e8eb  # 这个地方很重要的，每次都不一样，不要乱动
public_network = 10.0.0.0/24
cluster_network = 192.168.8.0/24
mon_initial_members = mon01, mon02, mon03
mon_host = 10.0.0.13,10.0.0.14,10.0.0.15
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
filestore_xattr_use_omap = true

# 查看集群通信的认证信息
[cephadm@admin ceph-cluster]$ cat ceph.mon.keyring
[mon.]
key = AQBZ9y1jAAAAABAAKnVONZ3l+EEpjUOjtK8Xmw==
caps mon = allow *

# 查看集群初始化的日志信息
[cephadm@admin ceph-cluster]$ cat ceph.log
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
...
```


{% endtab %}

{% tab title="部署mon" %}
ceph-deploy命令能够以远程的方式连入Ceph集群各节点完成程序包安装等操作

```
命令格式：
	ceph-deploy install {ceph-node} [{ceph-node} ...]
	示例：ceph-deploy install --nogpgcheck admin mon01 mon02 mon03
```

这里主要是ceph的工作角色的的节点 一般情况下，不推荐使用这种直接的方法来进行安装，效率太低，而且容易干扰其他主机环境

1. 手工在所有节点上安装ceph软件（<mark style="color:purple;">**推荐**</mark> ）

`yum install -y ceph ceph-osd ceph-mds ceph-mon ceph-radosgw`

2. 最后在admin角色主机上安装&#x20;

`ceph-deploy install --no-adjust-repos --nogpgcheck admin mon01 mon02 mon03`

{% code title="执行过程" %}
```
[cephadm@admin ceph-cluster]$ ceph-deploy install --no-adjust-repos --nogpgcheck admin mon01 mon02 mon03
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.25): /bin/ceph-deploy install --no-adjust-repos admin mon01 mon02 mon03
[ceph_deploy.install][DEBUG ] Installing stable version hammer on cluster ceph hosts admin mon01 mon02 mon03
[ceph_deploy.install][DEBUG ] Detecting platform for host mon01 ...
[admin][DEBUG ] connection detected need for sudo
[admin][DEBUG ] connected to host: admin
...
[mon03][DEBUG ] 完毕！
[mon03][INFO  ] Running command: sudo ceph --version
[mon03][DEBUG ] ceph version 14.2.22 (ca74598065096e6fcbd8433c8779a2be0c889351) nautilus (stable)
```
{% endcode %}


{% endtab %}

{% tab title="集群通信认证" %}
配置初始MON节点，同时向所有节点同步配置 `ceph-deploy mon create-initial`

为了避免因为认证方面导致的通信失败，尤其是在现有环境上，推荐使用 --overwrite-conf 参数 `ceph-deploy --overwrite-conf config push mon01 mon02 mon03`

{% code title="执行效果" %}
```bash
[cephadm@admin ceph-cluster]$ ceph-deploy mon create-initial
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /bin/ceph-deploy mon create-initial
...
[mon01][DEBUG ] ********************************************************************************
[mon01][DEBUG ] status for monitor: mon.mon01
[mon01][DEBUG ] {
[mon01][DEBUG ]   "election_epoch": 0,
[mon01][DEBUG ]   "extra_probe_peers": [
...
[mon01][DEBUG ]   "feature_map": {
...
[mon01][DEBUG ]   },
[mon01][DEBUG ]   "features": {
...
[mon01][DEBUG ]   },
[mon01][DEBUG ]   "monmap": {
...
[mon01][DEBUG ]     "mons": [
[mon01][DEBUG ]       {
[mon01][DEBUG ]         "addr": "10.0.0.13:6789/0",
[mon01][DEBUG ]         "name": "mon01",
[mon01][DEBUG ]         "public_addr": "10.0.0.13:6789/0",
[mon01][DEBUG ]         "public_addrs": {
[mon01][DEBUG ]           "addrvec": [
[mon01][DEBUG ]             {
[mon01][DEBUG ]               "addr": "10.0.0.13:3300",
[mon01][DEBUG ]               "nonce": 0,
[mon01][DEBUG ]               "type": "v2"
[mon01][DEBUG ]             },
[mon01][DEBUG ]             {
[mon01][DEBUG ]               "addr": "10.0.0.13:6789",
[mon01][DEBUG ]               "nonce": 0,
[mon01][DEBUG ]               "type": "v1"
[mon01][DEBUG ]             }
[mon01][DEBUG ]           ]
[mon01][DEBUG ]         },
[mon01][DEBUG ]         "rank": 0
[mon01][DEBUG ]       },
[mon01][DEBUG ]       {
[mon01][DEBUG ]         "addr": "0.0.0.0:0/1",
[mon01][DEBUG ]         "name": "mon02",
...
[mon01][DEBUG ]         "rank": 1
[mon01][DEBUG ]       },
[mon01][DEBUG ]       {
[mon01][DEBUG ]         "addr": "0.0.0.0:0/2",
[mon01][DEBUG ]         "name": "mon03",
...
[mon01][DEBUG ]         "rank": 2
[mon01][DEBUG ]       }
[mon01][DEBUG ]     ]
[mon01][DEBUG ]   },
[mon01][DEBUG ]   "name": "mon01",
[mon01][DEBUG ]   "outside_quorum": [
[mon01][DEBUG ]     "mon01"
[mon01][DEBUG ]   ],
[mon01][DEBUG ]   "quorum": [],
[mon01][DEBUG ]   "rank": 0,
[mon01][DEBUG ]   "state": "probing",
[mon01][DEBUG ]   "sync_provider": []
[mon01][DEBUG ] }
[mon01][DEBUG ] ********************************************************************************
[mon01][INFO  ] monitor: mon.mon01 is running
[mon01][INFO  ] Running command: sudo ceph --cluster=ceph --admin-daemon /var/run/ceph/ceph-mon.mon01.asok mon_status
...
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-osd.keyring
[ceph_deploy.gatherkeys][INFO  ] Storing ceph.bootstrap-rgw.keyring
[ceph_deploy.gatherkeys][INFO  ] Destroy temp directory /tmp/tmpyal7qW
```
{% endcode %}

在所有的节点主机上，都有一套ceph-mon的进程在进行。

```
到mon的节点上查看mon的守护进程
[cephadm@admin ceph-cluster]$ for i in {13..15}; do ssh cephadm@10.0.0.$i "ps aux | grep -v grep | grep ceph-mon"; done
ceph        2189  0.1  1.8 504004 34644 ?        Ssl  11:55   0:00 /usr/bin/ceph-mon -f --cluster ceph --id mon01 --setuser ceph --setgroup ceph
ceph        2288  0.1  1.7 502984 33328 ?        Ssl  11:55   0:00 /usr/bin/ceph-mon -f --cluster ceph --id mon02 --setuser ceph --setgroup ceph
ceph        2203  0.1  1.7 502988 32912 ?        Ssl  11:55   0:00 /usr/bin/ceph-mon -f --cluster ceph --id mon03 --setuser ceph --setgroup ceph
```

```
集群在初始化的时候，会为对应的mon节点生成配套的认证信息
[cephadm@admin ceph-cluster]$ ls /home/cephadm/ceph-cluster
ceph.bootstrap-mds.keyring  ceph.bootstrap-osd.keyring  ceph.client.admin.keyring  
ceph-deploy-ceph.log 		ceph.bootstrap-mgr.keyring  ceph.bootstrap-rgw.keyring
ceph.conf                  	ceph.mon.keyring
```

这里生成了一系列的与ceph集群相关的认证文件&#x20;

1. **ceph.bootstrap-mds.keyring 引导启动 mds的秘钥文件**&#x20;
2. **ceph.bootstrap-osd.keyring 引导启动 osd的秘钥文件**
3. **ceph.client.admin.keyring ceph客户端和管理端通信的认证秘钥，是最重要的**
4. **ceph-deploy-ceph.log ceph.bootstrap-mgr.keyring 引导启动 mgr的秘钥文件**
5. **ceph.bootstrap-rgw.keyring 引导启动 rgw的秘钥文件**&#x20;
6. **ceph.conf ceph.mon.keyring**&#x20;

注意： ceph.client.admin.keyring 拥有ceph集群的所有权限，一定不能有误。
{% endtab %}

{% tab title="mon认证" %}

{% endtab %}
{% endtabs %}

