# 部署mysql

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="单节点" %}
```
# docker run -p 3306:3306 \
 --name mysql \
 -v /opt/mysql/log:/var/log/mysql \
 -v /opt/mysql/data:/var/lib/mysql \
 -v /opt/mysql/conf:/etc/mysql \
 -e MYSQL_ROOT_PASSWORD=root \
 -d \
 mysql:5.7
```

```
# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                                                  NAMES
6d16ca21cf31   mysql:5.7   "docker-entrypoint.s…"   32 seconds ago   Up 30 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql
```

```
通过容器中客户端访问
# docker exec -it mysql mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.37 MySQL Community Server (GPL)
​
Copyright (c) 2000, 2022, Oracle and/or its affiliates.
​
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
​
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
mysql>
```

```
在docker host上访问
# yum -y install mariadb
​
# mysql -h 192.168.255.157 -uroot -proot -P 3306
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.37 MySQL Community Server (GPL)
​
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
​
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

\

{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}
