# 在net命名空间中执行多条命令

<pre class="language-bash" data-title="在网络命名空间中查看路由表"><code class="lang-bash"><strong># route -n
</strong>Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
</code></pre>

{% code title="在网络命名空间中查看防火墙规则" %}
```bash
# iptables -t nat -nL
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
```
{% endcode %}
