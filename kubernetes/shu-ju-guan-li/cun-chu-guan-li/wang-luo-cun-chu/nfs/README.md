---
description: network file system
---

# nfs

## 属性解析

{% code title="配置属性" %}
```bash
# kubectl explain pod.spec.volumes.nfs
```
{% endcode %}

> * path 指定nfs服务器暴露的共享地址
> * readOnly 是否只能读，默认false
> * server 指定nfs服务器的地址

{% hint style="danger" %}
注意：&#x20;

如果安装 nfs，则要求k8s所有集群都必须支持nfs命令，

即所有结点都必须执行 <mark style="color:purple;">**yum install nfs-utils -y**</mark>
{% endhint %}

{% code title="配置格式" %}
```yaml
  volumes:
  - name: 卷名称
    nfs:
     server: nfs_server_address
     path: "共享目录"
```
{% endcode %}

NFS环境准备
