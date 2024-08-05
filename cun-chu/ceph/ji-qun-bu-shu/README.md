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
为了方便后续的监控环境认证操作，在admin角色主机上，把配置文件和admin密钥拷贝Ceph集群各监控角色节点,拷贝前秘钥文件前的各个mon节点效果

```
[cephadm@admin ceph-cluster]$ for i in {13..15}; do ssh cephadm@10.0.0.$i "echo -----$i-----; ls /etc/ceph"; done
-----13-----
ceph.conf
rbdmap
tmpc8lO0G
-----14-----
ceph.conf
rbdmap
tmpkmxmh3
-----15-----
ceph.conf
rbdmap
tmp4GwYSs
```

原则上要求，所有mon节点上的 ceph.conf 内容必须一致，如果不一致的话，可以通过下面命令同步 ceph-deploy --overwrite-conf config push mon01 mon02 mon03

执行集群的认证文件的拷贝动作 `ceph-deploy admin mon01 mon02 mon03`

```
执行认证文件信息同步
[cephadm@admin ceph-cluster]$ ceph-deploy admin mon01 mon02 mon03
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
...
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to mon01
...
[mon01][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to mon02
...
[mon02][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to mon03
...
[mon03][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
```

所有的mon节点上多了一个 ceph的客户端与服务端进行认证的**秘钥文件**了。 **ceph.client.admin.keyring**：主要用于 ceph客户端与管理端的一个通信认证。

<mark style="background-color:blue;">如果我们不做交互式操作的话，这个文件可以不用复制。</mark>

```
查看效果
[cephadm@admin ceph-cluster]$ for i in {13..15}; do ssh cephadm@10.0.0.$i "echo -----$i-----; ls /etc/ceph"; done
-----13-----
ceph.client.admin.keyring
ceph.conf
rbdmap
tmpc8lO0G
-----14-----
ceph.client.admin.keyring
ceph.conf
rbdmap
tmpkmxmh3
-----15-----
ceph.client.admin.keyring
ceph.conf
rbdmap
tmp4GwYSs
```

### 认证文件权限

虽然我们把认证文件传递给对应的监控角色主机了，但是我们的服务式通过**普通用户**cephadm来进行交流的。而默认情况下，传递过去的认证文件，cephadm普通用户是无法正常访问的

我们需要在Ceph集群中需要运行ceph命令的的节点上，以root用户的身份设定普通用户cephadm能够读 取`/etc/ceph/ceph.client.admin.keyring`文件的权限。

```bash
[cephadm@admin ceph-cluster]$ for i in {13..15}; do ssh cephadm@10.0.0.$i "sudo setfacl -m u:cephadm:r /etc/ceph/ceph.client.admin.keyring"; done

查看文件权限效果
[cephadm@admin ceph-cluster]$ for i in {13..15}; do ssh cephadm@10.0.0.$i "echo -----$i-----; ls /etc/ceph/ceph.cl* -l"; done
-----13-----
-rw-r-----+ 1 root root 151 9月  25 12:06 /etc/ceph/ceph.client.admin.keyring
-----14-----
-rw-r-----+ 1 root root 151 9月  25 12:06 /etc/ceph/ceph.client.admin.keyring
-----15-----
-rw-r-----+ 1 root root 151 9月  25 12:06 /etc/ceph/ceph.client.admin.keyring

查看文件的授权信息
[root@mon01 ~]# getfacl /etc/ceph/ceph.client.admin.keyring
getfacl: Removing leading '/' from absolute path names
# file: etc/ceph/ceph.client.admin.keyring
# owner: root
# group: root
user::rw-
user:cephadm:r--
group::---
mask::r--
other::---
```

监控节点就可以自己来收集相关的数据了，比如我们在mon01上执行如下命令

```
[root@mon01 ~]# ceph -s
  cluster:
    id:     1d4e5773-619a-479d-861a-66ba451ce945
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 18m)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

我们的cluster状态不是正常的 对于service来说，有三个mon服务，选举的节点有三个，其他的服务没有。

集群状态不正常的原因，我们可以通过 ceph health命令来进行确认，效果如下

我们在所有的mon节点上进行提示属性的设定

```
[root@mon01 ~]# ceph health
HEALTH_WARN mons are allowing insecure global_id reclaim
[root@mon01 ~]# ceph health detail
HEALTH_WARN mons are allowing insecure global_id reclaim
AUTH_INSECURE_GLOBAL_ID_RECLAIM_ALLOWED mons are allowing insecure global_id reclaim
    mon.mon01 has auth_allow_insecure_global_id_reclaim set to true
    mon.mon02 has auth_allow_insecure_global_id_reclaim set to true
    mon.mon03 has auth_allow_insecure_global_id_reclaim set to true
```

解决

```
[root@mon01 ~]# ceph config set mon auth_allow_insecure_global_id_reclaim false
[root@mon01 ~]# ceph health detail
HEALTH_OK
```


{% endtab %}

{% tab title="mgr认证" %}
Ceph-MGR工作的模式是事件驱动型的，就是等待事件，事件来了则处理事件返回结果，又继续等待。

Ceph MGR 是 Ceph 12.2 依赖主推的功能之一，它负责 Ceph 集群管理的组件，它主要功能是把集群的一些指标暴露给外界使用。根据官方的架构原则上来说，mgr要有两个节点来进行工作。 对于我们的学习环境来说，其实一个就能够正常使用了,为了节省资源的使用，我们这里将mon01 和mon02主机节点兼做MGR节点，为了后续的节点扩充实践，我们暂时先安装一个节点，后面再安装一个节点。

```
未部署mgr节点的集群状态效果
[cephadm@admin ceph-cluster]$ ssh mon01  ceph -s
  cluster:
    id:     1d4e5773-619a-479d-861a-66ba451ce945
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 29m)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

mgr服务配置

```bash
配置Manager节点，启动ceph-mgr进程：
[cephadm@admin ceph-cluster]$ ceph-deploy mgr create mon01
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /bin/ceph-deploy mgr create mon01
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  mgr                           : [('mon01', 'mon01')]
[mon01][INFO  ] Running command: sudo systemctl start ceph-mgr@mon
​
```

在mon01上，部署了一个mgr的服务进程

```
在指定的mgr节点上，查看守护进程
[cephadm@admin ceph-cluster]$ ssh mon01 ps aux | grep -v grep | grep ceph-mgr
ceph        3065  2.8  6.6 1037824 124244 ?      Ssl  12:27   0:01 /usr/bin/ceph-mgr -f --cluster ceph --id mon01 --setuser ceph --setgroup ceph
```

这个时候，service上，多了一个mgr的服务，在mon01节点上，服务状态是 active。

```
查看集群服务的运行状态
[cephadm@admin ceph-cluster]$ ssh mon01  ceph -s
  cluster:
    ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 33m)
    mgr: mon01(active, since 86s)
    osd: 0 osds: 0 up, 0 in
  data:
```

admin查看状态

远程查看状态方式不太方便，我们可以在admin主机上进行一下操作来实现admin主机查看集群状态

```
sudo yum -y install ceph-common
ceph-deploy admin admin
sudo setfacl -m u:cephadm:rw /etc/ceph/ceph.client.admin.keyring
```

```
确认效果
[cephadm@admin ceph-cluster]$ ceph -s
  cluster:
    id:     1d4e5773-619a-479d-861a-66ba451ce945
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
​
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 37m)
    mgr: mon01(active, since 4m)
    osd: 0 osds: 0 up, 0 in
​
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```
{% endtab %}
{% endtabs %}

## OSD操作

osd存储引擎有两种：BlueStore和FileStore，自动Ceph L版之后，默认都是BlueStore了。

### 一般流程

我们可以通过一下四个步骤来设置OSD环境：&#x20;

1. 要知道对应的主机上有哪些磁盘可以提供给主机来进行正常的使用。&#x20;
2. 格式化磁盘(非必须)&#x20;
3. ceph擦除磁盘上的数据&#x20;
4. 添加osd

{% tabs %}
{% tab title="格式化" %}
为所有的节点主机都准备了两块额外的磁盘，

`fdisk -l`

<mark style="color:purple;">**磁盘格式化**</mark>

我们在所有的osd角色的主机上，进行磁盘的格式化操作,对所有的osd节点主机进行磁盘格式化。 `mkfs.ext4 /dev/sdb`&#x20;

`mkfs.ext4 /dev/sdc`

<mark style="color:purple;">**查看磁盘格式化效果**</mark>

以mon03为例&#x20;

```
[root@mon03 ~]# blkid | egrep "sd[bc]" 
/dev/sdb: UUID="49b5be7c-76dc-4ac7-a3e6-1f54526c83df" TYPE="ext4" 
/dev/sdc: UUID="ecdc0ce6-8045-4caa-b272-2607e70700ee" TYPE="ext4"
```
{% endtab %}

{% tab title="ceph擦除磁盘上的数据" %}
1. 保证所有包含OSD磁盘上主机上，安装ceph的命令&#x20;

`yum install -y ceph radosgw`

2. 检查并列出OSD节点上所有可用的磁盘的相关信息

{% code title="" %}
```
[cephadm@admin ceph-cluster]$ ceph-deploy disk list stor01 stor02 stor03
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /bin/ceph-deploy disk list stor01 stor02 stor03
...
[stor01][DEBUG ] connection detected need for sudo
...
[stor01][INFO  ] Running command: sudo fdisk -l
[stor02][DEBUG ] connection detected need for sudo
...
[stor02][INFO  ] Running command: sudo fdisk -l
[stor03][DEBUG ] connection detected need for sudo
...
[stor03][INFO  ] Running command: sudo fdisk -l
```
{% endcode %}

3. &#x20;在管理节点上使用ceph-deploy命令擦除计划专用于OSD磁盘上的所有分区表和数据以便用于OSD

```
，
[cephadm@admin ceph-cluster]$ for i in {1..3}
do 
   ceph-deploy disk zap stor0$i /dev/sdb /dev/sdc
done
...
[stor03][WARNIN]  stderr: 记录了10+0 的读入
[stor03][WARNIN] 记录了10+0 的写出
[stor03][WARNIN] 10485760字节(10 MB)已复制
[stor03][WARNIN]  stderr: ，0.018883 秒，555 MB/秒
[stor03][WARNIN] --> Zapping successful for: <Raw Device: /dev/sdc>
```
{% endtab %}
{% endtabs %}

### OSD实践

对于osd的相关操作，可以通过 ceph-deploy osd 命令来进行，帮助信息如下

`$ ceph-deploy osd --help`

两类的存储机制：

1. <mark style="color:purple;">**bluestore**</mark>
   1. \--data    /path/to/data ceph 保存的对象数据
   2. \--block-db    /path/to/db-device ceph 保存的对象数据
   3. \--block-wal   /path/to/wal-device 数据库的 wal 日志
2. <mark style="color:purple;">**filestore**</mark>
   1. \--data /path/to/data ceph的文件数据
   2. \--journal /path/to/journal 文件系统日志数据

对于 osd来说，它主要有两个动作：&#x20;

1. list 列出osd相关的信息
2. &#x20;create 创建osd设备

{% tabs %}
{% tab title="添加osd" %}
```
创建osd，我们这里全部用于存储数据。
ceph-deploy --overwrite-conf osd create stor01 --data /dev/sdb
ceph-deploy --overwrite-conf osd create stor01 --data /dev/sdc
```

这里只能一个磁盘一个磁盘的添加

```bash
查看效果
[cephadm@admin ceph-cluster]$ ceph-deploy --overwrite-conf osd create stor01 --data /dev/sdc
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /bin/ceph-deploy --overwrite-conf osd create stor01 --data /dev/sdc
...
[stor01][WARNIN] --> ceph-volume lvm activate successful for osd ID: 1
[stor01][WARNIN] --> ceph-volume lvm create successful for: /dev/sdc
[stor01][INFO  ] checking OSD status...
[stor01][DEBUG ] find the location of an executable
[stor01][INFO  ] Running command: sudo /bin/ceph --cluster=ceph osd stat --format=json
[ceph_deploy.osd][DEBUG ] Host stor01 is now ready for osd use.
```

在services部分，osd多了信息，有两个osd是up的状态,而且都加入到了集群中。

```
查看命令执行后的ceph的集群状态
[cephadm@admin ceph-cluster]$ ceph -s
  cluster:
    ...

  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 59m)
    mgr: mon01(active, since 27m)
    osd: 2 osds: 2 up (since 98s), 2 in (since 98s)

  data:
    ...
```

批量操作进行磁盘都格式化

```
for i in 2 3
do
  ceph-deploy --overwrite-conf osd create stor0$i --data /dev/sdb
  ceph-deploy --overwrite-conf osd create stor0$i --data /dev/sdc
done
```

再次查看集群状态

osd的磁盘数量达到了 6个，都是处于up的状态。 对于ceph集群的数据容量来说，一共有120G的磁盘空间可以使用

```
[cephadm@admin ceph-cluster]$ ceph -s
  cluster:
    ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 66m)
    mgr: mon01(active, since 34m)
    osd: 6 osds: 6 up (since 5s), 6 in (since 5s)

  data:
    ...
```
{% endtab %}

{% tab title="查看磁盘状态" %}
```
[cephadm@admin ceph-cluster]$ ceph-deploy osd list stor01
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /bin/ceph-deploy osd list stor01
...
[stor01][DEBUG ] connection detected need for sudo
[stor01][DEBUG ] connected to host: stor01
[stor01][DEBUG ] detect platform information from remote host
[stor01][DEBUG ] detect machine type
[stor01][DEBUG ] find the location of an executable
[ceph_deploy.osd][INFO  ] Distro info: CentOS Linux 7.9.2009 Core
[ceph_deploy.osd][DEBUG ] Listing disks on stor01...
[stor01][DEBUG ] find the location of an executable
[stor01][INFO  ] Running command: sudo /usr/sbin/ceph-volume lvm list
[stor01][DEBUG ]
[stor01][DEBUG ]
[stor01][DEBUG ] ====== osd.0 =======
[stor01][DEBUG ]
...
[stor01][DEBUG ]       type                      block
[stor01][DEBUG ]       vdo                       0
[stor01][DEBUG ]       devices                   /dev/sdb
[stor01][DEBUG ]
[stor01][DEBUG ] ====== osd.1 =======
[stor01][DEBUG ]
...
[stor01][DEBUG ]       type                      block
[stor01][DEBUG ]       vdo                       0
[stor01][DEBUG ]       devices                   /dev/sdc
```


{% endtab %}
{% endtabs %}

### OSD操作

OSD全称<mark style="color:purple;">Object Storage Device</mark>，负责响应客户端请求返回具体数据的进程。

一个Ceph集群中，有专门的osd角色主机，在这个主机中一般有很多个OSD设备。

```bash
ceph osd --help  这里面有几个是与osd信息查看相关的 
        ls		查看所有OSD的id值
	dump	     	查看 OSD 的概述性信息
	status		查看 OSD 的详细的状态信息
	stat		查看 OSD 的精简的概述性信息
	tree 		查看 OSD 在主机上的分布信息
	perf		查看 OSD 磁盘的延迟统计信息
	df		查看 OSD 磁盘的使用率信息
```



<details>

<summary>查看所有OSD的id值</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd ls
0
1
2
3
4
5
```

</details>

<details>

<summary>查看 OSD 的数据映射信息</summary>

```bash
[cephadm@admin ceph-cluster]$ ceph osd dump
epoch 25
...
max_osd 6
osd.0 up   in  weight 1 up_from 5 up_thru 0 down_at 0 last_clean_interval [0,0) [v2:10.0.0.13:6802/5544,v1:10.0.0.13:6803/5544] [v2:192.168.8.13:6800/5544,v1:192.168.8.13:6801/5544] exists,up 78bd7a7e-9dcf-4d91-be20-d5de2fbec7dd
...
```

</details>

<details>

<summary>查看指定OSD节点的信息</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd dump 3
epoch 3
fsid 1d4e5773-619a-479d-861a-66ba451ce945
...
osd.0 down out weight 0 up_from 0 up_thru 0 down_at 0 last_clean_interval [0,0)   exists,new 78bd7a7e-9dcf-4d91-be20-d5de2fbec7dd
```

</details>

<details>

<summary>查看 OSD 的精简的概述性信息</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd stat
6 osds: 6 up (since 4m), 6 in (since 4m); epoch: e25
```

OSD节点数量(osds) 集群内(in)、集群外(out) 运行(up)、不再运行(down) OSD 的每一次状态变更的历史信息(epoch)

</details>

<details>

<summary>查看 OSD 的详细的状态信息</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd status
+----+-------+-------+-------+--------+---------+--------+---------+-----------+
| id |  host |  used | avail | wr ops | wr data | rd ops | rd data |   state   |
+----+-------+-------+-------+--------+---------+--------+---------+-----------+
| 0  | mon01 | 1027M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 1  | mon01 | 1027M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 2  | mon02 | 1027M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 3  | mon02 | 1027M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 4  | mon03 | 1027M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 5  | mon03 | 1027M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
+----+-------+-------+-------+--------+---------+--------+---------+-----------+
```

</details>

<details>

<summary>查看OSD在各个主机上的分布情况</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0      up  1.00000 1.00000
 1   hdd 0.01949         osd.1      up  1.00000 1.00000
-5       0.03897     host mon02
 2   hdd 0.01949         osd.2      up  1.00000 1.00000
 3   hdd 0.01949         osd.3      up  1.00000 1.00000
-7       0.03897     host mon03
 4   hdd 0.01949         osd.4      up  1.00000 1.00000
 5   hdd 0.01949         osd.5      up  1.00000 1.00000
```

</details>

<details>

<summary>查看OSD磁盘的延迟统计信息</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd perf
osd commit_latency(ms) apply_latency(ms)
  5                  0                 0
  4                  0                 0
  0                  0                 0
  1                  0                 0
  2                  0                 0
  3                  0                 0
```

主要解决单块磁盘问题，如果有问题及时剔除OSD。&#x20;

统计的是平均值&#x20;

commit\_latency 表示从接收请求到设置commit状态的时间间隔&#x20;

apply\_latency 表示从接收请求到设置apply状态的时间间隔

</details>

<details>

<summary>查看OSD磁盘的使用率信息</summary>

```
[cephadm@admin ceph-cluster]$ ceph osd df
ID CLASS WEIGHT  REWEIGHT SIZE    RAW USE DATA    OMAP META  AVAIL   %USE VAR  PGS STATUS
 0   hdd 0.01949  1.00000  20 GiB 1.0 GiB 3.2 MiB  0 B 1 GiB  19 GiB 5.02 1.00   0     up
 1   hdd 0.01949  1.00000  20 GiB 1.0 GiB 3.2 MiB  0 B 1 GiB  19 GiB 5.02 1.00   0     up
 2   hdd 0.01949  1.00000  20 GiB 1.0 GiB 3.2 MiB  0 B 1 GiB  19 GiB 5.02 1.00   0     up
 3   hdd 0.01949  1.00000  20 GiB 1.0 GiB 3.2 MiB  0 B 1 GiB  19 GiB 5.02 1.00   0     up
 4   hdd 0.01949  1.00000  20 GiB 1.0 GiB 3.2 MiB  0 B 1 GiB  19 GiB 5.02 1.00   0     up
 5   hdd 0.01949  1.00000  20 GiB 1.0 GiB 3.2 MiB  0 B 1 GiB  19 GiB 5.02 1.00   0     up
                    TOTAL 120 GiB 6.0 GiB  20 MiB  0 B 6 GiB 114 GiB 5.02
MIN/MAX VAR: 1.00/1.00  STDDEV: 0
```



</details>

#### 进阶

{% tabs %}
{% tab title="osd 暂停开启" %}
```bash
ceph osd pause		集群暂停接收数据
ceph osd unpause	集群开始接收数据
```

<mark style="color:purple;">**pause**</mark>

```
[cephadm@admin ceph-cluster]$ ceph osd pause
pauserd,pausewr is set
[cephadm@admin ceph-cluster]$ ceph -s
  ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 2h)
    mgr: mon01(active, since 2h)
    osd: 6 osds: 6 up (since 107m), 6 in (since 107m)
         flags pauserd,pausewr			# 可以看到，多了pause的标签
  ...
```

<mark style="color:purple;">**unpause**</mark>

pause的标签已经被移除

```
[cephadm@admin ceph-cluster]$ ceph osd unpause
pauserd,pausewr is unset
[cephadm@admin ceph-cluster]$ ceph -s
  ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 2h)
    mgr: mon01(active, since 2h)
    osd: 6 osds: 6 up (since 107m), 6 in (since 107m)
  ...		
```
{% endtab %}

{% tab title="osd数据操作比重" %}
```bash
osd节点上线：ceph osd crush reweight osd.编号 权重值
```

```bash
查看默认的OSD操作权重值
[cephadm@admin ceph-cluster]$ ceph osd crush tree
ID CLASS WEIGHT  TYPE NAME
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0
 1   hdd 0.01949         osd.1
...

修改osd的数据操作权重值
[cephadm@admin ceph-cluster]$ ceph osd crush reweight osd.0 0.1
reweighted item id 0 name 'osd.0' to 0.1 in crush map
[cephadm@admin ceph-cluster]$ ceph osd crush tree
ID CLASS WEIGHT  TYPE NAME
-1       0.19742 root default
-3       0.11948     host mon01
 0   hdd 0.09999         osd.0
 1   hdd 0.01949         osd.1
...

恢复osd的数据操作权重值 
[cephadm@admin ceph-cluster]$ ceph osd crush reweight osd.0 0.01949
reweighted item id 0 name 'osd.0' to 0.01949 in crush map
[cephadm@admin ceph-cluster]$ ceph osd crush tree
ID CLASS WEIGHT  TYPE NAME
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0
 1   hdd 0.01949         osd.1
```
{% endtab %}

{% tab title="osd上下线" %}
```bash
osd节点上线：ceph osd down osd编号
osd节点下线：ceph osd up osd编号
```

由于OSD有专门的管理服务器控制，一旦发现被下线，会尝试启动它

```bash
# 将磁盘快速下线，然后查看状态
[cephadm@admin ceph-cluster]$ ceph osd down 0 ; ceph osd tree
marked down osd.0.
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0    down  1.00000 1.00000
...

# 等待一秒钟后查看状态，指定的节点又上线了
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0      up  1.00000 1.00000
...
```
{% endtab %}

{% tab title="驱逐加入OSD对象" %}
```
ceph osd out osd编号
ceph osd in osd编号
```

所谓的从OSD集群中驱离或者加入OSD对象，本质上Ceph集群数据操作的权重值调整

```
将0号OSD下线
[cephadm@admin ceph-cluster]$ ceph osd out 0
marked out osd.0.
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0      up        0 1.00000
... 

将0号OSD上线
[cephadm@admin ceph-cluster]$ ceph osd in 0;
marked in osd.0.
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
-1       0.11691 root default
-3       0.03897     host mon01
 0   hdd 0.01949         osd.0      up  1.00000 1.00000
...
```

OSD在上下线实践的时候，所谓的REWEIGHT会进行调整，1代表上线，空代表下线
{% endtab %}
{% endtabs %}

### OSD删除

将OSD删除需要遵循一定的步骤：&#x20;

1. 修改osd的数据操作权重值，让数据不分布在这个节点上&#x20;
2. 到指定节点上，停止指定的osd进程&#x20;
3. 将移除OSD节点状态标记为out&#x20;
4. 从crush中移除OSD节点，该节点不作为数据的载体&#x20;
5. 删除OSD节点&#x20;
6. 删除OSD节点的认证信息

#### <mark style="color:purple;">删除OSD节点实践</mark>

{% tabs %}
{% tab title="1 " %}
```
[cephadm@admin ceph-cluster]$ ceph osd crush reweight osd.5 0
reweighted item id 5 name 'osd.5' to 0 in crush map
[cephadm@admin ceph-cluster]$ ceph osd crush tree
ID CLASS WEIGHT  TYPE NAME
...
-7       0.01949     host mon03
 4   hdd 0.01949         osd.4
 5   hdd       0         osd.5
```
{% endtab %}

{% tab title="2" %}
```
[cephadm@admin ceph-cluster]$ ssh mon03 sudo systemctl disable ceph-osd@5
Removed symlink /etc/systemd/system/ceph-osd.target.wants/ceph-osd@5.service.
[cephadm@admin ceph-cluster]$ ssh mon03 sudo systemctl stop ceph-osd@5
[cephadm@admin ceph-cluster]$ ssh mon03 sudo systemctl status ceph-osd@5
...
9月 25 15:25:32 mon03 systemd[1]: Stopped Ceph object storage daemon osd.5.
```
{% endtab %}

{% tab title="3" %}
```
[cephadm@admin ceph-cluster]$ ceph osd out osd.5
marked out osd.5.
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
...
-7       0.01949     host mon03
 4   hdd 0.01949         osd.4      up  1.00000 1.00000
 5   hdd       0         osd.5    down        0 1.00000
```
{% endtab %}

{% tab title="4" %}
```
[cephadm@admin ceph-cluster]$ ceph osd crush remove osd.5
removed item id 5 name 'osd.5' from crush map

查看效果
[cephadm@admin ceph-cluster]$ ceph osd crush tree
ID CLASS WEIGHT  TYPE NAME
...
-7       0.01949     host mon03
 4   hdd 0.01949         osd.4
```

osd.5 已经被移除了
{% endtab %}

{% tab title="5" %}
```
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
...
-7       0.01949     host mon03
 4   hdd 0.01949         osd.4      up  1.00000 1.00000
 5             0 osd.5            down        0 1.00000
 
移除无效的osd节点
[cephadm@admin ceph-cluster]$ ceph osd rm osd.5
removed osd.5

再次确认效果
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
...
-7       0.01949     host mon03
 4   hdd 0.01949         osd.4      up  1.00000 1.00000
```

osd节点已经被移除了
{% endtab %}

{% tab title="6" %}
```
查看历史认证信息
[cephadm@admin ceph-cluster]$ ceph auth ls
...
osd.5
        key: AQCy4C9jI1wzDhAAaPjSPSQpMdu8NUPIRecctQ==
        caps: [mgr] allow profile osd
        caps: [mon] allow profile osd
        caps: [osd] allow *
...

删除OSD节点的认证信息
[cephadm@admin ceph-cluster]$ ceph auth del osd.5
updated
[cephadm@admin ceph-cluster]$ ceph auth ls
...
```

已经没有历史的节点信息了
{% endtab %}
{% endtabs %}

### OSD添加

将OSD增加到集群需要遵循一定的步骤：&#x20;

1. 确定OSD节点没有被占用&#x20;
2. 磁盘格式化&#x20;
3. ceph擦除磁盘上的数据&#x20;
4. 添加OSD到集群

#### <mark style="color:purple;">添加OSD节点实践</mark>

{% tabs %}
{% tab title="1" %}
<pre class="language-bash"><code class="lang-bash"># 确定OSD节点没有被占用
[root@mon03 ~]# blkid | egrep 'sd[bc]'
/dev/sdb: UUID="h3NNr5-Sgr8-nJ8k-QWbY-UiRu-Pbmu-nxs3pZ" TYPE="LVM2_member"
/dev/sdc: UUID="VdOW5k-zxMN-4Se5-Bxlc-8h93-uhPA-R30P3u" TYPE="LVM2_member"

<strong># 格式化磁盘失败
</strong>[root@mon03 ~]# mkfs.ext4 /dev/sdc
mke2fs 1.42.9 (28-Dec-2013)
/dev/sdc is entire device, not just one partition!
无论如何也要继续? (y,n) y
/dev/sdc is apparently in use by the system; will not make a 文件系统 here!

# 查看被占用的磁盘
[root@mon03 ~]# dmsetup status
ceph--9dd352be--1b6e--4d60--957e--81c59eafa7b3-osd--block--49251ee9--b1cc--432f--aed7--3af897911b60: 0 41934848 linear
ceph--3712fe04--cc04--4ce4--a75e--f7f746d62d6f-osd--block--99787a7e--40ee--48ae--802a--491c4f326d0c: 0 41934848 linear

# 确定OSD节点没有被占用，注意ceph的osd挂载目录
[root@mon03 ~]# cat /var/lib/ceph/osd/ceph-4/fsid
49251ee9-b1cc-432f-aed7-3af897911b60

# 移除被占用磁盘
[root@mon03 ~]# dmsetup remove ceph--3712fe04--cc04--4ce4--a75e--f7f746d62d6f-osd--block--99787a7e--40ee--48ae--802a--491c4f326d0c
[root@mon03 ~]# dmsetup status | grep 99787a7e
</code></pre>
{% endtab %}

{% tab title="2" %}
`mkfs.ext4 /dev/sdc`
{% endtab %}

{% tab title="3" %}
`ceph-deploy disk zap stor03 /dev/sdc`
{% endtab %}

{% tab title="4" %}
`ceph-deploy --overwrite-conf osd create stor03 --data /dev/sdc`

```
[cephadm@admin ceph-cluster]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME      STATUS REWEIGHT PRI-AFF
...
-7       0.03897     host mon03
 4   hdd 0.01949         osd.4      up  1.00000 1.00000
 5   hdd 0.01949         osd.5      up  1.00000 1.00000
```

之前被移除的osd节点已经被找回来了
{% endtab %}
{% endtabs %}
