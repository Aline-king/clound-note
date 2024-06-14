---
description: 特点：成本高，可控性高，维护难度大。
---

# 独立的etcd集群

使用3台或者5台服务器只运行etcd，独立维护和升级。

etcd集群将作为数据存储基础设施用于构建整个集群。

etcd集群的节点增减需要显式的通知整个集群，保证etcd集群节点稳定可以更方便的用程序完成集群滚动升级，减轻维护负担。

## 部署



{% hint style="info" %}
准备证书

CFSSL是CloudFlare开源的一款PKI/TLS工具。 CFSSL 包含一个命令行工具 和一个用于 签名，验证并且捆绑TLS证书的 HTTP API 服务。 使用Go语言编写。

代码地址： [https://github.com/cloudflare/cfssl](https://github.com/cloudflare/cfssl)

官网地址： [https://pkg.cfssl.org/](https://pkg.cfssl.org/)
{% endhint %}



{% tabs %}
{% tab title="准备环境" %}
所有节点破坏kubernetes集群环境

```bash
kubeadm reset 
rm -f /etc/cni/net.d/* 
reboot
```

## 环境

```
mkdir /data/kubernetes/etcd ; cd /data/kubernetes/etcd 
for i in cfssl cfssljson
do
  wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/$i_1.6.1_linux_amd64
  chmod +x $i_1.6.1_linux_amd64
  mv $i_1.6.1_linux_amd64 /usr/local/bin/$i
done
```

{% hint style="info" %}
创建ca证书，客户端，服务端，节点之间的证书

* ca证书 自己给自己签名的权威证书，用来给其他证书签名
* server证书 etcd的证书
* client证书 客户端，比如etcdctl的证书
* peer证书 节点与节点之间通信的证书
{% endhint %}


{% endtab %}

{% tab title="创建文件" %}
```
[root@kubernetes-master1 ~]# mkdir -p /etc/etcd/ssl
[root@kubernetes-master1 ~]# cd /etc/etcd/ssl
```

## 生成证书初始化模板文件

server auth表示client可以用该ca对server提供的证书进行验证

client auth表示server可以用该ca对client提供的证书进行验证

```bash
[root@kubernetes-master1 /etc/etcd/ssl]# cfssl print-defaults config
{
    "signing": {
        "default": {
            "expiry": "168h"
        },
        "profiles": {
            "www": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            }
        }
    }
}
```

## 生成证书签名请求初始化模板文件

```bash
[root@kubernetes-master1 /etc/etcd/ssl]# cfssl print-defaults csr
{
    "CN": "example.net",
    "hosts": [
        "example.net",
        "www.example.net"
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "US",
            "ST": "CA",
            "L": "San Francisco"
        }
    ]
}
```

## 定制ca证书文件 ca-config.json

需要定义server、client、peer 三种角色的认证通信

```json
{
    "signing": {
        "default": {
            "expiry": "438000h"
        },
        "profiles": {
            "server": {
                "expiry": "438000h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            },
            "client": {
                "expiry": "438000h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "438000h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```
{% endtab %}

{% tab title="证书设置" %}
## 定制前面请求文件 ca-csr.json

只需要定制算法和名称即可

```
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}
```

## 生成私钥和证书

```
[root@kubernetes-master1 /etc/etcd/ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca

...

110542548803478808677950627582407176512561912812

[root@kubernetes-master1 /etc/etcd/ssl]# ls

ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

## 生成客户端证书

## 定制专属配置文件 client.json

```
{
  "CN": "client",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BJ",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

## 生成证书

```
[root@kubernetes-master1 /etc/etcd/ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client.json  | cfssljson -bare client -
...
specifically, section 10.2.3 ("Information Requirements").

查看效果
[root@kubernetes-master1 /etc/etcd/ssl]# ls
ca-config.json  ca-csr.json  ca.pem      client.json     client.pem
ca.csr          ca-key.pem   client.csr  client-key.pem
```

## 定制etcd节点间证书

定制集群节点间证书文件 etcd.json

```
{
  "CN": "etcd",
  "hosts": [
    "10.0.0.12",
    "10.0.0.13",
    "10.0.0.14",
    "127.0.0.1"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BJ",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

## 生成服务端通信专属的证书

```
[root@kubernetes-master1 /etc/etcd/ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server etcd.json | cfssljson -bare server
```

```
[root@kubernetes-master1 /etc/etcd/ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer etcd.json | cfssljson -bare peer
```

```
确认效果
[root@kubernetes-master1 /data/etcd/ssl]# ls
ca-config.json  ca-key.pem  client.json     etcd.json     peer.pem        server.pem
ca.csr          ca.pem      client-key.pem  peer.csr      server.csr
ca-csr.json     client.csr  client.pem      peer-key.pem  server-key.pem
```
{% endtab %}

{% tab title="安装etcd" %}
```
获取软件
cd /data/softs
wget https://github.com/etcd-io/etcd/releases/download/v3.5.2/etcd-v3.5.2-linux-amd64.tar.gz

解压文件到命令目录
tar zxf etcd-v3.5.2-linux-amd64.tar.gz
cp etcd-v3.5.2-linux-amd64/{etcd etcdctl} /usr/local/bin/
rm -rf etcd-v3.5.2-linux-amd64
```

## 准备服务文件

```
[root@kubernetes-master1 ~]# vim /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=neCNork.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
--name=kubernetes-master1 \
--data-dir=/var/lib/etcd/default.etcd \
--listen-peer-urls=https://10.0.0.12:2380 \
--listen-client-urls=https://10.0.0.12:2379,http://127.0.0.1:2379 \
--advertise-client-urls=https://10.0.0.12:2379 \
--initial-advertise-peer-urls=https://10.0.0.12:2380 \
--initial-cluster=kubernetes-master1=https://10.0.0.12:2380,kubernetes-master2=https://10.0.0.13:2380,kubernetes-master3=https://10.0.0.14:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster-state=new \
--cert-file=/etc/etcd/ssl/server.pem \
--key-file=/etc/etcd/ssl/server-key.pem \
--peer-cert-file=/etc/etcd/ssl/peer.pem \
--peer-key-file=/etc/etcd/ssl/peer-key.pem \
--trusted-ca-file=/etc/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/etc/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
```

## 同步所有文件到所有etcd节点

```
 for i in 13 14
do
  ssh root@10.0.0.$i mkdir -p /etc/etcd
  scp -r /etc/etcd/* root@10.0.0.$i:/etc/etcd/
  scp /usr/lib/systemd/system/etcd.service root@10.0.0.$i:/usr/lib/systemd/system/etcd.service
done

master2节点修改服务启动文件
[root@kubernetes-master2 ~]# sed -i '/name/s/master1/master2/' /usr/lib/systemd/system/etcd.service
[root@kubernetes-master2 ~]# sed -i '/urls/s/0.12/0.13/g' /usr/lib/systemd/system/etcd.service
[root@kubernetes-master2 ~]# egrep 'name|urls' /usr/lib/systemd/system/etcd.service
  --name=kubernetes-master2 \
  --initial-advertise-peer-urls=https://10.0.0.13:2380 \
  --listen-peer-urls=https://10.0.0.13:2380 \
  --listen-client-urls=https://10.0.0.13:2379 \
  --advertise-client-urls=https://10.0.0.13:2379 \
```
{% endtab %}

{% tab title="master更改" %}
## 3个master节点修改服务启动文件

```
[root@kubernetes-master3 ~]# sed -i '/name/s/master1/master3/' /usr/lib/systemd/system/etcd.service
[root@kubernetes-master3 ~]# sed -i '/urls/s/0.12/0.14/g' /usr/lib/systemd/system/etcd.service
[root@kubernetes-master3 ~]# egrep 'name|urls' /usr/lib/systemd/system/etcd.service
  --name=kubernetes-master3 \
  --initial-advertise-peer-urls=https://10.0.0.14:2380 \
  --listen-peer-urls=https://10.0.0.14:2380 \
  --listen-client-urls=https://10.0.0.14:2379 \
  --advertise-client-urls=https://10.0.0.14:2379 \
```

## 启动

```
启动etcd服务
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
systemctl status etcd
```

```
检查效果
cd /etc/etcd/ssl

查看集群状态
etcdctl --endpoints="https://10.0.0.12:2379,https://10.0.0.13:2379,https://10.0.0.14:2379," --cacert=ca.pem --cert=server.pem --key=server-key.pem endpoint status

查看集群主机
etcdctl --endpoints="https://10.0.0.12:2379,https://10.0.0.13:2379,https://10.0.0.14:2379," --cacert=ca.pem --cert=server.pem --key=server-key.pem member list
```


{% endtab %}

{% tab title="k8s改造" %}
## 传输证书

```bash
etcd集群的ca证书
cp /etc/etcd/ssl/ca.pem /etc/kubernetes/pki/etcd/ca.pem

etcd集群的client证书，apiserver访问etcd使用
cp /etc/etcd/ssl/client.pem /etc/kubernetes/pki/apiserver-etcd-client.pem

etcd集群的client私钥
cp /etc/etcd/ssl/client-key.pem /etc/kubernetes/pki/apiserver-etcd-client-key.pem
```

## 改造配置文件

```
[root@kubernetes-master1 /etc/kubernetes]# cat /data/kubernetes/cluster_init/kubeadm_init_1.23.8.yml
...
# 原内容
# etcd:
#   local:
#     dataDir: /var/lib/etcd
# 修改后内容
etcd: 
  external:
    endpoints:
    - https://10.0.0.12:2379
    - https://10.0.0.13:2379
    - https://10.0.0.14:2379
    caFile: /etc/kubernetes/pki/etcd/ca.pem
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.pem
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client-key.pem
...
```

集群初始化

```bash
kubeadm init --config=kubeadm-init.yaml 
```
{% endtab %}
{% endtabs %}
