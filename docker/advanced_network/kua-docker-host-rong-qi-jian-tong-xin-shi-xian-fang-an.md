# 跨Docker Host容器间通信实现方案

{% tabs %}
{% tab title="Docker原生方案" %}
1.  overlay

    基于VXLAN封装实现Docker原生overlay网络
2.  macvlan

    Docker主机网卡接口逻辑上分为多个子接口，每个子接口标识一个VLAN，容器接口直接连接Docker Host
3.  网卡接口

    通过路由策略转发到另一台Docker Host
{% endtab %}

{% tab title="第三方方案" %}
**隧道方案**

* Flannel
  * 支持UDP和VLAN封装传输方式
* Weave
  * 支持UDP和VXLAN
* OpenvSwitch
  * 支持VXLAN和GRE协议

**路由方案**

* Calico
  * 支持BGP协议和IPIP隧道
  * 每台宿主机作为虚拟路由，通过BGP协议实现不同主机容器间通信。
{% endtab %}
{% endtabs %}



