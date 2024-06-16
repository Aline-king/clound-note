# HPA

HorizontalPodAutoscaler (HPA) 和 VerticalPodAutoscaler (VPA) 使用 metrics API 中的数据调整工作负载副本和资源，以满足客户需求。

水平(Horizontal)扩缩意味着对增加的负载的响应是部署更多的 Pods。&#x20;

垂直(Vertical)扩缩扩缩意味着将更多资源分配给已经为工作负载运行的 Pod。

## 资源属性

```yaml
apiVersion: autoscaling/v2 			# API群组及版本
kind: HorizontalPodAutoscaler			# 资源类型特有标识
metadata:
  name <string>  					# 资源名称，在作用域中要唯一
  namespace <string>  				# 名称空间；资源隶属名称空间级别
spec:
  scaleTargetRef  <Object>  			# 带扩展的资源对象，必选字段
  maxReplicas <integer>				# 必选字段，扩容最大值
  minReplicas <integer>				# 可选字段，缩容最小值
  metrics      <[]Object> 			# 以什么指标监控
  - type: Resource					# 监控的类型
    resource:  <Object>					# 资源对象的属性
      name: <string>				# 必选字段，监视资源名称，可以是cpu和mem
      target:	<Object>			# 必选字段，资源目标值
        type: <string>				# 必选字段，值的类型，
        averageValue: <string>			# 普通目标数据值
	averageUtilization  <integer>		# 百分比目标数据值
  behavior:
    scaleDown:  <Object>			# 缩容行为
      stabilizationWindowSeconds: 90		# 扩缩容的基本时间范围，0-3600，默认300
```

## 实例

{% tabs %}
{% tab title="First Tab" %}
```yaml
定制基础的应用对象
[root@kubernetes-master1 /data/kubernetes/controller]# cat 07_kubernetes_pod_hpa_1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: superopsmsb-django-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: django-web
  template:
    metadata:
      labels:
        app: django-web
    spec:
      containers:
      - name: django-web
        image: kubernetes-register.superopsmsb.com/superopsmsb/django_web:v0.1
---
apiVersion: v1
kind: Service
metadata:
  name: superopsmsb-django-service
spec:
  selector:
    app: django-web
  ports:
  - name: http
    port: 8000
```
{% endtab %}

{% tab title="手工hpa" %}
手工hpa实践

```
注意：
    由于这里查询的是核心指标，所以它是从metrics-server中获取到的
```

```bash
在k8s中有一个自动的扩缩容命令 autoscale,可以创建hpa对象
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl autoscale deployment superopsmsb-django-web --min=2 --max=5 --cpu-percent=40
horizontalpodautoscaler.autoscaling/superopsmsb-django-web autoscaled
        --min=2             # 最少的pod数量
        --max=5             # 最多的pod数量
        --cpu-percent=40    # 调整的标准，以cpu为例
​
查看效果
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get hpa
NAME                     REFERENCE                           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
superopsmsb-django-web   Deployment/superopsmsb-django-web   23%/40%   2         5         2          35s
```

```
创建测试pod
[root@kubernetes-master1 ~]# kubectl run flask-web --image=kubernetes-register.superopsmsb.com/superopsmsb/flask_web:v0.1
pod/flask-web created
[root@kubernetes-master1 ~]# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
flask-web                                 1/1     Running   0          3s
​
进入容器环境
[root@kubernetes-master1 ~]# kubectl  exec -it flask-web -- /bin/bash
root@flask-web:/# while true; do curl -s http://superopsmsb-django-service:8000; sleep 0.001; done
...
​
查看hpa的效果
[root@kubernetes-master1 ~]# kubectl get hpa -w
NAME                     REFERENCE                           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
superopsmsb-django-web   Deployment/superopsmsb-django-web   32%/40%   2         5         2          26m
superopsmsb-django-web   Deployment/superopsmsb-django-web   81%/40%   2         5         4          27m
superopsmsb-django-web   Deployment/superopsmsb-django-web   62%/40%   2         5         5          27m
结果显示：
    hpa设定的cpu阈值超过 40%的时候，资源会自动扩容
    这里会有一个缓冲时间，默认是1分钟
```

```
进入容器环境停止测试过程
[root@kubernetes-master1 ~]# kubectl  exec -it flask-web -- /bin/bash
root@flask-web:/# while true; do curl -s http://superopsmsb-django-service:8000; sleep 0.001; done
^C
​
再次查看hpa的效果
[root@kubernetes-master1 ~]# kubectl get hpa -w
NAME                     REFERENCE                           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
...
superopsmsb-django-web   Deployment/superopsmsb-django-web   81%/40%   2         5         4          27m
...
superopsmsb-django-web   Deployment/superopsmsb-django-web   23%/40%   2         5         5          32m
superopsmsb-django-web   Deployment/superopsmsb-django-web   22%/40%   2         5         3          33m
...
superopsmsb-django-web   Deployment/superopsmsb-django-web   22%/40%   2         5         2          40m
结果显示：
    hpa设定的cpu阈值低于 40%的时候，资源会自动缩容，
    但是为了防止过载反复，这里的缩容会有一个缓冲时间，默认为5分钟左右
```

```
清理资源对象
[root@kubernetes-master1 ~]# kubectl delete hpa superopsmsb-django-web
horizontalpodautoscaler.autoscaling "superopsmsb-django-web" deleted
```

\

{% endtab %}

{% tab title="资源清单hpa" %}
资源清单hpa实践

```
[root@kubernetes-master1 /data/kubernetes/controller]# cat 08_kubernetes_pod_hpa_2.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: superopsmsb-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: django-web
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 90
      
创建资源对象
[root@kubernetes-master1 /data/backup/controller]# kubectl  apply -f 08_kubernetes_pod_hpa_2.yaml
horizontalpodautoscaler.autoscaling/superopsmsb-hpa created
```

{% hint style="warning" %}
注意：&#x20;

这里获取的数据内容是需要等待几秒才会出现&#x20;

]# echo $((49086464 /1024/1024)) 46  M
{% endhint %}

```
查看效果
[root@kubernetes-master1 /data/backup/controller]# kubectl get hpa
NAME              REFERENCE                           TARGETS                   MINPODS   MAXPODS   REPLICAS   AGE
superopsmsb-hpa   Deployment/superopsmsb-django-web   49086464/500Mi, 23%/50%   2         5         2          75s
    
在测试终端执行测试
while true; do curl -s http://superopsmsb-django-service:8000; sleep 0.001; done
```

```
在控制端确认hpa效果
[root@kubernetes-master1 ~]# kubectl  get hpa -w
NAME            REFERENCE                         TARGETS                 MINPODS MAXPODS RE PLICAS  AGE
superopsmsb-hpa Deployment/superopsmsb-django-web 49086464/500Mi, 23%/50% 2       5       2          6m33s
superopsmsb-hpa Deployment/superopsmsb-django-web 49362944/500Mi, 35%/50% 2       5       2          6m46s
superopsmsb-hpa Deployment/superopsmsb-django-web 49412096/500Mi, 83%/50% 2       5       2          7m1s
superopsmsb-hpa Deployment/superopsmsb-django-web 49551360/500Mi, 79%/50% 2       5       5          7m16s
superopsmsb-hpa   Deployment/superopsmsb-django-web 49135616/500Mi, 24%/50% 2     5       5          11m
superopsmsb-hpa Deployment/superopsmsb-django-web 49623040/500Mi, 23%/50% 2       5       2          14m
```
{% endtab %}
{% endtabs %}

