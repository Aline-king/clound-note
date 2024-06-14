# emptyDir

## 定义

一个emptyDir volume在pod被调度到某个Node时候自动创建的，无需指定宿主机上对应的目录。

特点如下：

&#x20;1、跟随Pod初始化而来，开始是空数据卷&#x20;

2、Pod移除，emptyDir中数据随之永久消除&#x20;

3、emptyDir一般做本地临时存储空间使用&#x20;

4、emptyDir数据卷介质种类跟当前主机的磁盘一样。

<figure><img src="../../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

## 属性解读

关于volume和容器的关系，每个容器都可以单独挂载多个volume，每种volume都可以被多个容器挂载

<pre class="language-bash"><code class="lang-bash"> # kubectl explain pod.spec.volumes.emptyDir
    medium   	指定媒介类型，主要有 disk  Memory(默认) 两种
    sizeLimit  	现在资源使用情况
    
<strong># kubectl explain pod.spec.containers.volumeMounts
</strong>    mountPath   挂载到容器中的路径
    name 		指定挂载的volumes名称
    readOnly   	是否只读挂载
    subPath   	是否挂载子路径
</code></pre>



## 挂载演示

{% tabs %}
{% tab title="准备清单目录" %}
```bash
[root@kubernetes-master1 ~]# mkdir /data/kubernetes/storage -p ;cd /data/kubernetes/storage
[root@kubernetes-master1 /data/kubernetes/storage]#
```


{% endtab %}

{% tab title="准备资源清单文件" %}
```
[root@kubernetes-master1 /data/kubernetes/storage]# cat 01_kubernetes-storage_emptydir.yml 
apiVersion: v1
kind: Pod
metadata:
  name: superopsmsb-emptydir
spec:
  containers:
  - name: nginx-web
    image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
    volumeMounts:
    - name: nginx-index
      mountPath: /usr/share/nginx/html
  - name: change-index
    image: kubernetes-register.superopsmsb.com/superopsmsb/busybox:1.28
    # 每过2秒更改一下文件内容
    command: ['sh', '-c', 'for i in $(seq 100); do echo index-$i > /testdir/index.html;sleep 2;done']
    volumeMounts:
    - name: nginx-index
      mountPath: /testdir
  volumes:
  - name: nginx-index
    emptyDir: {}
```


{% endtab %}

{% tab title="执行资源清单文件" %}
```bash
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  apply -f 01_kubernetes-storage_emptydir.yml
pod/superopsmsb-emptydir created

查看效果
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl  get pod -o wide
NAME                   READY   ...  IP           NODE              ...
superopsmsb-emptydir   2/2     ...  10.244.1.5   kubernetes-node1  ...
[root@kubernetes-master1 /data/kubernetes/storage]# curl 10.244.1.5
index-38
[root@kubernetes-master1 /data/kubernetes/storage]# curl 10.244.1.5
index-39
```


{% endtab %}

{% tab title="查看描述信息" %}
```bash
[root@kubernetes-master1 /data/kubernetes/storage]# kubectl describe pod superopsmsb-emptydir
Name:         superopsmsb-emptydir
...
Containers:
  nginx-web:
    ...
    Mounts:
      /usr/share/nginx/html from nginx-index (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gbwh8 (ro)
  change-index:
    ...
    Mounts:
      /testdir from nginx-index (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gbwh8 (ro)
...
Volumes:
  nginx-index:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    ...
```

文件是随着pod而存在的，一旦pod删除了，文件也丢失了

```
关于临时存储的文件存放的位置在指定节点的/var/lib/kubelet目录下

查找文件
[root@kubernetes-node1 ~]# find /var/lib/kubelet/ -name "index.html"
/var/lib/kubelet/pods/9b086c71-ad40-4c59-8525-17a97ae0a95b/volumes/kubernetes.io~empty-dir/nginx-index/index.html

查看文件
[root@kubernetes-node1 ~]# cat /var/lib/kubelet/pods/9b086c71-ad40-4c59-8525-17a97ae0a95b/volumes/kubernetes.io~empty-dir/nginx-index/index.html
index-46
```


{% endtab %}
{% endtabs %}





