---
description: 虚拟局域网
---

# Vlan

VLAN（Virtual Local Area Network）即虚拟局域网，

是将一个物理(交换机)的网络在逻辑上划分成多个广播域的通信技术，VLAN 内的主机间可以直接通信，而 VLAN 网络外的主机需要通过三层网络设备转发才可以通信，因此一个 vlan 可以将服务器的广播报文限制在一个 VLAN 内，从而降低单个网络环境中的广播报文，vlan 采用 12 位标识 vlan ID，即一个交换机设备最多为 2^12=4096 个 vlan。
