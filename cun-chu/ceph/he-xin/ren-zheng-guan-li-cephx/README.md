# 认证管理 Cephx

## <mark style="color:orange;">**Cephx**</mark>

Ceph作为一个分布式存储系统，支持对象存储、块设备和文件系统。

为了在网络传输中防止数据被篡改，做到较高程度的安全性，加入了<mark style="color:orange;">**Cephx**</mark>加密认证协议。其目的是客户端和管理端之间的身份识别，加密、验证传输中的数据。&#x20;

注意：所谓的client就是使用ceph命令的客户端

如果我们提供的是公共的信息 -- 不需要进行认证，所以我们可以禁用CephX功能。将对应的属性值设置为 none即可，比如 `auth_cluster_required = none`

```bash
ceph集群默认开启了cephx协议功能
[cephadm@admin ceph-cluster]$ cat ceph.conf
[global]
...
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
...
```

### 认证场景

1. 命令行与Ceph集群操作
   1. RadosGW 对象网关认证（对象网关认证系统+cephx）
   2. RBD 认证
   3. CephFS 用户认证（文件路径+cephx）
2. Ceph集群内部组件之间通信
   1. 执行ceph命令时，会使用client.admin 用户并加载 /etc/ceph/ceph.client.admin.keyring文件

### 认证和授权

在ceph系统中，所有元数据保存在mon节点的ceph-mon的进程中，mon保存了系统中重要的认证相关元数据，

例如每个用户的key以及权限。样式结构如下：

```bash
名称             key       Caps（权限）
client.admin    xxxxxx    osd allow rw, mon allow rw
```

对于ceph的认证和授权来说，主要涉及到三个内容：<mark style="color:blue;">ceph用户</mark>、<mark style="color:purple;">资源权限</mark>、<mark style="color:orange;">用户授权</mark>。

* Ceph用户必须拥有执行权限才能执行Ceph的管理命令&#x20;
* Ceph用户需要拥有<mark style="color:green;">**存储池访问权限**</mark>才能到ceph中读取和写入数据

#### 基本概念

<table data-header-hidden><thead><tr><th width="112"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>用户</td><td><ul><li>ceph创建出来的用户，可以进入到ceph集群里面</li><li>ceph支持多种类型，可管理的用户属于Client类型</li><li>MON、OSD和MDS等系统组件属于系统参与者客户端</li></ul></td><td></td></tr><tr><td>授权</td><td><ul><li>将某些资源的使用权限交给特定的用户。</li><li>allow</li></ul></td><td></td></tr><tr><td>权限</td><td><ul><li>描述用户可针对MON、OSD或MDS等资源的使用权限范围或级别</li><li>MON具备的权限包括r、w、x和allow profile cap</li><li>OSD具备的权限包括r、w、x和class-read、class-write和profile osd、存储池和名称空间设置</li><li>OSD具备的权限包括allow</li></ul></td><td></td></tr></tbody></table>

#### 权限解析

1. <mark style="color:purple;">**allow**</mark> 需先于守护进程的访问设置指定，标识赋予的xx权限&#x20;
2. <mark style="color:purple;">r</mark>：读取权限，访问MON以检索CRUSH时依赖此使能&#x20;
3. <mark style="color:purple;">**w**</mark>：对象写入权限&#x20;
4. <mark style="color:purple;">**x**</mark>：调用类方法（读取和写入）的能力，以及在MON上执行auth操作的能力&#x20;
5. <mark style="color:purple;">**class-read**</mark>：x能力的子集，授予用户调用类读取方法的能力&#x20;
6. <mark style="color:purple;">**class-writ**</mark>e：x的子集，授予用户调用类写入方法的能力&#x20;
7. <mark style="color:purple;">**\***</mark>：授予用户对特定守护进程/存储池的读取、写入和执行权限，以及执行管理命令的能力

***

1. <mark style="color:green;">**profile osd**</mark>：授予用户以某个OSD身份连接到其他OSD或监视器的权限&#x20;
2. <mark style="color:green;">**profile mds**</mark>：授予用户以某个MDS身份连接到其他MDS或监视器的权限
3. <mark style="color:green;">**profile bootstrap-osd**</mark>：授予用户引导OSD的权限，在部署时候产生&#x20;
4. <mark style="color:green;">**profile bootstrap-mds**</mark>：授予用户引导元数据服务器的权限，在部署时候产生
