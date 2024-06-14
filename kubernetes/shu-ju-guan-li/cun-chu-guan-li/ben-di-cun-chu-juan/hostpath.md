# hostPath

hostPath将宿主机上的目录挂载到Pod中作为数据的存储目录，一般用在如下场景：

## 挂载演示

```
 获取互联网镜像
[root@kubernetes-master1 ~]# docker pull redis
[root@kubernetes-master1 ~]# docker history redis | grep REDIS_VER
<missing>      3 days ago    /bin/sh -c #(nop)  ENV REDIS_VERSION=7.0.4      0B
[root@kubernetes-master1 ~]# docker tag redis:latest kubernetes-register.superopsmsb.com/superopsmsb/redis:7.0.4
[root@kubernetes-master1 ~]# docker push kubernetes-register.superopsmsb.com/superopsmsb/redis:7.0.4
[root@kubernetes-master1 ~]# docker rmi redis:latest

创建所有主机的备份目录
[root@kubernetes-master1 ~]# for i in 15 16 17;do ssh root@10.0.0.$i "mkdir /data/backup/redis -p"; done
```

定制redis的数据备份功能

```
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
         mountPath: /data
	注意：
	redis的镜像将数据保存到了容器的/data 目录下。
```

```
创建资源对象
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  apply -f 02_kubernetes-storage_hostpath.yml
pod/superopsmsb-hostpath created

[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  get pod -o wide
NAME                   READY   ...  IP           NODE              ...
superopsmsb-hostpath   1/1     ...  10.244.3.3   kubernetes-node3  ...

查看描述信息
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
