# Redis

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 运行Redis容器

```
# mkdir -p /opt/redis/conf
```

```
# touch /opt/redis/conf/redis.conf
```

```
# docker run -p 6379:6379 --name redis -v /opt/redis/data:/data \
-v /opt/redis/conf:/etc/redis \
-d redis redis-server /etc/redis/redis.conf
```

```
# docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                                                                  NAMES
9bd2b39cd92a   redis                  "docker-entrypoint.s…"   44 seconds ago   Up 42 seconds   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp                                              redis
```

### 7.3 验证

```
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

```
# yum -y install redis
```

```
# redis-cli -h 192.168.255.157 -p 6379
​
192.168.255.157:6379> set test1 a
OK
192.168.255.157:6379> get test1
"a"
```

### 7.4 Redis集群

安装redis-cluster；3主3从方式，从为了同步备份，主进行slot数据分片

{% tabs %}
{% tab title="First Tab" %}
```
编辑运行多个redis容器脚本文件
# vim redis-cluster.sh
# cat redis-cluster.sh
for port in $(seq 8001 8006); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port ${port}
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 192.168.255.157
cluster-announce-port ${port}
cluster-announce-bus-port 1${port}
appendonly yes
EOF
docker run -p ${port}:${port} -p 1${port}:1${port} --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d redis:5.0.7 redis-server /etc/redis/redis.conf; \
done
```

```
执行脚本
# sh redis-cluster.sh
```

```
查看已运行容器
# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS              PORTS                                                                                                NAMES
8d53864a98ce   redis:5.0.7   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:8006->8006/tcp, :::8006->8006/tcp, 6379/tcp, 0.0.0.0:18006->18006/tcp, :::18006->18006/tcp   redis-8006
e2b5da0f0605   redis:5.0.7   "docker-entrypoint.s…"   2 minutes ago        Up About a minute   0.0.0.0:8005->8005/tcp, :::8005->8005/tcp, 6379/tcp, 0.0.0.0:18005->18005/tcp, :::18005->18005/tcp   redis-8005
70e8e8f15aea   redis:5.0.7   "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes        0.0.0.0:8004->8004/tcp, :::8004->8004/tcp, 6379/tcp, 0.0.0.0:18004->18004/tcp, :::18004->18004/tcp   redis-8004
dff8e4bf02b4   redis:5.0.7   "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes        0.0.0.0:8003->8003/tcp, :::8003->8003/tcp, 6379/tcp, 0.0.0.0:18003->18003/tcp, :::18003->18003/tcp   redis-8003
c34dc4c423ef   redis:5.0.7   "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes        0.0.0.0:8002->8002/tcp, :::8002->8002/tcp, 6379/tcp, 0.0.0.0:18002->18002/tcp, :::18002->18002/tcp   redis-8002
b8cb5feffb43   redis:5.0.7   "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes        0.0.0.0:8001->8001/tcp, :::8001->8001/tcp, 6379/tcp, 0.0.0.0:18001->18001/tcp, :::18001->18001/tcp   redis-8001
```


{% endtab %}

{% tab title="Second Tab" %}
```
登录redis容器
# docker exec -it redis-8001 bash
root@b8cb5feffb43:/data#
```

```
创建redis-cluster
root@b8cb5feffb43:/data# redis-cli --cluster create 192.168.255.157:8001 192.168.255.157:8002 192.168.255.157:8003 192.168.255.157:8004 192.168.255.157:8005 192.168.255.157:8006 --cluster-replicas 1
```

```
输出：
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.255.157:8005 to 192.168.255.157:8001
Adding replica 192.168.255.157:8006 to 192.168.255.157:8002
Adding replica 192.168.255.157:8004 to 192.168.255.157:8003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: abd07f1a2679fe77558bad3ff4b7ab70ec41efa5 192.168.255.157:8001
   slots:[0-5460] (5461 slots) master
M: 40e69202bb3eab13a8157c33da6240bb31f2fd6f 192.168.255.157:8002
   slots:[5461-10922] (5462 slots) master
M: 9a927abf3c2982ba9ffdb29176fc8ffa77a2cf03 192.168.255.157:8003
   slots:[10923-16383] (5461 slots) master
S: 81d0a4056328830a555fcd75cf523d4c9d52205c 192.168.255.157:8004
   replicates 9a927abf3c2982ba9ffdb29176fc8ffa77a2cf03
S: 8121a28519e5b52e4817913aa3969d9431bb68af 192.168.255.157:8005
   replicates abd07f1a2679fe77558bad3ff4b7ab70ec41efa5
S: 3a8dd5343c0b8f5580bc44f6b3bb5b4371d4dde5 192.168.255.157:8006
   replicates 40e69202bb3eab13a8157c33da6240bb31f2fd6f
Can I set the above configuration? (type 'yes' to accept): yes 输入yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.....
>>> Performing Cluster Check (using node 192.168.255.157:8001)
M: abd07f1a2679fe77558bad3ff4b7ab70ec41efa5 192.168.255.157:8001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 81d0a4056328830a555fcd75cf523d4c9d52205c 192.168.255.157:8004
   slots: (0 slots) slave
   replicates 9a927abf3c2982ba9ffdb29176fc8ffa77a2cf03
M: 40e69202bb3eab13a8157c33da6240bb31f2fd6f 192.168.255.157:8002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 8121a28519e5b52e4817913aa3969d9431bb68af 192.168.255.157:8005
   slots: (0 slots) slave
   replicates abd07f1a2679fe77558bad3ff4b7ab70ec41efa5
M: 9a927abf3c2982ba9ffdb29176fc8ffa77a2cf03 192.168.255.157:8003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 3a8dd5343c0b8f5580bc44f6b3bb5b4371d4dde5 192.168.255.157:8006
   slots: (0 slots) slave
   replicates 40e69202bb3eab13a8157c33da6240bb31f2fd6f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```


{% endtab %}
{% endtabs %}

\
