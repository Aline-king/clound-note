# 配置虚拟网卡IP地址

```
再次创建虚拟网卡，添加到msb网络命名空间，并设置IP地址
# ip link add veth0 type veth peer name veth1
# ip link set veth1 netns msb
# ip netns exec msb ip addr add 192.168.50.2/24 dev veth1
```

```
在Linux系统命令行查看网络状态
# ip netns exec msb ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
12: veth1@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether fe:20:ac:a8:13:4c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.50.2/24 scope global veth1
       valid_lft forever preferred_lft forever
```

```
启动虚拟网卡,veth1与lo全部要启动
# ip netns exec msb ip link set veth1 up

# ip netns exec msb ip link set lo up
```

```
为物理机veth0添加IP地址

# ip a s
......
15: veth0@if14: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group defau
lt qlen 1000
    link/ether 2e:b4:40:c8:73:dc brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

```
# ip addr add 192.168.50.3/24 dev veth0

# ip a s veth0
15: veth0@if14: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 2e:b4:40:c8:73:dc brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.50.3/24 scope global veth0
       valid_lft forever preferred_lft forever
```

```
# ip link set veth0 up
```

```
在宿主机上ping msb中的veth1
# ping 192.168.50.2
PING 192.168.50.2 (192.168.50.2) 56(84) bytes of data.
64 bytes from 192.168.50.2: icmp_seq=1 ttl=64 time=0.102 ms
64 bytes from 192.168.50.2: icmp_seq=2 ttl=64 time=0.068 ms
64 bytes from 192.168.50.2: icmp_seq=3 ttl=64 time=0.068 ms
```

```
在msb中的veth1 ping 宿主机上veth0
# ip netns exec msb ping 192.168.50.3
PING 192.168.50.3 (192.168.50.3) 56(84) bytes of data.
64 bytes from 192.168.50.3: icmp_seq=1 ttl=64 time=0.053 ms
64 bytes from 192.168.50.3: icmp_seq=2 ttl=64 time=0.031 ms
64 bytes from 192.168.50.3: icmp_seq=3 ttl=64 time=0.029 ms
```

```
如果需要访问本机的其它网段，可手动添加如下默认路由条目。
# ip netns exec msb ip route add default via 192.168.50.3
```

关于如何ping通外网主机，可设置路由转发完成。
