# 创建可配置的Chart

#### 官方的预定义变量

* Release.Name：发布的名称（不是chart）
* Release.Time：chart发布上次更新的时间。这将匹配Last ReleasedRelease对象上的时间。
* Release.Namespace：chart发布到的名称空间。
* Release.Service：进行发布的服务。
* Release.IsUpgrade：如果当前操作是升级或回滚，则设置为true。
* Release.IsInstall：如果当前操作是安装，则设置为true。
* Release.Revision：修订号。它从1开始，每个都递增helm upgrade。
* Chart：内容Chart.yaml。因此，chart版本可以Chart.Version和维护者一样获得 Chart.Maintainers。
* Files：类似于chart的对象，包含chart中的所有非特殊文件。这不会授予您访问模板的权限，但可以访问存在的其他文件（除非使用它们除外.helmignore）。可以使用\{{index .Files "file.name"\}}或使用\{{.Files.Get name\}}或 \{{.Files.GetStringname\}}函数访问文件。您也可以访问该文件的内容，\[]byte使用\{{.Files.GetBytes\}}
* Capabilities：类似于地图的对象，包含有关Kubernetes（\{{.Capabilities.KubeVersion\}}，Tiller（\{{.Capabilities.TillerVersion\}}和支持的Kubernetes API）版本（\{{.Capabilities.APIVersions.Has "batch/v1"）的版本的信息

## 新增values.yaml文件

```
[root@k8s-master01 nginx]# pwd
/helm/nginx
[root@k8s-master01 nginx]# vim values.yaml
image:
  repository: nginx
  tag: '1.15-alpine'
replicas: 2
```

## 配置deploy引用values的值

```
[root@k8s-master01 nginx]# vim templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helm-nginx
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: helm-nginx
  template:
    metadata:
      labels:
        app: helm-nginx
    spec:
      containers:
      - name: helm-nginx
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: IfNotPresent
```

## 测试

{% tabs %}
{% tab title="直接应用测试" %}
deployment.yaml将直接使用values.yaml中的配置

```bash
# helm install helm-nginx-new /helm/nginx
NAME: helm-nginx-new
LAST DEPLOYED: Sat Jul 30 09:44:21 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

# kubectl get pods
NAME                                     READY   STATUS    RESTARTS      AGE
helm-nginx-65f57fb758-pcmkg              1/1     Running   0             38s
helm-nginx-65f57fb758-rmmv5              1/1     Running   0             38s
```
{% endtab %}

{% tab title="通过命令行设置变量后干运行测试" %}
通过在命令行设置变量为deployment.yaml赋值，使用--set选项，使用`--dry-run`选项来打印出生成的清单文件内容，而不执行部署

```bash
# helm install helm-nginx --set replicas=3 /helm/nginx/ --dry-run
NAME: helm-nginx
LAST DEPLOYED: Fri Nov 13 20:57:45 2020
NAMESPACE: default
STATUS: pending-install    # 状态表示是测试，不是真的部署了
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: helm-nginx/templates/service.yaml
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
---
# Source: helm-nginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helm-nginx
spec:
  replicas: 3            # 副本数量3传参成功
  selector:
    matchLabels:
      app: helm-nginx
  template:
    metadata:
      labels:
        app: helm-nginx
    spec:
      containers:
      - name: helm-nginx
        image: nginx:1.15-alpine       #  镜像名:TAG 传参成功
        imagePullPolicy: IfNotPresent
```


{% endtab %}

{% tab title="Untitled" %}
```bash
# helm install helm-nginx --set replicas=3 /helm/nginx
NAME: helm-nginx
LAST DEPLOYED: Sat Jul 30 09:54:00 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

# helm ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
helm-nginx      default         1               2022-07-30 09:54:00.744748457 +0800 CST deployed        helm-nginx-1.0.0

# kubectl get pods,svc
NAME                                         READY   STATUS    RESTARTS      AGE
pod/helm-nginx-65f57fb758-j768m              1/1     Running   0             59s
pod/helm-nginx-65f57fb758-pscjh              1/1     Running   0             58s
pod/helm-nginx-65f57fb758-s6qqj              1/1     Running   0             58s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/helm-nginx   ClusterIP   10.96.1.197   <none>        80/TCP    59s
```


{% endtab %}
{% endtabs %}

## 将Chart包进行打包

> 将chart打包成一个压缩文件，便于存储与分享。

```
[root@k8s-master01 nginx]# helm package .
Successfully packaged chart and saved it to: /helm/nginx/helm-nginx-1.0.0.tgz
```

```
[root@k8s-master01 nginx]# ls
Chart.yaml  helm-nginx-1.0.0.tgz  templates  values.yaml
打包出mychart-0.1.0.tgz文件
```

## 使用Chart安装

```
[root@master nginx]# helm install helm-nginx2 /helm/nginx/helm-nginx-1.0.0.tgz
```

\


