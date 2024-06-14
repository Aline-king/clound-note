# 迁移虚拟网卡到命名空间中

这两个网卡还都属于“default”或“global”命名空间，和物理网卡一样。把其中一个网卡转移到命名空间msb中。

<pre class="language-bash" data-title="把创建的veth1网卡添加到msb网络命名空间中"><code class="lang-bash"><strong># ip link set veth1 netns msb
</strong></code></pre>

<pre class="language-bash" data-title="在Linux系统命令行查看网络命名空间中的网络"><code class="lang-bash"><strong># ip netns exec msb ip link
</strong>1: lo: &#x3C;LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
10: veth1@if11: &#x3C;BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether de:44:f8:b7:12:65 brd ff:ff:ff:ff:ff:ff link-netnsid 0
</code></pre>
