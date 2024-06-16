# hostPath



hostPath将宿主机上的目录挂载到Pod中作为数据的存储目录

hostPath使用注意事项：

&#x20;1、不同宿主机的文件内容不一定完全相同，所以Pod迁移前后的访问效果不一样&#x20;

2、宿主机的目录不属于资源对象的资源，所以我们对资源设置的资源配额限制对hostPath目录无效

## 配置属性

{% embed url="https://kubernetes.io/docs/concepts/storage/volumes#hostpath" %}

{% code title="配置格式" %}
```yaml
  volumes:
  - name: volume_name
    hostPath:
     path: /path/to/host
```
{% endcode %}

{% code title="配置属性解读" %}
```
    kubectl explain pod.spec.volumes.hostPath
    path  指定宿主机的路径
    type  指定路径的类型，一共有7种，默认的类型是没有指定的话则自己创建
        DirectoryOrCreate  宿主机上不存在，创建此0755权限的空目录，属主属组均为kubelet  
        Directory 必须存在挂载已存在的目录，最常用
        FileOrCreate 宿主机上不存在挂载文件，就创建0644权限的空文件，属主和属组同为kubelet  
        File 必须存在文件 
        Socket 事先必须存在的Socket文件路径
        CharDevice 事先必须存在的字符设备文件路径
        BlockDevice 事先必须存在的块设备文件路径
```
{% endcode %}

## 挂载演示

{% tabs %}
{% tab title="准备" %}
{% code title="获取互联网镜像" %}
```bash
[root@kubernetes-master1 ~]# docker pull redis
[root@kubernetes-master1 ~]# docker history redis | grep REDIS_VER
<missing>      3 days ago    /bin/sh -c #(nop)  ENV REDIS_VERSION=7.0.4      0B
[root@kubernetes-master1 ~]# docker tag redis:latest kubernetes-register.superopsmsb.com/superopsmsb/redis:7.0.4
[root@kubernetes-master1 ~]# docker push kubernetes-register.superopsmsb.com/superopsmsb/redis:7.0.4
[root@kubernetes-master1 ~]# docker rmi redis:latest

创建所有主机的备份目录
[root@kubernetes-master1 ~]# for i in 15 16 17;do ssh root@10.0.0.$i "mkdir /data/backup/redis -p"; done
```
{% endcode %}


{% endtab %}

{% tab title="定制redis的数据备份功能" %}
```yaml
 准备资源清单文件
[root@kubernetes-master1 /data/kubernetes/storage]# cat 02_kubernetes-storage_hostpath.yml 
apiVersion: v1
kind: Pod
metadata:
  name: superopsmsb-hostpath
spec:
  volumes:
  - name: redis-backup
    hostPath:
     path: /data/backup/redis
  containers:
    - name: hostpath-redis
      image: kubernetes-register.superopsmsb.com/superopsmsb/redis:7.0.4
      volumeMounts:
       - name: redis-backup
         mountPath: /data       # redis的镜像将数据保存到了容器的/data 目录下
	
```


{% endtab %}

{% tab title="创建" %}
```
创建资源对象
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  apply -f 02_kubernetes-storage_hostpath.yml
pod/superopsmsb-hostpath created

[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  get pod -o wide
NAME                   READY   ...  IP           NODE              ...
superopsmsb-hostpath   1/1     ...  10.244.3.3   kubernetes-node3  ...
```

```
查看描述信息
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl describe pod superopsmsb-emptydir
Name:         superopsmsb-emptydir
...
Containers:
   hostpath-redis:
    ...
    Mounts:
      /data from redis-backup (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8x257 (ro)
...
Volumes:
  redis-backup:
    Type:          HostPath (bare host directory volume)
    Path:          /data/backup/redis
    ...
```


{% endtab %}

{% tab title="查看效果" %}
```
进入容器确认效果
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl exec -it superopsmsb-hostpath -- /bin/bash
root@superopsmsb-hostpath:/data# redis-cli
127.0.0.1:6379> set volume hostpath
OK
127.0.0.1:6379> BGSAVE
Background saving started
127.0.0.1:6379> exit
root@superopsmsb-hostpath:/data# exit
exit

到指定的node3上看效果
[root@kubernetes-master1 /data/kubernetes/storage]# ssh root@10.0.0.17 ls /data/backup/redis
dump.rdb
```
{% endtab %}
{% endtabs %}

