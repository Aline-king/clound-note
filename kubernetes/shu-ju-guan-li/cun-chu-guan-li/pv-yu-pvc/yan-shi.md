# 演示

PV 作为存储资源，主要包括存储能力、访问模式、存储类型、回收策略等关键信息

```
kubectl explain pv.spec
        capacity  定义pv使用多少资源，仅限于空间的设定
        accessModes   访问模式
        存储类型	每种存储类型的样式的属性名称都是专有的。
        persistentVolumeReclaimPolicy	资源回收策略，主要三种Retain、Delete、Recycle
```

pvc是属于名称空间级别的资源对象，也就是说只有特定的资源才能使用

只有 accessModes 和 resources 都匹配成功后，PVC 才会与 PV 绑定在一起。

```
kubectl explain pods.spec.volumes.persistentVolumeClaim
        claimName	定义pvc的名称
        readOnly	设定pvc是否只读
    kubectl  explain pvc.spec
        accessModes 			访问模式	*
        resources  			资源限制	*
        selector 　			标签选择器
        storageClassName  		动态存储名称
        volumeMode  			后端存储卷的模式
        volumeName  			指定卷(pv)的名称
```

```bash
定制资源清单
[root@kubernetes-master1 /data/kubernetes/storage]# vim 04_kubernetes-storage_pv_pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: superopsmsb-pv
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /superopsmsb/nfs-data
    server: 10.0.0.18

---
apiVersion: v1	
kind: PersistentVolumeClaim
metadata:
  name: superopsmsb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
      
创建资源对象
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  apply -f 04_kubernetes-storage_pv_pvc.yml
persistentvolume/superopsmsb-pv created
persistentvolumeclaim/superopsmsb-pvc created
```

```
检查效果
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   REASON   AGE
superopsmsb-pv   3Gi        RWO            Retain           Bound    default/superopsmsb-pvc                           36s
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl get pvc
NAME              STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
superopsmsb-pvc   Bound    superopsmsb-pv   3Gi        RWO                           39s
结果显示：
	PVC 和 PV 已经绑定到一起了
```

```
定制资源清单
[root@kubernetes-master1 /data/kubernetes/storage]# vim 05_kubernetes-storage-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: superopsmsb-nginx
spec:
  volumes:
    - name: nginx-volume
      persistentVolumeClaim:
        claimName: superopsmsb-pvc
  containers:
    - name: nginx-web
      image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
      volumeMounts:
      - name: nginx-volume
        mountPath: "/usr/share/nginx/html"
    属性解析：
    	spec.volumes 是针对pod资源申请的存储资源来说的，我们这里使用的主要是pvc的方式。
    	spec.containers.volumeMounts 是针对pod资源对申请到的存储资源的具体使用信息。
		这里将本地的nfs目录挂载到

创建资源对象
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  apply -f 05_kubernetes-storage-pod.yml
pod/superopsmsb-nginx created
```

```
查看Pod对象信息
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl describe pod superopsmsb-nginx
Name:		superopsmsb-nginx
...
Containers:
  nginx-pv:
    ...
    Mounts:
      /usr/share/nginx/html from nginx-volume (rw)
... 
Volumes:
  nginx-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  superopsmsb-pvc
...

查看pod地址
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  get pod -o wide
NAME                   READY   ...  IP           NODE              ...
superopsmsb-nginx      1/1     ...  10.244.3.5   kubernetes-node3  ...
```

```
浏览器访问效果
[root@kubernetes-master1 /data/kubernetes/storage]# curl 10.244.3.5
Hello Nginx, superopsmsb-nginx-1.23.0

查看nfs地址效果
[root@kubernetes-ha1 ~]# cat /superopsmsb/nfs-data/index.html
Hello Nginx, superopsmsb-nginx-1.23.0
结果显示：
	默认将nginx的首页存放到了nfs中

修改nfs的首页内容
[root@kubernetes-ha1 ~]# echo 'Hello NFS Nginx Website' > /superopsmsb/nfs-data/index.html

浏览器访问效果
[root@kubernetes-master1 /data/kubernetes/storage]# curl 10.244.3.5
Hello NFS Nginx Website
```
