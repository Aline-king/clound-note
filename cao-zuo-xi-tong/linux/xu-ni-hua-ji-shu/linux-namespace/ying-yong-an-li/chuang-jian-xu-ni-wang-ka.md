# 创建虚拟网卡

同时创建一对虚拟网卡

{% code title="创建虚拟网卡对" %}
```bash
# ip link add veth0 type veth peer name veth1
```
{% endcode %}

{% code title="在物理机上查看" %}
```bash
# ip a s
......
10: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether de:44:f8:b7:12:65 brd ff:ff:ff:ff:ff:ff
11: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 46:5e:89:8c:cb:b3 brd ff:ff:ff:ff:ff:ff
```
{% endcode %}
