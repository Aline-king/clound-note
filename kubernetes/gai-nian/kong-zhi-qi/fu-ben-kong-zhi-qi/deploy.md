# deploy

## 扩缩容

Deployment资源对象在内部使用Replica Set来实现Pod的自动化编排。

{% hint style="info" %}
{% code title="基于资源对象调整：" %}
```bash
kubectl scale <--current-replicas=3> --replicas=5 deployment/deploy_name 
```
{% endcode %}

{% code title="基于资源文件调整: " %}
```bash
kubectl scale --replicas=4 -f deploy_name.yaml
```
{% endcode %}
{% endhint %}



<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



## 动态更新

{% tabs %}
{% tab title="更新" %}
更新命令：

```bash
kubectl set SUBCOMMAND [options] 资源类型 资源名称   
```

子命令：

常用的子命令就是image   &#x20;

参数详解：        --record=true 更改的时候，将信息增加到历史记录中
{% endtab %}

{% tab title="回滚" %}
```
回滚命令： kubectl rollout SUBCOMMAND [options] 资源类型 资源名称
    子命令：
    history     显示 rollout 历史
    status      显示 rollout 的状态
    undo        撤销上一次的 rollout

```

```bash
资源回滚操作
# kubectl rollout undo deployment superopsmsb-deployment
deployment.apps/superopsmsb-deployment rolled back
# kubectl rollout history deployment superopsmsb-deployment
deployment.apps/superopsmsb-deployment
REVISION  CHANGE-CAUSE
1         <none>
4         kubectl set image nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.16.0 --filename=03_kubernetes_deploy_test.yaml --record=true
5         kubectl set image nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.0 --filename=03_kubernetes_deploy_test.yaml --record=true
```
{% endtab %}
{% endtabs %}

## RS和RC的区别



<table data-view="cards"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td></td><td></td><td></td></tr><tr><td></td><td></td><td></td></tr><tr><td></td><td></td><td></td></tr></tbody></table>

## 资源对象属性

```yaml
apiVersion: apps/v1             # API群组及版本
kind: Deployment                # 资源类型特有标识
metadata:
  name <string>                 # 资源名称，在作用域中要唯一
  namespace <string>            # 名称空间；Deployment隶属名称空间级别
spec:
  minReadySeconds <integer>     # Pod就绪后多少秒内任一容器无crash方可视为“就绪”
  replicas <integer>            # 期望的Pod副本数，默认为1
  selector <object>             # 标签选择器，必须匹配template字段中Pod模板中的标签
  template <object>             # Pod模板对象
​
  revisionHistoryLimit <integer> # 滚动更新历史记录数量，默认为10
  strategy <Object>             # 滚动更新策略
    type <string>               # 滚动更新类型，可用值有Recreate和RollingUpdate；
    rollingUpdate <Object>      # 滚动更新参数，专用于RollingUpdate类型
      maxSurge <string>         # 更新期间可比期望的Pod数量多出的数量或比例；
      maxUnavailable <string>   # 更新期间可比期望的Pod数量缺少的数量或比例，10， 
  progressDeadlineSeconds <integer> # 滚动更新故障超时时长，默认为600秒
  paused <boolean>              # 是否暂停部署过程
```

### 例子

{% tabs %}
{% tab title="手工命令实践" %}
{% code title="创建资源对象" %}
```
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl create deployment nginx-test --image=kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1 --replicas=3
deployment.apps/nginx-test created
```
{% endcode %}

```
查看资源的体系结构
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  get deployments.apps
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-test   3/3     3            3           9s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  get rs
NAME                   DESIRED   CURRENT   READY   AGE
nginx-test-657b8889c   3         3         3       13s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  get pod
NAME                         READY   STATUS    RESTARTS   AGE
nginx-test-657b8889c-k264w   1/1     Running   0          15s
nginx-test-657b8889c-l65gh   1/1     Running   0          15s
nginx-test-657b8889c-txfhr   1/1     Running   0          15s
​

```

```
清理资源对象
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl delete deployments.apps nginx-test
deployment.apps "nginx-test" deleted
```
{% endtab %}

{% tab title="资源清单实践" %}
11

```
创建资源清单文件
[root@kubernetes-master1 /data/kubernetes/controller]# cat 03_kubernetes_deploy_test.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: superopsmsb-deployment
spec:
  minReadySeconds: 0
  replicas: 3
  selector:
    matchLabels:
      app: deploy-test
  template:
    metadata:
      labels:
        app: deploy-test
    spec:
      containers:
      - name: nginx-web
        image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
    
应用资源清单文件
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl apply -f 03_kubernetes_deploy_test.yaml
deployment.apps/superopsmsb-deployment created
​
```

```
查看资源结构
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get deployments.apps
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
superopsmsb-deployment   3/3     3            3           65s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  get rs
NAME                               DESIRED   CURRENT   READY   AGE
superopsmsb-deployment-677cc5798   3         3         3       69s
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  get pod
NAME                                     READY   STATUS    RESTARTS   AGE
superopsmsb-deployment-677cc5798-dzq9b   1/1     Running   0          72s
superopsmsb-deployment-677cc5798-q2zbk   1/1     Running   0          72s
superopsmsb-deployment-677cc5798-tnqrm   1/1     Running   0          72s
```


{% endtab %}

{% tab title="扩缩容" %}
{% code title="资源扩容" %}
```bash
# kubectl get deployments.apps
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
superopsmsb-deployment   3/3     3            3           3m24s
# kubectl  scale deployment superopsmsb-deployment --replicas=10
deployment.apps/superopsmsb-deployment scaled
# kubectl get deployments.apps                                NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
superopsmsb-deployment   10/10   10           10          3m40s
```
{% endcode %}

{% code title="资源缩容" %}
```bash
# kubectl get deployments.apps    NAME  READY   UP-TO-DATE   AVAILABLE   AGE
superopsmsb-deployment   10/10   10           10          5m6s
# kubectl  scale -f 03_kubernetes_deploy_test.yaml --replicas=1
deployment.apps/superopsmsb-deployment scaled
# kubectl get deployments.apps   NAME   READY   UP-TO-DATE   AVAILABLE   AGE
superopsmsb-deployment   1/1     1       1       5m15s
```
{% endcode %}

\

{% endtab %}

{% tab title="更新" %}
更新软件版本

```bash
更新软件
# kubectl set image -f 03_kubernetes_deploy_test.yaml nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.16.0
deployment.apps/superopsmsb-deployment image updated
​
查看效果
# kubectl get deployments.apps -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                                                         SELECTOR
superopsmsb-deployment   1/1     1            1           7m19s   nginx-web    kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.16.0   app=deploy-test
​
查看效果
# kubectl  exec -it superopsmsb-deployment-cc6444576-ngvt4 -- env | grep VERSION
NGINX_VERSION=1.16.0 # 更新完
```

查看历史记录

更新时候没有显示记录，我们可以借助于 已经被废弃的--record=true 的方式，增加更新记录

```bash
查看更新状态
# kubectl rollout status deployment superopsmsb-deployment
deployment "superopsmsb-deployment" successfully rolled out
​
查看更新历史
# kubectl rollout history deployment superopsmsb-deployment
deployment.apps/superopsmsb-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```bash
携带记录式更新
# kubectl set image -f 03_kubernetes_deploy_test.yaml nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.0 --record=true
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/superopsmsb-deployment image updated
# kubectl rollout history deployment superopsmsb-deployment   deployment.apps/superopsmsb-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.0 --filename=03_kubernetes_deploy_test.yaml --record=true
​
携带记录式更新
# kubectl set image -f 03_kubernetes_deploy_test.yaml nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.16.0 --record=true
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/superopsmsb-deployment image updated
# kubectl rollout history deployment superopsmsb-deployment   deployment.apps/superopsmsb-deployment
REVISION  CHANGE-CAUSE
1         <none>
3         kubectl set image nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.0 --filename=03_kubernetes_deploy_test.yaml --record=true
4         kubectl set image nginx-web=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.16.0 --filename=03_kubernetes_deploy_test.yaml --record=true
```
{% endtab %}
{% endtabs %}

\
