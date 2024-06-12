# Bridge

```
查看创建网络模型的帮助方法
# docker network create --help
```

```
创建一个名称为mybr0的网络
# docker network create -d bridge --subnet "192.168.100.0/24" --gateway "192.168.100.1" -o com.docker.network.bridge.name=docker1 mybr0
```

```
查看已创建网络
# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
......
a6a1ad36c3c0   mybr0     bridge    local
......
```

```
在docker host主机上可以看到多了一个网桥docker1
# ifconfig
docker1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.100.1  netmask 255.255.255.0  broadcast 192.168.100.255
        ether 02:42:14:aa:f5:04  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 20  bytes 1598 (1.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```
启动一个容器并连接到已创建mybr0网络
# docker run -it --network mybr0 --rm busybox
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:65:02
          inet addr:192.168.100.2  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:2185 (2.1 KiB)  TX bytes:0 (0.0 B)
​
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
​
/ # exit
```

\
