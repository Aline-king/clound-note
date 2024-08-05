# Swift测试

## Swift API接口简介

Swift的用户账号对应于radosgw中的subuser（子用户），它隶属于某个事先存在的user（用户账号）

```bash
radosgw-admin user create --uid="swiftuser" --display-name="Swift Testing User"
radosgw-admin subuser create --uid=swiftuser --subuser=swiftuser:swift --access=full
```

{% hint style="info" %}
Swift API的上下文中，存储桶以<mark style="color:purple;">**container**</mark>表示，而非S3中的bucket，但二者在功用上类同，都是<mark style="color:orange;">**对象数据的容器**</mark>
{% endhint %}

Python Swiftclient 是一个用于与Swift API交互的Python客户端程序，它包含了Python API（swift 模块）和一个命令行工具swift.

```
环境安装如下
    pip instal --upgrade python-swiftclient
​
swift命令可以通过Swift API完成容器和对象数据的管理操作，其基础语法格式为“
    swift [-A Auth URL] [-U username] [-K password] subcommand
```

**简单实践**

用户创建

```
swift的实践用户，我们直接使用上面s3创建的用户来创建
[cephadm@admin ceph-cluster]$ radosgw-admin user list
[
    "s3user"
]
```

```
基于s3user用户创建swift拥有所有权限的用户
[cephadm@admin ceph-cluster]$ radosgw-admin subuser create --uid s3user --subuser=s3user:swift --access=full
{
    "user_id": "s3user",
    ...
    "subusers": [
        {
            "id": "s3user:swift",
            "permissions": "full-control"
        }
    ],
    "keys": [
        {
            "user": "s3user",
            "access_key": "BX2IDCO5ADZ8IWZ4AVX7",
            "secret_key": "wKXAiBBspTx9kl1NpcAi4eOCHxvQD7ORHnrmvBlf"
        }
    ],
    "swift_keys": [
        {
            "user": "s3user:swift",
            "secret_key": "pZ5DGT3YDBqdAtb10aFiNGeO2AtJKJN4rLKuI4wU"
        }
    ],
    ...
}
```

生成 s3user:swift 对应的secret\_key

```

[cephadm@admin ceph-cluster]$ radosgw-admin key create --subuser=s3user:swift --key-type=swift --gen-secret
{
    "user_id": "s3user",
    "display_name": "S3 Testing user",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [
        {
            "id": "s3user:swift",
            "permissions": "full-control"
        }
    ],
    "keys": [
        {
            "user": "s3user",
            "access_key": "BX2IDCO5ADZ8IWZ4AVX7",
            "secret_key": "wKXAiBBspTx9kl1NpcAi4eOCHxvQD7ORHnrmvBlf"
        }
    ],
    "swift_keys": [
        {
            "user": "s3user:swift",
            "secret_key": "O7On31ApRWE6aZkSPby9bTp4y19srGTfDKl7IA04"
        }
    ],
    ...
}
```

客户端环境

{% hint style="danger" %}
如果需要安装最新版本的话，需要提前将Python版本提升到3.6+ https://pypi.org/project/python-swiftclient/
{% endhint %}

```
安装swift命令
[root@stor04 ~]# yum -y install python-setuptools python-pip
[root@stor04 ~]# pip install python-swiftclient==3.4.0
```

`swift -A 认证URL/auth -U 用户名称:swift -K secret_key 命令`

```
​命令测试效果
[root@stor04 ~]# swift -A http://10.0.0.16:7480/auth -U s3user:swift -K O7On31ApRWE6aZkSPby9bTp4y19srGTfDKl7IA04 list
ceph-s3-bucket
```

```
配置环境变量
cat > ~/.swift << EOF
export ST_AUTH=http://10.0.0.16:7480/auth
export ST_USER=s3user:swift
export ST_KEY=O7On31ApRWE6aZkSPby9bTp4y19srGTfDKl7IA04
EOF
```

```
加载环境变量
[root@stor04 ~]# source ~/.swift
​
确认效果
[root@stor04 ~]# swift list
ceph-s3-bucket
```

```
查看状态信息
[root@stor04 ~]# swift stat
                                    Account: v1
                                 Containers: 2
                                    Objects: 1
                                      Bytes: 23
Objects in policy "default-placement-bytes": 0
  Bytes in policy "default-placement-bytes": 0
   Containers in policy "default-placement": 2
      Objects in policy "default-placement": 1
        Bytes in policy "default-placement": 23
                     X-Openstack-Request-Id: tx00000000000000000012c-0063324b45-63ad-default
                X-Account-Bytes-Used-Actual: 4096
                                 X-Trans-Id: tx00000000000000000012c-0063324b45-63ad-default
                                X-Timestamp: 1664240453.17581
                               Content-Type: text/plain; charset=utf-8
                              Accept-Ranges: bytes
```

## 各种测试

{% tabs %}
{% tab title="存储桶增删测试" %}
```
创建存储桶
[root@stor04 ~]# swift post swift-super
​
删除存储桶
[root@stor04 ~]# swift delete ceph-s3-bucket
ceph-s3-bucket
​
查看存储桶
[root@stor04 ~]# swift list
swift-super
```


{% endtab %}

{% tab title="文件上传操作" %}
```
上传文件
[root@stor04 ~]# swift upload swift-super /etc/passwd
etc/passwd
​
查看效果
[root@stor04 ~]# swift list swift-super
etc/passwd
```


{% endtab %}

{% tab title="目录上传操作" %}
```
上传目录操作
[root@stor04 ~]# swift upload swift-super /data/test
data/test/scripts/4.sh
...
data/test/file/3.txt
​
查看效果
[root@stor04 ~]# swift list swift-super
data/test/conf/1.conf
...
etc/passwd
```


{% endtab %}

{% tab title="文件下载操作" %}
```
下载文件
[root@stor04 ~]# swift download swift-super etc/passwd
etc/passwd [auth 0.014s, headers 0.023s, total 0.024s, 0.120 MB/s]
[root@stor04 ~]# ls etc/passwd
etc/passwd
​
下载所有
[root@stor04 ~]# swift download --all
swift-super/data/test/file/4.txt [auth 0.008s, headers 0.019s, total 0.019s, 0.000 MB/s]
...
它对于目录的递归下载不是太友好
```


{% endtab %}

{% tab title="文件删除操作" %}
```
删除文件
[root@stor04 ~]# swift delete swift-super etc/passwd
etc/passwd
它不支持目录删除，但是可以通过批量删除来实现
​
批量删除
[root@stor04 ~]# swift delete swift-super --prefix=data
data/test/file/4.txt
...
data/test/scripts/1.sh
​
批量删除
[root@stor04 ~]# swift delete swift-super --all
```


{% endtab %}
{% endtabs %}



**小结**
