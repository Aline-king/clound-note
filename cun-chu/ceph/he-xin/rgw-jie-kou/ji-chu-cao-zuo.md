# 基础操作

部署RGW主机

```
到stor04主机上的部署ceph环境
# yum install -y ceph-radosgw
```

```
admin节点上，创建rgw的主机环境
$ ceph-deploy rgw create stor04
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephadm/.cephdeploy.conf
...
[ceph_deploy.rgw][INFO  ] The Ceph Object Gateway (RGW) is now running on host stor04 and default port 7480
```

查看效果

```
查看集群的状态
[cephadm@admin ceph-cluster]$ ceph -s
  cluster:
    ...
  services:
    mon: 3 daemons, quorum mon01,mon02,mon03 (age 3h)
    mgr: mon01(active, since 30h), standbys: mon02
    osd: 6 osds: 6 up (since 26h), 6 in (since 26h)
    rgw: 1 daemon active (stor04)
    ...
```

```
查看服务状况
[root@stor04 ~]# netstat -tnulp | grep radosgw
tcp        0      0 0.0.0.0:7480   0.0.0.0:*  LISTEN      4295/radosgw
tcp6       0      0 :::7480        :::*       LISTEN      4295/radosgw
​
查看服务进程
[root@stor04 ~]# ps aux | grep -v grep | grep radosgw
ceph       4295  0.5  2.1 5145976 39612 ?       Ssl  18:37   0:00 /usr/bin/radosgw -f --cluster ceph --name client.rgw.stor04 --setuser ceph --setgroup ceph
```

```
RGW会在rados集群上生成包括如下存储池的一系列存储池
[cephadm@admin ceph-cluster]$ ceph osd pool ls
.rgw.root
default.rgw.control
default.rgw.meta
default.rgw.log
结果显示：
    默认情况下，rgw自动创建4个存储池
```

浏览器查看 10.0.0.16:7480 的web简单应用效果

<figure><img src="../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## 实践&#x20;

{% tabs %}
{% tab title="修改端口" %}
默认情况下，RGW实例监听于TCP协议的7480端口，需要定制的话，可以通过在运行RGW的节点上编辑其主配置文件ceph.conf进行修改，

```
[root@stor04 ~]# cat /etc/ceph/ceph.conf
...
[client.rgw.stor04]
rgw_frontends = "civetweb port=8080"
```

```
重启服务
[root@stor04 ~]# systemctl restart ceph-radosgw@rgw.stor04
[root@stor04 ~]# systemctl status ceph-radosgw@rgw.stor04
    
查看效果
[root@stor04 ~]# netstat -tnulp | grep 8080
tcp     0   0 0.0.0.0:8080    0.0.0.0:*    LISTEN  152636/radosgw
```

<figure><img src="../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>


{% endtab %}

{% tab title="ssl通信" %}
需要提供pem证书文件，需要将 证书和私钥文件放在同一个文件里面

```
生成秘钥
[root@stor04 ~]# mkdir /etc/ceph/ssl && cd /etc/ceph/ssl
[root@stor04 /etc/ceph/ssl]# openssl genrsa -out civetweb.key 2048
​
生成证书
[root@stor04 /etc/ceph/ssl]# openssl req -new -x509 -key civetweb.key -out civetweb.crt -days 3650 -subj "/CN=stor04.superopsmsb.com"
[root@stor04 /etc/ceph/ssl]# ls
civetweb.crt  civetweb.key
​
合并证书信息
root@stor04:/etc/ceph/ssl# cat civetweb.key civetweb.crt > civetweb.pem
```


{% endtab %}

{% tab title="修改认证信息" %}
```
在运行RGW的节点上编辑其主配置文件ceph.conf进行修改，相关参数如下所示：
[root@stor04 ~]# cat /etc/ceph/ceph.conf
...
[client.rgw.stor04]
rgw_frontends = "civetweb port=8443s ssl_certificate=/etc/ceph/ssl/civetweb.pem" 
```

这里的8443s，最后的s代表必须使用ssl通信方式

```
重启服务
[root@stor04 ~]# systemctl restart ceph-radosgw@rgw.stor04
[root@stor04 ~]# systemctl status ceph-radosgw@rgw.stor04
    
查看效果
[root@stor04 ~]# netstat -tnulp | grep 8443
tcp        0      0 0.0.0.0:8443   0.0.0.0:*     LISTEN      153319/radosgw
```

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>


{% endtab %}

{% tab title="同时开启 https和http方式" %}
在运行RGW的节点上编辑其主配置文件ceph.conf进行修改，相关参数如下所示：

```
[root@stor04 ~]# cat ceph.conf
...
[client.rgw.stor04]
rgw_frontends = "civetweb port=7480+8443s ssl_certificate=/etc/ceph/ssl/civetweb.pem"
```

通过 port+port 的方式实现多端口开启服务

```
重启服务
[root@stor04 ~]# systemctl restart ceph-radosgw@rgw.stor04
[root@stor04 ~]# systemctl status ceph-radosgw@rgw.stor04
    
查看效果
[root@stor04 ~]# netstat -tnulp | grep radosgw
tcp        0      0 0.0.0.0:7480   0.0.0.0:*     LISTEN      153941/radosgw
tcp        0      0 0.0.0.0:8443   0.0.0.0:*     LISTEN      153941/radosgw
```

<figure><img src="../../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>


{% endtab %}

{% tab title="高并发配置" %}
对于ceph来说，它内部的高并发主要是通过线程方式定义的，而且默认值是50，如果我们要调整并发数量的话，可以调整num\_threads的属性值

```
[client.rgw.stor04]
    rgw_frontends = "civetweb port=7480+8443s ssl_certificate=/etc/ceph/ssl/civetweb.pem num_threads=2000"
```


{% endtab %}
{% endtabs %}
