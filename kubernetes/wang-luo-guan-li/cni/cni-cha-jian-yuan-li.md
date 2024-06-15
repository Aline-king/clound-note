---
coverY: 0
---

# CNI插件原理



对容器网络的设置和操作都通过插件（Plugin）进行具体实现，CNI插件包括两种类型：CNIPlugin和IPAM（IP Address Management）Plugin。

* CNI Plugin负责为容器配置网络资源，
* IPAM Plugin负责对容器 的IP地址进行分配和管理。IPAM Plugin作为CNI Plugin的一部分，与CNI Plugin一起工作。

1. Main插件：<mark style="color:green;">实现某种特定的网络功能，如loopback、bridge、macvlan、ipvlan</mark>
   1. bridge： 在宿主机上创建网桥然后通过veth pair的方式连接到容器
   2. macvlan：虚拟出多个macvtap，每个macvtap都有不同的mac地址
   3. ipvlan：和macvlan相似，也是通过一个主机接口虚拟出多个虚拟网络接口，不同的是ipvlan虚拟出来的是共享MAC地址，ip地址不同
   4. loopback： lo设备（将回环接口设置成up）
   5. ptp： Veth Pair设备
   6. vlan： 分配vlan设备
   7. host-device： 移动宿主上已经存在的设备到容器中
2. IPAM(IP Address Management)插件: <mark style="color:yellow;">仅用于分配IP地址，不提供网络实现</mark>
   1. dhcp： 宿主机上运行的守护进程，代表容器发出DHCP请求
   2. host-local： 使用提前分配好的IP地址段来分配
   3. static：用于为容器分配静态的IP地址，主要是调试使用
3. Meta插件： 由CNI社区维护的内部插件，<mark style="color:purple;">自身不提供任何网络实现，而是调用其他插件，如flannel</mark>
   1. flannel: 专门为Flannel项目提供的插件
   2. tuning：通过sysctl调整网络设备参数的二进制文件
   3. portmap：通过iptables配置端口映射的二进制文件
   4. bandwidth：使用 Token Bucket Filter (TBF)来进行限流的二进制文件
   5. firewall：通过iptables或者firewalled添加规则控制容器的进出流量
