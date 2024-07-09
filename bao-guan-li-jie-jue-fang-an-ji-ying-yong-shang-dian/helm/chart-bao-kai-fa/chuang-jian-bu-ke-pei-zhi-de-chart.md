# 创建不可配置的chart



{% tabs %}
{% tab title="创建目录与chart.yaml" %}
```
# mkdir -p /helm/nginx/templates
# cd  /helm/nginx

# vim Chart.yaml
name: helm-nginx
version: 1.0.0
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
```
{% endtab %}

{% tab title="创建deployment.yaml" %}
```yaml
# vim templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helm-nginx
spec:
  replicas: 1                                   
  selector:
    matchLabels:
      app: helm-nginx
  template:
    metadata:
      labels:
        app: helm-nginx
    spec:
      containers:
      - name: c1
        image: nginx:1.15-alpine
        imagePullPolicy: IfNotPresent
```


{% endtab %}

{% tab title="创建service.yaml" %}
```yaml
# vim templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: helm-nginx
spec:
  selector:
    app: helm-nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```


{% endtab %}

{% tab title="使用chart安装应用" %}
```
[root@k8s-master01 nginx]# helm install /helm/nginx --generate-name
NAME: nginx-1659144826
LAST DEPLOYED: Sat Jul 30 09:33:46 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```


{% endtab %}

{% tab title="查看与验证" %}
```bash
# helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
nginx-1659144826        default         1               2022-07-30 09:33:46.881083524 +0800 CST deployed        helm-nginx-1.0.0

# kubectl get pods,service
NAME                                         READY   STATUS    RESTARTS      AGE
pod/helm-nginx-65f57fb758-nrpvf              1/1     Running   0             51s
pod/nfs-client-provisioner-9d46587b5-7n2vf   1/1     Running   4 (31m ago)   42h
​
NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/helm-nginx   ClusterIP   10.96.2.120   <none>        80/TCP    51s
```

```
[root@k8s-master01 nginx]# curl http://10.96.2.120
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


{% endtab %}

{% tab title="删除" %}
```
[root@k8s-master01 ~]# helm uninstall nginx-1659144826
release "nginx-1659144826" uninstalled
```
{% endtab %}
{% endtabs %}

\
