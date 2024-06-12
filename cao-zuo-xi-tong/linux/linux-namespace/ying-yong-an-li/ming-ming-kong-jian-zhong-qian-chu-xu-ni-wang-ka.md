# 命名空间中迁出虚拟网卡

{% code title="在Linux系统命令行把虚拟网卡veth1从网络命名空间删除" %}
```bash
# ip netns exec msb ip link delete veth1
```
{% endcode %}

```bash
在Linux系统命令行查看结果
# ip netns exec msb ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
