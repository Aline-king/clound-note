# 部署mysql

环境说明：k8s集群中存在storageclass:nfs-client

我们现在安装一个 `mysql` 应用：

{% tabs %}
{% tab title="搜索" %}
```bash
# helm search repo mysql
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/mysql                            1.6.9           5.7.30          DEPRECATED - Fast, reliable, scalable, and easy...
```
{% endtab %}

{% tab title="安装" %}
```bash
# helm install stable/mysql --generate-name  --set persistence.storageClass=nfs-client --set mysqlRootPassword=test123
```

```bash
部署过程输出的信息
NAME: mysql-1658996042
LAST DEPLOYED: Thu Jul 28 16:14:03 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-1658996042.default.svc.cluster.local
​
To get your root password run:
​
    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql-1658996042 -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
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
    $ mysql -h mysql-1658996042 -p
​
To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306
​
    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql-1658996042 3306
​
    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```


{% endtab %}

{% tab title="查看" %}
```bash
# helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mysql-1658996042        default         1               2022-07-28 16:14:03.530489788 +0800 CST deployed        mysql-1.6.9     5.7.30

# kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
mysql-1658996042-755f5f64f6-j5s67        1/1     Running   0          82s

# kubectl get pvc
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-1658996042   Bound    pvc-7fcb894e-5b8c-4f3e-945d-21b60b9309e5   8Gi        RWO            nfs-client     93s

# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
pvc-7fcb894e-5b8c-4f3e-945d-21b60b9309e5   8Gi        RWO            Delete           Bound    default/mysql-1658996042   nfs-client              97s
```
{% endtab %}
{% endtabs %}

## 再次安装

**一个 chart 包是可以多次安装到同一个集群中的，每次安装都会产生一个release, 每个release都可以独立管理和升级。**

{% tabs %}
{% tab title="First Tab" %}
```bash
# helm install stable/mysql --generate-name  --set persistence.storageClass=nfs-client --set mysqlRootPassword=root
```

```bash
# helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mysql-1658996042        default         1               2022-07-28 16:14:03.530489788 +0800 CST deployed        mysql-1.6.9     5.7.30
mysql-1658996297        default         1               2022-07-28 16:18:19.282074215 +0800 CST deployed        mysql-1.6.9     5.7.30

# kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
mysql-1658996042-755f5f64f6-j5s67        1/1     Running   0          45m
mysql-1658996297-75f6f86d84-5qd8r        1/1     Running   0          41m
nfs-client-provisioner-9d46587b5-7n2vf   1/1     Running   0          123m
```


{% endtab %}

{% tab title="Second Tab" %}
```
[root@k8s-master01 ~]# kubectl exec -it mysql-1658996042-755f5f64f6-j5s67 -- bash
```

```
root@mysql-1658996042-755f5f64f6-j5s67:/# mysql -uroot -ptest123
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 547
Server version: 5.7.30 MySQL Community Server (GPL)
​
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
​
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
​
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```
{% endtab %}
{% endtabs %}

\
