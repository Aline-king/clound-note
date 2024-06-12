# 联盟网络

#### 4.2.4 联盟网络

```
创建c1容器，使用默认网络模型
# docker run -it --name c1 --rm busybox:latest
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1916 (1.8 KiB)  TX bytes:0 (0.0 B)
​
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
查看c1容器状态
# docker ps
CONTAINER ID   IMAGE            COMMAND   CREATED          STATUS          PORTS     NAMES
0905bc8ebfb6   busybox:latest   "sh"      13 seconds ago   Up 11 seconds             c1
```

```
创建c2容器，与c1容器共享网络命名空间
# docker run -it --name c2 --network container:c1 --rm busybox:latest
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:22 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:2574 (2.5 KiB)  TX bytes:0 (0.0 B)
​
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
在c2容器中创建文件并开启httpd服务
/ # echo "hello world" >> /tmp/index.html
/ # ls /tmp
index.html
/ # httpd -h /tmp
​
验证80端口是否打开
/ # netstat -npl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 :::80                   :::*                    LISTEN      10/httpd
```

```
在c1容器中进行访问验证
# docker exec c1 wget -O - -q 127.0.0.1
hello world
```

```
查看c1容器/tmp目录，发现没有在c2容器中创建的文件，说明c1与c2仅共享了网络命名空间，没有共享文件系统
# docker exec c1 ls /tmp
```

\
