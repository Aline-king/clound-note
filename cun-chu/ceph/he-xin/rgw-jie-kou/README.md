# RGW接口

{% embed url="https://app.gitbook.com/o/yrIaYU7odlzYhLkHdGCW/s/CciQTOc6CLGkCYciwhF1/~/changes/75/cun-chu/yu-qian-zhi-shi/oss-dui-xiang-cun-chu" %}
对象存储基础知识
{% endembed %}

## 简介

Ceph对象存储使用Ceph对象网关守护进程（RADOS gw），它是用于与Ceph存储群集进行交互的HTTP服务器。

由于它提供与OpenStack Swift和Amazon S3兼容的接口，因此Ceph对象网关具有自己的用户管理。

Ceph对象网关可以将数据存储在用于存储来自Ceph文件系统客户端或Ceph块设备客户端的数据的同一Ceph存储群集中。

S3和Swift API共享一个公共的名称空间，因此可以使用一个API编写数据，而使用另一个API检索数据。

Ceph RGW基于 **librados**，是为应用提供RESTful类型的对象存储接口。

RGW提供两种类型的接口： 　　

1. S3：兼容Amazon S3RESTful API； 　　
2. Swift：兼容OpenStack Swift API。 　　&#x20;

同时，RGW为了实现RESTful接口的功能，默认使用Civetweb作为其Web Sevice，而Civetweb默认使用端口7480提供服务，如果想修改端口（如80端口），就需要修改Ceph的配置文件。

### 注意

RGW在创建的时候，自动会初始化自己的存储池，而且RGW还需要自己独有的守护进程服务才可以正常的使用

RGW并非必须的接口，仅在需要用到与S3和Swift兼容的RESTful接口时才需要部署RGW实例

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>RGW内部逻辑处理层级结构图</p></figcaption></figure>

<mark style="color:purple;">**HTTP 前端:**</mark> 接收数据操作请求。&#x20;

<mark style="color:purple;">**REST API接口：**</mark> 从http请求中解析出 S3 或 Swift 数据并进行一系列检查。&#x20;

<mark style="color:purple;">**API操作逻辑:**</mark> 经Restful 接口检查通过后，根据内部业务逻辑交由不同业务模块进行数据处理。&#x20;

<mark style="color:purple;">**RADOS接口**</mark>： 如果需要写入或者操作ceph集群数据，则经由RADOS + librados 调用将数据发送给集群的后端主机

### 变动

自0.80版本起，Ceph放弃了基于apache和fastcgi提供~~radosgw~~服务的传统而代之以默认嵌入在ceph-radosgw进程中的**Citeweb**，

这种新的实现方式更加轻便和简洁，但直到Ceph 11.0.1版本，Citeweb才开始支持SSL协议。

Citeweb默认监听于TCP协议的7480端口提供 http  服务，修改配置需要编辑ceph.conf配置文件，以如下格式进行定义

```
[client.rgw.<gateway-node>]
rgw_host = <hostname OR ipaddr>
rgw_frontends = "civetweb port=80"
```

配置完成后需要重启 ceph-radosgw 进程以生效新配置

```
ceph-radosgw@rgw.<gateway-node>
```

<mark style="color:purple;">其他相关的配置</mark>

```
配置https
    额外添加参数ssl_certificate=/PATH/TO/PEM_FILE
    定义port=443s，或者port=80+443s
​
其它配置参数
    num_threads：Citeweb以线程模型处理客户端请求，它为每个连接请求分配一个专用线
                        程，因而此参数定义了其支持的最大并发连接数，默认值为50
    request_timeout_ms：网络发送与接收操作的超时时长，以ms为单位，默认值为30000
                        可以在必要时通过增大此值实现长连接的效果
    access_log_file：访问日志的文件路径，默认为空
    error_log_file：错误日志的文件路径，默认为空
    rgw_dns_name： 定制专属的dns服务解析域名
```

## 底层

RGW是一个对象处理网关。数据实际存储在ceph集群中。利用librados的接口，与ceph集群通信。

RGW主要存储三类数据：元数据(metadata)、索引数据(bucket index)、数据(data)。&#x20;

1. <mark style="color:purple;">元数据信息</mark>：包括user(用户信息)，bucket(存储桶关系)，bucket.instance(存储桶实例)。
2. &#x20;<mark style="color:purple;">索引数据信息</mark>: 主要k/v的结构来维护bucket中rgw对象的索引信息，val是关于rgw对象的元数据信息
3. &#x20;<mark style="color:purple;">数据信息</mark>: 其实就是rgw object内容，它存放在一个或多个rados object中

这三类数据一般存储在不同的pool中，元数据也分多种元数据，存在不同的ceph pool中。

默认情况下，RGW安装完毕后，会在rados集群上生成包括如下存储池的一系列存储池

```
[cephadm@admin ceph-cluster]$ ceph osd pool ls
.rgw.root
default.rgw.control
default.rgw.meta
default.rgw.log

当我们开始基于对象存储的方式实现数据访问后，它就会生成另外一些存储池
[cephadm@admin ~]$ ceph osd pool ls
.rgw.root			# 包含的都是zone,zonegroup,realm等信息
default.rgw.control		# pool中对象的控制信息
default.rgw.meta		# 包含的是元数据信息
default.rgw.log			# 包含的是日志处理信息，包括垃圾回收机制
default.rgw.buckets.index	# 包含的是存储桶和对象的映射信息
default.rgw.buckets.data	# 包含的是存储桶的对象数据信息
```

这里的 default 指的是 ceph 的  zone 区域。&#x20;

control利用librados提供的对象 watch-notify 功能，当有数据更新时，通知其他RGW刷新cache。

### 元数据

{% tabs %}
{% tab title="查看元数据" %}
```
[cephadm@admin ~]$ radosgw-admin metadata list
[
    "bucket",
    "bucket.instance",
    "otp",      # One-time password 一次性密码机制
    "user"
]
```


{% endtab %}

{% tab title="查看用户" %}
```
查看用户
[cephadm@admin ~]$ radosgw-admin metadata list user
[
    "s3user"
]
​
获取用户细节信息
[cephadm@admin ~]$ radosgw-admin metadata list user:s3user
...
```


{% endtab %}

{% tab title="存储桶信息" %}
```
查看存储桶
[cephadm@admin ~]$ radosgw-admin metadata list bucket
[
    "images",
    "swift-super"
]
​
获取存储桶信息
[cephadm@admin ~]$ radosgw-admin metadata get bucket:images:
...
```


{% endtab %}

{% tab title="存储桶事例" %}
```
获取存储桶实例
[cephadm@admin ~]$ radosgw-admin metadata list bucket.instance
[
    "images:09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7",
    "swift-super:09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.4"
]
​
获取存储桶实例信息
[cephadm@admin ~]$ radosgw-admin metadata get bucket.instance:images:09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7
...
```


{% endtab %}
{% endtabs %}

### **简单实践**

元数据存储池

```bash
{zone}.rgw.meta 存放的是元数据
    root: bucket及bucket-instance
    users.keys: 用户key
    users.email:用户Email,object的key值=email
    users.swift: swift账号
    users.uid: s3用户及用户的Bucket信息
    roles：
    heap：
```

{% tabs %}
{% tab title="查看用户的uid信息" %}
```bash
[cephadm@admin ~]$ rados -p default.rgw.meta -N users.uid ls
s3user
s3user.buckets
```
{% endtab %}

{% tab title="获取用户管理的buckets" %}
```
[cephadm@admin ~]$ rados -p default.rgw.meta  -N users.uid listomapkeys s3user.buckets
images
swift-super
```
{% endtab %}

{% tab title="默认查看bucket信息" %}
```
​所以我们需要将其保存到一个文件中，通过专用命令进行查看
[cephadm@admin ~]$ rados -p default.rgw.meta  -N users.uid getomapval s3user.buckets images images_bucket
Writing to images_bucket
​
查看加密文件信息
[cephadm@admin ~]$ ceph-dencoder import images_bucket type cls_user_bucket_entry decode dump_json
{
    "bucket": {
        "name": "images",
        "marker": "09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7",
        "bucket_id": "09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7"
    },
    "size": 23,
    "size_rounded": 4096,
    "creation_time": "2022-09-27 00:49:53.839482Z",
    "count": 1,
    "user_stats_sync": "true"
}
```
{% endtab %}

{% tab title="其他信息获取" %}
```
bucket信息： rados -p default.rgw.meta -N root ls
bucket元数据：rados -p default.rgw.meta -N root get {bucket_id} meta_file
```
{% endtab %}
{% endtabs %}

索引数据存储池

```
获取buckets的索引信息
[cephadm@admin ~]$ rados -p default.rgw.buckets.index ls
.dir.09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.4
.dir.09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7
```

```
查看存储桶的对象信息
[cephadm@admin ~]$ rados -p default.rgw.buckets.index listomapkeys .dir.09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7
linux/issue
```

```
查看对象元数据信息
[cephadm@admin ~]$ rados -p default.rgw.buckets.index getomapval .dir.09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7 linux/issue object_key
Writing to object_key
​
查看文件内容信息
[cephadm@admin ~]$ ceph-dencoder type rgw_bucket_dir_entry import object_key decode dump_json         {
    "name": "linux/issue",
    "instance": "",
    "ver": {
        "pool": 11,
        "epoch": 1
    },
    "locator": "",
    "exists": "true",
    ...
}
```

```
文件对象的header中记录了一些统计信息
[cephadm@admin ~]$ rados -p default.rgw.buckets.index getomapheader .dir.09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7 index_object_header
Writing to index_object_header
​
查看统计信息
[cephadm@admin ~]$ ceph-dencoder type rgw_bucket_dir_header import index_object_header decode dump_json
{
    "ver": 2,
    "master_ver": 0,
    "stats": [
        1,
        {
            "total_size": 23,
            "total_size_rounded": 4096,
            "num_entries": 1,
            "actual_size": 23
        }
    ],
    "new_instance": {
        "reshard_status": "not-resharding",
        "new_bucket_instance_id": "",
        "num_shards": -1
    }
}
```

数据信息存储池

```
查看数据对象
[cephadm@admin ~]$ rados -p default.rgw.buckets.data ls
09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7_linux/issue
​
查看对象数据的属性信息
[cephadm@admin ~]$ rados -p default.rgw.buckets.data listxattr 09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7_linux/issue
user.rgw.acl
user.rgw.content_type
user.rgw.etag
user.rgw.idtag
user.rgw.manifest
user.rgw.pg_ver
user.rgw.source_zone
user.rgw.storage_class
user.rgw.tail_tag
user.rgw.x-amz-content-sha256
user.rgw.x-amz-date
user.rgw.x-amz-meta-s3cmd-attrs
```

```
获取对象的manifest的信息
[cephadm@admin ~]$ rados -p default.rgw.buckets.data getxattr 09d0b6f1-fc49-4838-bda1-08cc60553e29.15577.7_linux/issue user.rgw.manifest > manifest
查看信息详情
[cephadm@admin ~]$ ceph-dencoder type RGWObjManifest import manifest decode dump_json
{
    "objs": [],
    ...
```

**小结**
