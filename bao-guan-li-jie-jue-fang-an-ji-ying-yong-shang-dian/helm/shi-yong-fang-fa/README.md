# 使用方法

## 添加及删除仓库

{% tabs %}
{% tab title="查看仓库" %}
```
[root@master1 ~]# helm repo list
Error: no repositories to show
```
{% endtab %}

{% tab title="添加新的仓库地址" %}
```
微软源
[root@k8s-master01 ~]# helm repo add stable http://mirror.azure.cn/kubernetes/charts/
​
bitnami源
[root@k8s-master01 ~]# helm repo add bitnami https://charts.bitnami.com/bitnami
​
prometheus源
[root@k8s-master01 ~]# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```


{% endtab %}

{% tab title="查看已经添加的仓库" %}
```
[root@k8s-master01 ~]# helm repo list
NAME    URL
stable  http://mirror.azure.cn/kubernetes/charts/
```


{% endtab %}

{% tab title="更新仓库" %}
```
[root@k8s-master01 ~]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

**再查看**

```
[root@master ~]# helm repo list
NAME    URL
stable  http://mirror.azure.cn/kubernetes/charts/
```


{% endtab %}

{% tab title="删除仓库" %}
```
[root@k8s-master01 ~]# helm repo remove stable
"stable" has been removed from your repositories
```

```
[root@k8s-master01 ~]# helm repo list
Error: no repositories to show
```


{% endtab %}
{% endtabs %}



<details>

<summary>查看charts</summary>

使用`helm search repo 关键字`可以查看相关charts

```
[root@k8s-master01 ~]# helm search repo stable
NAME                                    CHART VERSION   APP VERSION             DESCRIPTION
stable/acs-engine-autoscaler            2.2.2           2.1.1                   DEPRECATED Scales worker nodes within agent pools
stable/aerospike                        0.3.5           v4.5.0.5                DEPRECATED A Helm chart for Aerospike in Kubern...
stable/airflow                          7.13.3          1.10.12                 DEPRECATED - please use: https://github.com/air...
stable/ambassador                       5.3.2           0.86.1                  DEPRECATED A Helm chart for Datawire Ambassador
stable/anchore-engine                   1.7.0           0.7.3                   Anchore container analysis and policy evaluatio...
stable/apm-server                       2.1.7           7.0.0                   DEPRECATED The server receives data from the El...
stable/ark                              4.2.2           0.10.2                  DEPRECATED A Helm chart for ark
stable/artifactory                      7.3.2           6.1.0                   DEPRECATED Universal Repository Manager support...
stable/artifactory-ha                   0.4.2           6.2.0                   DEPRECATED Universal Repository Manager support...
stable/atlantis                         3.12.4          v0.14.0                 DEPRECATED A Helm chart for Atlantis https://ww...
stable/auditbeat                        1.1.2           6.7.0                   DEPRECATED A lightweight shipper to audit the a...
stable/aws-cluster-autoscaler           0.3.4                                   DEPRECATED Scales worker nodes within autoscali...
stable/aws-iam-authenticator            0.1.5           1.0                     DEPRECATED A Helm chart for aws-iam-authenticator
stable/bitcoind                         1.0.2           0.17.1                  DEPRECATED Bitcoin is an innovative payment net...
stable/bookstack                        1.2.4           0.27.5                  DEPRECATED BookStack is a simple, self-hosted, ...
......
```

```
[root@k8s-master01 ~]# helm search repo nginx
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
stable/nginx-ingress            1.41.3          v0.34.1         DEPRECATED! An nginx Ingress controller that us...
stable/nginx-ldapauth-proxy     0.1.6           1.13.5          DEPRECATED - nginx proxy with ldapauth
stable/nginx-lego               0.3.1                           Chart for nginx-ingress-controller and kube-lego
stable/gcloud-endpoints         0.1.2           1               DEPRECATED Develop, deploy, protect and monitor...
```

### 查看chart资源

```
[root@k8s-master01 ~]# kubectl get all -l release=mysql-1658996042
NAME                                    READY   STATUS    RESTARTS   AGE
pod/mysql-1658996042-755f5f64f6-j5s67   1/1     Running   0          72m
​
NAME                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/mysql-1658996042   ClusterIP   10.96.2.136   <none>        3306/TCP   72m
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-1658996042   1/1     1            1           72m
​
NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-1658996042-755f5f64f6   1         1         1       72m
```

我们也可以 `helm show chart` 命令来了解 MySQL 这个 chart 包的一些特性：

```
[root@k8s-master01 ~]# helm show chart stable/mysql
apiVersion: v1
appVersion: 5.7.30
deprecated: true
description: DEPRECATED - Fast, reliable, scalable, and easy to use open-source relational
  database system.
home: https://www.mysql.com/
icon: https://www.mysql.com/common/logos/logo-mysql-170x115.png
keywords:
- mysql
- database
- sql
name: mysql
sources:
- https://github.com/kubernetes/charts
- https://github.com/docker-library/mysql
version: 1.6.9
```

如果想要了解更多信息，可以用 `helm show all` 命令：

```
[root@k8s-master01 ~]# helm show all stable/mysql
......
```

</details>

<details>

<summary>删除Release</summary>

如果需要删除这个 release，只需要使用 `helm uninstall`或`helm delete` 命令即可：

```
[root@k8s-master01 ~]# helm uninstall mysql-1605195227
release "mysql-1605195227" uninstalled
```

`uninstall` 命令会从 Kubernetes 中删除 release，也会删除与 release 相关的所有 Kubernetes 资源以及 release 历史记录。

```
[root@k8s-master01 ~]# helm ls
NAME              NAMESPACE     REVISION     UPDATED    STATUS        CHART           APP VERSION
mysql-1605192239     default     1        .........     deployed      mysql-1.6.9       5.7.30
```

在删除的时候使用 `--keep-history` 参数，则会保留 release 的历史记录，该 release 的状态就是 `UNINSTALLED`，

```
[root@k8s-master01 ~]# helm uninstall mysql-1605192239 --keep-history
release "mysql-1605192239" uninstalled
​
[root@k8s-master01 ~]# helm ls -a
NAME                    NAMESPACE       REVISION        UPDATED     STATUS        CHART     APP VERSION
mysql-1605192239        default         1              ........    uninstalled     mysql-1.6.9     5.7.30
状态为uninstalled
```

审查历史时甚至可以取消删除`release`。

`Usage: helm rollback <RELEASE> [REVISION] [flags]`

```
[root@k8s-master01 ~]# helm rollback mysql-1605192239 1
Rollback was a success! Happy Helming!
​
[root@k8s-master01 ~]# helm ls
NAME              NAMESPACE     REVISION     UPDATED    STATUS        CHART           APP VERSION
mysql-1605192239     default     2        .........     deployed      mysql-1.6.9       5.7.30
rollback后，又回到deployed状态
```

</details>



<details>

<summary>定制参数部署应用</summary>

上面我们都是直接使用的 `helm install` 命令安装的 chart 包，这种情况下只会使用 chart 的默认配置选项，但是更多的时候，是各种各样的需求，所以我们希望根据自己的需求来定制 chart 包的配置参数。

我们可以使用 `helm show values` 命令来查看一个 chart 包的所有可配置的参数选项：

```
[root@k8s-master01 ~]# helm show values stable/mysql
......
......
```

上面我们看到的所有参数都是可以用自己的数据来覆盖的，可以在安装的时候通过 YAML 格式的文件来传递这些参数

1，准备参数文件

```bash
# vim mysql-config.yml
mysqlDatabase: helm
persistence:
  enabled: true  # 没有存储卷情况下，改为false
  storageClass: nfs-client
```

2, 使用`-f mysql-config.yml`安装应用并覆盖参数

```bash
# helm install mysql -f mysql-config.yml stable/mysql
```

{% code title="输出内容" %}
```
NAME: mysql
LAST DEPLOYED: Fri Jul 29 14:07:17 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql.default.svc.cluster.local
​
To get your root password run:
​
    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
​
To connect to your database:
​
1. Run an Ubuntu pod that you can use as a client:
​
    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
​
2. Install the mysql client:
​
    $ apt-get update && apt-get install mysql-client -y
​
3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql -p
​
To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306
​
    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql 3306
​
    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```
{% endcode %}

3, 查看覆盖的参数

```
# helm get values mysql
USER-SUPPLIED VALUES:
mysqlDatabase: helm
persistence:
  enabled: true
  storageClass: nfs-client
```

4, 查看部署的相关资源

```
[root@k8s-master01 helmdir]# kubectl get all -l release=mysql
NAME                         READY   STATUS    RESTARTS   AGE
pod/mysql-855976764d-npvgm   1/1     Running   0          40m
​
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/mysql   ClusterIP   10.96.0.84   <none>        3306/TCP   40m
​
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql   1/1     1            1           40m
​
NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-855976764d   1         1         1       40m
```

5, 查看pod的IP

```
[root@k8s-master01 helmdir]# kubectl get pods -o wide -l release=mysql
NAME                     READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
mysql-855976764d-npvgm   1/1     Running   0          41m   100.119.84.71   k8s-worker01   <none>           <none>
​
得到pod的IP为100.119.84.71
```

6, 安装mysql客户端并连接测试

```bash
# yum install mariadb -y
```

```bash
# kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
wL2SD0RCsT

# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP    27h
mysql        ClusterIP   10.96.0.84   <none>        3306/TCP   5m21s

# mysql -h 10.96.0.84 -uroot -pwL2SD0RCsT -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| helm               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

# kubectl get pods -o wide -l release=mysql
NAME                     READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
mysql-855976764d-npvgm   1/1     Running   0          41m   100.119.84.71   k8s-worker01   <none>           <none>

# mysql -h 100.119.84.71 -uroot -pwL2SD0RCsT -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| helm               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

</details>



## 升级和回滚

当新版本的 chart 包发布的时候，或者当你要更改 release 的配置的时候，你可以使用 `helm upgrade` 命令来操作。升级需要一个现有的 release，并根据提供的信息对其进行升级。因为 Kubernetes charts 可能很大而且很复杂，Helm 会尝试以最小的侵入性进行升级，它只会更新自上一版本以来发生的变化：

{% tabs %}
{% tab title="升级前查看版本" %}
```
[root@k8s-master01 helmdir]# mysql -h 10.96.0.84 -uroot -pwL2SD0RCsT -e "select version()"
+-----------+
| version() |
+-----------+
| 5.7.30    |     版本为5.7.30
+-----------+
​
[root@k8s-master01 helmdir]#  kubectl get deployment mysql -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
mysql   1/1     1            1           54m   mysql        mysql:5.7.30   app=mysql,release=mysql
images版本为5.7.30
```
{% endtab %}

{% tab title="修改配置并升级" %}
```
# vim mysql-config.yml
mysqlDatabase: kubemsb
persistence:
  enabled: true
  storageClass: nfs-client
```

升级并且加一个`--set imageTag=5.7.31`参数设置为5.7.31版本

```bash
# helm upgrade mysql -f mysql-config.yml --set imageTag=5.7.31 stable/mysql
```

{% code title="升级过程中的输出" %}
```
WARNING: This chart is deprecated
Release "mysql" has been upgraded. Happy Helming!
NAME: mysql
LAST DEPLOYED: Fri Jul 29 15:04:20 2022
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql.default.svc.cluster.local
​
To get your root password run:
​
    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
​
To connect to your database:
​
1. Run an Ubuntu pod that you can use as a client:
​
    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
​
2. Install the mysql client:
​
    $ apt-get update && apt-get install mysql-client -y
​
3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql -p
​
To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306
​
    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql 3306
​
    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
    
 注意：更新过程中，密码会被更新，但是实际使用中，密码并未更新。
```
{% endcode %}
{% endtab %}

{% tab title=" 升级后确认版本" %}
```bash
# kubectl get deployment mysql -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
mysql   1/1     1            1           58m   mysql        mysql:5.7.31   app=mysql,release=mysql

# kubectl get pods -o wide
NAME                    READY  STATUS   RESTARTS  AGE     IP              NODE           NOMINATED NODE   READINESS GATES
mysql-6f57f64c9d-sc72v  1/1    Running  0         2m20s   100.119.84.72   k8s-worker01   <none>           <none>

# kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS  AGE     IP              NODE           NOMINATED NODE   READINESS GATES
mysql-6f57f64c9d-sc72v   1/1     Running   0         2m20s   100.119.84.72   k8s-worker01   <none>           <none>

# mysql -h 100.119.84.72 -uroot -pwL2SD0RCsT -e "select version()"
+-----------+
| version() |
+-----------+
| 5.7.31    |       版本升级为5.7.31
+-----------+
```
{% endtab %}

{% tab title="回滚 " %}
```bash
# helm history mysql
REVISION  UPDATED                    STATUS      CHART        APP VERSION   DESCRIPTION
1         Fri Jul 29 14:07:17 2022   superseded  mysql-1.6.9  5.7.30        Install complete
2         Fri Jul 29 15:04:20 2022   deployed    mysql-1.6.9  5.7.30        Upgrade complete

# helm rollback mysql 1
Rollback was a success! Happy Helming!
```
{% endtab %}

{% tab title="验证" %}
```bash
# kubectl get deployment mysql -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
mysql   1/1     1            1           65m   mysql        mysql:5.7.30   app=mysql,release=mysql

# helm history mysql
REVISION  UPDATED                    STATUS       CHART         APP VERSION  DESCRIPTION
1         Fri Jul 29 14:07:17 2022   superseded   mysql-1.6.9   5.7.30       Install complete
2         Fri Jul 29 15:04:20 2022   superseded   mysql-1.6.9   5.7.30       Upgrade complete
3         Fri Jul 29 15:12:24 2022   deployed     mysql-1.6.9   5.7.30       Rollback to 1
```
{% endtab %}
{% endtabs %}

\


\
\
