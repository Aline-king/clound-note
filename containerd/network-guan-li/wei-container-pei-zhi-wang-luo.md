# 为container配置网络

## 创建一个容器

```
# ctr images ls
REF TYPE DIGEST SIZE PLATFORMS LABELS
​
# ctr images pull docker.io/library/busybox:latest
​
# ctr run -d docker.io/library/busybox:latest busybox
​
# ctr container ls
CONTAINER    IMAGE                               RUNTIME
busybox      docker.io/library/busybox:latest    io.containerd.runc.v2
​
# ctr tasks ls
TASK       PID     STATUS
busybox    8377    RUNNING
```

## 进入容器查看其网络情况

```
# ctr tasks exec --exec-id $RANDOM -t busybox sh
​
/ # ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```

## 获取容器进程ID及其网络命名空间

```
在宿主机中完成指定容器进程ID获取
# pid=$(ctr tasks ls | grep busybox | awk '{print $2}')
# echo $pid
8377
​
```

```
在宿主机中完成指定容器网络命名空间路径获取
# netnspath=/proc/$pid/ns/net
# echo $netnspath
/proc/8377/ns/net
```

## 为指定容器添加网络配置

```
确认执行脚本文件时所在的目录
[root@localhost scripts]# pwd
/home/cni/scripts
```

```
执行脚本文件为容器添加网络配置
[root@localhost scripts]# CNI_PATH=/home/cni-plugins ./exec-plugins.sh add $pid $netnspath
```

```
进入容器确认是否添加网卡信息
# ctr tasks exec --exec-id $RANDOM -t busybox sh
/ # ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether a2:35:b7:e0:60:0a brd ff:ff:ff:ff:ff:ff
    inet 10.66.0.3/16 brd 10.66.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a035:b7ff:fee0:600a/64 scope link
       valid_lft forever preferred_lft forever
       
在容器中ping容器宿主机IP地址
/ # ping -c 2 192.168.10.164
PING 192.168.10.164 (192.168.10.164): 56 data bytes
64 bytes from 192.168.10.164: seq=0 ttl=64 time=0.132 ms
64 bytes from 192.168.10.164: seq=1 ttl=64 time=0.044 ms
​
--- 192.168.10.164 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.044/0.088/0.132 ms
​
在容器中ping宿主机所在网络的网关IP地址
/ # ping -c 2 192.168.10.2
PING 192.168.10.2 (192.168.10.2): 56 data bytes
64 bytes from 192.168.10.2: seq=0 ttl=127 time=0.338 ms
64 bytes from 192.168.10.2: seq=1 ttl=127 time=0.280 ms
​
--- 192.168.10.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.280/0.309/0.338 ms
​
在容器中ping宿主机所在网络中的其它主机IP地址
/ # ping -c 2 192.168.10.165
PING 192.168.10.165 (192.168.10.165): 56 data bytes
64 bytes from 192.168.10.165: seq=0 ttl=63 time=0.422 ms
64 bytes from 192.168.10.165: seq=1 ttl=63 time=0.908 ms
​
--- 192.168.10.165 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.422/0.665/0.908 ms
​
```

```
在容器中开启httpd服务
/ # echo "containerd net web test" > /tmp/index.html
/ # httpd -h /tmp
​
/ # wget -O - -q 127.0.0.1
containerd net web test
/ # exit
​
```

```
在宿主机访问容器提供的httpd服务
[root@localhost scripts]# curl http://10.66.0.3
containerd net web test
```

\
