# RC\&RS

简介

Replication Controller（RC），是kubernetes系统中的核心概念之一。

RC是Kubernetes集群实现Pod资源对象自动化管理的基础。

​ 简单来说，RC其实是定义了一个期望的场景，RC有以下特点：

&#x20;1、组成：定义了Pod副本的期望状态：包括数量，筛选标签和模板 Pod期待的副本数量(replicas). 永远筛选目标Pod的标签选择器(Label Selector) Pod数量不满足预期值，自动创建Pod时候用到的模板(template)

&#x20;2、意义：自动监控Pod运行的副本数目符合预期，保证Pod高可用的核心组件，常用于Pod的生命周期管理 ​&#x20;

RC在kubernetes的早期kubernetes v1.2版本就被RS(ReplicaSet)替代了，但是RC目前仍然能够正常使用，因为RC的核心功能不仅仅是RS的核心，也是Deployment资源对象的核心。 RC 和 RS 的区别仅仅是资源对象名称和标签选择器不一样而已。

### 属性解读

{% hint style="info" %}
注意：

RS和RC之间selector的格式区别 &#x20;

RC 的标签匹配是 key：value  &#x20;

RS 的标签匹配是 matchExpressions | matchLabels
{% endhint %}

```yaml
apiVersion: apps/v1
kind: ReplicaSet | ReplicationController
metadata:
  name: …
  namespace: …
spec:
  minReadySeconds <integer>         # Pod就绪后多少秒内，Pod任一容器无crash方可视为“就绪”
  replicas <integer>                # 期望的Pod副本数，默认为1
  selector:                         # 标签选择器，必须匹配template字段中Pod模板中的标签；
    matchExpressions <[]Object>     # 标签选择器表达式列表，多个列表项之间为“与”关系
    matchLabels <map[string]string> # map格式的标签选择器
  template:                         # Pod模板对象
    metadata:                       # Pod对象元数据
      labels:                       # 由模板创建出的Pod对象所拥有的标签，必须要能够匹配前面定义的标签选择器
    spec:                           # Pod规范，格式同自主式Pod
      ……
```

当我们通过"资源定义文件"定义好了一个RC资源对象，把它提交到Kubernetes集群中以后，Master节点上的Controller Manager组件就得到通知(问：为什么？因为什么？)，定期巡检系统中当前存活的Pod，并确保Pod实例数量刚到满足RC的期望值。&#x20;

如果Pod数量大于RC定义的期望值，那么就杀死一些Pod&#x20;

如果Pod数量小于RC定义的期望值，那么就创建一些Pod&#x20;

所以：通过RC资源对象，Kubernetes实现了业务应用集群的高可用性，大大减少了人工干预，提高了管理的自动化。 拓展： 想要扩充Pod副本的数量，可以直接修改replicas的值即可

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**简单实践**

```
创建资源目录
[root@kubernetes-master1 ~]# mkdir /data/kubernetes/controller -p
[root@kubernetes-master1 ~]# cd /data/kubernetes/controller/
[root@kubernetes-master1 /data/kubernetes/controller]#
```

```
创建资源清单文件
[root@kubernetes-master1 /data/kubernetes/controller]# cat 01_kubernetes_rc_test.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: superopsmsb-rc
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:   
      containers:
      - name: nginx-web
        image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
    
应用资源清单文件
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  apply -f 01_kubernetes_rc_test.yaml
replicationcontroller/superopsmsb-rc created
```

```
查看效果
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get rc
NAME             DESIRED   CURRENT   READY   AGE
superopsmsb-rc   2         2         2       34s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get pod
NAME                   READY   STATUS    RESTARTS   AGE
superopsmsb-rc-pxblj   1/1     Running   0          3s
superopsmsb-rc-qtv4f   1/1     Running   0          3s
```

RS实践

```
创建资源清单文件
[root@kubernetes-master1 /data/kubernetes/controller]# cat 02_kubernetes_rs_test.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: superopsmsb-rs
spec:
  minReadySeconds: 0
  replicas: 3
  selector:
    matchLabels:
      app: rs-test
  template:
    metadata:
      labels:
        app: rs-test
    spec:
      containers:
      - name: nginx-web
        image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
    
应用资源清单文件
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  apply -f 02_kubernetes_rs_test.yaml
replicaset.apps/superopsmsb-rs created
```

```
查看效果
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get rs
NAME             DESIRED   CURRENT   READY   AGE
superopsmsb-rs   3         3         3       2s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  get pod -l app=rs-test
NAME                   READY   STATUS    RESTARTS   AGE
superopsmsb-rs-csjs5   1/1     Running   0          23s
superopsmsb-rs-d6wb7   1/1     Running   0          23s
superopsmsb-rs-n7k6w   1/1     Running   0          23s
```

控制实践

```
删除pod
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  delete pod -l app=nginx
pod "superopsmsb-rc-pxblj" deleted
pod "superopsmsb-rc-qtv4f" deleted
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  delete pod -l app=rs-test
pod "superopsmsb-rs-csjs5" deleted
pod "superopsmsb-rs-d6wb7" deleted
pod "superopsmsb-rs-n7k6w" deleted
​
查看删除后效果
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get rc
NAME             DESIRED   CURRENT   READY   AGE
superopsmsb-rc   2         2         2       5m4s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get rs
NAME             DESIRED   CURRENT   READY   AGE
superopsmsb-rs   3         3         3       3m33s
​
检查pod效果
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get pod
NAME                   READY   STATUS    RESTARTS   AGE
superopsmsb-rc-bsmqg   1/1     Running   0          99s
superopsmsb-rc-mzv9s   1/1     Running   0          99s
superopsmsb-rs-7fgr8   1/1     Running   0          46s
superopsmsb-rs-8k9kd   1/1     Running   0          46s
superopsmsb-rs-gnxp8   1/1     Running   0          46s
```

环境清理

```
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  delete -f 01_kubernetes_rc_test.yaml -f 02_kubernetes_rs_test.yaml
replicationcontroller "superopsmsb-rc" deleted
replicaset.apps "superopsmsb-rs" deleted
```

```
```
