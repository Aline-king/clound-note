# 直接运行一个动态容器

> 说明：
>
> * \-d 代表dameon，后台运行
> * \--net-host 代表容器的IP就是宿主机的IP(相当于docker里的host类型网络)

```
# ctr run -d --net-host docker.io/library/nginx:alpine nginx2
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
```

```
查看已运行容器
# ctr container ls
CONTAINER    IMAGE                             RUNTIME
nginx2       docker.io/library/nginx:alpine    io.containerd.runc.v2
```

```
查看已运行容器中运行的进程,既tasks
# ctr tasks ls
TASK      PID     STATUS
nginx2    4061    RUNNING
```

```
进入容器
# ctr task exec --exec-id 1 -t nginx2 /bin/sh
```

```
/ # ifconfig 
ens33     Link encap:Ethernet  HWaddr 00:0C:29:B1:B6:1D
          inet addr:192.168.10.164  Bcast:192.168.10.255  Mask:255.255.255.0
          inet6 addr: fe80::2b33:40ed:9311:8812/64 Scope:Link
          inet6 addr: fe80::adf4:a8bc:a1c:a9f7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:55360 errors:0 dropped:0 overruns:0 frame:0
          TX packets:30526 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:53511295 (51.0 MiB)  TX bytes:2735050 (2.6 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:68 errors:0 dropped:0 overruns:0 frame:0
          TX packets:68 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5916 (5.7 KiB)  TX bytes:5916 (5.7 KiB)

virbr0    Link encap:Ethernet  HWaddr 52:54:00:E9:51:82
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
为容器中运行的网站添加网站文件
/ # echo "nginx2" > /usr/share/nginx/html/index.html
/ # exit
```

```
在宿主机上访问网站
[root@localhost ~]# curl 192.168.10.164
nginx2
```
