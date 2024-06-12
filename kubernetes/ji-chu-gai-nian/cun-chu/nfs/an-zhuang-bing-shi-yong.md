# 安装并使用

NFS环境准备

```YAML
我们将10.0.0.18作为NFS服务环境，这里加载一块磁盘并进行格式化
[root@kubernetes-ha1 ~]# fdisk -l | grep 'sdb'
磁盘 /dev/sdb：21.5 GB, 21474836480 字节，41943040 个扇区

磁盘格式化
[root@kubernetes-ha1 ~]# mkfs.ext4 /dev/sdb
mke2fs 1.42.9 (28-Dec-2013)
[root@kubernetes-ha1 ~]# blkid | grep sdb
/dev/sdb: UUID="b01ca48c-f8a6-4817-a9b4-7bb7a53c03c2" TYPE="ext4"

创建数据目录
[root@kubernetes-ha1 ~]# mkdir /superopsmsb/nfs-data -p
```

```YAML
部署nfs服务
[root@kubernetes-ha1 ~]# yum install -y rpcbind nfs-utils

配置共享目录
echo '/superopsmsb/nfs-data *(rw,no_root_squash,sync)' > /etc/exports
注意：
         *：表示允许所有连接，该值可以是一个网段|IP|域名的形式
         rw 表示读写共享权限
         no_root_squash 表示root登录nfs资源的时候，虽然是以nobody身份，单不压缩其权限
         sync         所有操作数据会同时写入硬盘和内存
         
配置生效
[root@kubernetes-ha1 ~]# exportfs -r

重启服务
[root@kubernetes-ha1 ~]# systemctl start rpcbind.service
[root@kubernetes-ha1 ~]# systemctl start nfs.service
[root@kubernetes-ha1 ~]# systemctl enable rpcbind.service nfs.service

确认效果
[root@kubernetes-ha1 ~]# showmount -e localhost
Export list for localhost:
/superopsmsb/nfs-data *
```

```YAML
所有的k8s节点部署nfs命令
[root@kubernetes-master1 ~]# for i in {12..17};do ssh root@10.0.0.$i "yum install nfs-utils -y";done

确认效果
[root@kubernetes-master1 ~]# showmount -e 10.0.0.18
Export list for 10.0.0.18:
/superopsmsb/nfs-data *

挂载测试
[root@kubernetes-master1 ~]# mount -t nfs 10.0.0.18:/superopsmsb/nfs-data /tmp
[root@kubernetes-master1 ~]# mount | grep nfs
10.0.0.18:/superopsmsb/nfs-data on /tmp type nfs4 (rw,relatime,vers=4.1,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.0.0.12,local_lock=none,addr=10.0.0.18)
[root@kubernetes-master1 ~]# umount /tmp
```

使用举例

```YAML
准备资源清单文件
[root@kubernetes-master1 /data/kubernetes/storage]# cat 03_kubernetes-storage_nfs.yml 
apiVersion: v1
kind: Pod
metadata:
  name: superopsmsb-nfs
spec:
  volumes:
    - name: redis-backup
      nfs:
        server: 10.0.0.18
        path: /superopsmsb/nfs-data
  containers:
    - name: nfs-redis
      image: kubernetes-register.superopsmsb.com/superopsmsb/redis:7.0.4
      volumeMounts:
       - name: redis-backup
         mountPath: /data
```

```YAML
创建资源对象
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  apply -f 03_kubernetes-storage_nfs.yml
pod/superopsmsb-nfs created

[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  get pod -o wide
NAME                   READY   ...  IP           NODE              ...
superopsmsb-nfs        1/1     ...  10.244.3.4   kubernetes-node3  ...
```

```YAML
查看描述信息
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl describe pod superopsmsb-nfs
Name:         superopsmsb-emptydir
...
Containers:
   nfs-redis:
    ...
    Mounts:
      /data from redis-backup (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pfz9h (ro)
...
Volumes:
  redis-backup:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    10.0.0.18
    Path:      /superopsmsb/nfs-data
```

```YAML
进入容器确认效果
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl exec -it superopsmsb-nfs -- /bin/bash
root@superopsmsb-nfs:/data# redis-cli
127.0.0.1:6379> set volumes nfs
OK
127.0.0.1:6379> BGSAVE
Background saving started
127.0.0.1:6379> exit
root@superopsmsb-nfs:/data# exit
exit

到nfs上看效果
[root@kubernetes-master1 /data/kubernetes/storage]# ssh root@10.0.0.18 ls /superopsmsb/nfs-data
dump.rdb
```
