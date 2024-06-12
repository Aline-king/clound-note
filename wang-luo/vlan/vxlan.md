# VxLAN

VxLAN 全称是 Visual eXtensible Local Area Network（虚拟扩展本地局域网）

主要有 Cisco 推出，vxlan 是一个 VLAN 的扩展协议，是由 IETF 定义的 NVO3（Network Virtualization over Layer 3）标准技术之一

> 通信过程
>
> [https://support.huawei.com/enterprise/zh/doc/EDOC1100087027](https://support.huawei.com/enterprise/zh/doc/EDOC1100087027) [https://www.pianshen.com/article/1890293327/](https://www.pianshen.com/article/1890293327/)

VXLAN 的特点是将 L2 的以太帧封装到 UDP 报文（即 L2 over L4）中，并在 L3 网络中传输，即使用 MAC in UDP 的方法对报文进行重新封装, VxLAN 本质上是一种 overlay 的隧道封装技术，它将 L2 的以太网帧封装成 L4 的 UDP 数据报，然后在 L3 的网络中传输，效果就像 L2 的以太网帧在一个广播域中传输一样，实际上 L2 的以太网帧跨越了 L3 网络传输，但是缺不受 L3 网络的限制，vxlan 采用 24 位标识 vlan ID 号，因此可以支持 2^24=16777216 个 vlan，其可扩展性比 vlan 强大的多，可以支持大规模数据中心的网络需求。



vxlan 简单通信流程： 1.VM A 发送 L2 帧与 VM 请求与 VM B 通信。 2.源宿主机 VTEP 添加或者封装 VXLAN、UDP 及 IP 头部报文。 3.网络层设备将封装后的报文通过标准的报文在三层网络进行转发到目标主机。 4.目标宿主机 VTEP 删除或者解封装 VXLAN、UDP 及 IP 头部。 5.将原始 L2 帧发送给目标 VM。
