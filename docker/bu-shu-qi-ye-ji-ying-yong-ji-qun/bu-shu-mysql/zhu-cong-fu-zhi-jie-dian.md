# 主从复制节点

#### MySQL主节点部署

```
# docker run -p 3306:3306 \
 --name mysql-master \
 -v /opt/mysql-master/log:/var/log/mysql \
 -v /opt/mysql-master/data:/var/lib/mysql \
 -v /opt/mysql-master/conf:/etc/mysql \
 -e MYSQL_ROOT_PASSWORD=root \
 -d mysql:5.7
```

```
# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                                                  NAMES
2dbbed8e35c7   mysql:5.7   "docker-entrypoint.s…"   58 seconds ago   Up 57 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql-master
```

#### MySQL主节点配置

```
# vim /opt/mysql-master/conf/my.cnf
# cat /opt/mysql-master/conf/my.cnf
[client]
default-character-set=utf8
​
[mysql]
default-character-set=utf8
​
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
​
server_id=1
log-bin=mysql-bin
read-only=0
binlog-do-db=kubemsb_test
​
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
```

#### MySQL从节点部署

```
# docker run -p 3307:3306 \
 --name mysql-slave \
 -v /opt/mysql-slave/log:/var/log/mysql \
 -v /opt/mysql-slave/data:/var/lib/mysql \
 -v /opt/mysql-slave/conf:/etc/mysql \
 -e MYSQL_ROOT_PASSWORD=root \
 -d 
 --link mysql-master:mysql-master
 mysql:5.7
```

```
# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
caf7bf3fc68f   mysql:5.7   "docker-entrypoint.s…"   8 seconds ago   Up 6 seconds   33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysql-slave
```

#### 4.2.4 MySQL从节点配置

```
# vim /opt/mysql-slave/conf/my.cnf
# cat /opt/mysql-slave/conf/my.cnf
[client]
default-character-set=utf8
​
[mysql]
default-character-set=utf8
​
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
​
server_id=2
log-bin=mysql-bin
read-only=1
binlog-do-db=kubemsb_test
​
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
```

#### master节点配置

{% tabs %}
{% tab title="First Tab" %}
```
# mysql -h 192.168.255.157 -uroot -proot -P 3306
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.37 MySQL Community Server (GPL)
​
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
​
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
MySQL [(none)]>
```

```
授权
MySQL [(none)]> grant replication slave on *.* to 'backup'@'%' identified by '123456';
```
{% endtab %}

{% tab title="Second Tab" %}
```
重启容器，使用配置生效
# docker restart mysql-master
```

```
查看状态
MySQL [(none)]> show master status\G
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 154
     Binlog_Do_DB: kubemsb_test
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```
{% endtab %}
{% endtabs %}

#### 4.2.6 slave节点配置

{% tabs %}
{% tab title="First Tab" %}
```
# docker restart mysql-slave
```

```
# mysql -h 192.168.255.157 -uroot -proot -P 3307
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.37 MySQL Community Server (GPL)
​
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
​
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
MySQL [(none)]>
```
{% endtab %}

{% tab title="Second Tab" %}
```
MySQL [(none)]> change master to master_host='mysql-master', master_user='backup', master_password='123456', master_log_file='mysql-bin.000001', master_log_pos=154, master_port=3306;
```

```
MySQL [(none)]> start slave;
```
{% endtab %}

{% tab title="Untitled" %}
```
MySQL [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: mysql-master
                  Master_User: backup
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 154
               Relay_Log_File: e0872f94c377-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB: mysql,sys,information_schema,performance_schema
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 534
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 0130b415-8b21-11ec-8982-0242ac110002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```


{% endtab %}
{% endtabs %}

#### 4.2.7 验证MySQL集群可用性

{% tabs %}
{% tab title="First Tab" %}
```
在MySQL Master节点添加kubemsb_test数据库
# mysql -h 192.168.255.157 -uroot -proot -P3306
​
MySQL [(none)]> create database kubemsb_test;
Query OK, 1 row affected (0.00 sec)
​
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kubemsb_test       |     |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)
```


{% endtab %}

{% tab title="Second Tab" %}
```
在MySQL Slave节点查看同步情况
# mysql -h 192.168.255.157 -uroot -proot -P3307
​
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kubemsb_test       |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```


{% endtab %}
{% endtabs %}

\
