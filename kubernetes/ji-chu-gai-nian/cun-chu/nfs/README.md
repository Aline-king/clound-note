# NFS

`kubectl explain pod.spec.volumes.nfs`&#x20;

{% hint style="info" %}
要求k8s所有集群都必须支持nfs命令，即所有结点都必须执行 yum install nfs-utils -y
{% endhint %}

```yaml
volumes:
  - name: 卷名称
    nfs:
     server: nfs_server_address      # server 指定nfs服务器的地址
     path: "共享目录"                 # path 指定nfs服务器暴露的共享地址 
                                     # readOnly 是否只能读，默认false 
```

