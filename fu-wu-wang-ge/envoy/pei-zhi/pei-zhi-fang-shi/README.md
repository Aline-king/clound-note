# 配置方式

{% hint style="info" %}


Envoy架构支持非常灵活的配置方式：

{% tabs %}
{% tab title="纯静态配置- 简单场景" %}
用户自行提供侦听器、过滤器链、集群及HTTP路由（http代理场景），上游站点的发现仅可通过DNS服务进行，且配置的重新加载必须通过内置的热重启完成。
{% endtab %}

{% tab title="动静混合配置" %}
11

<details>

<summary>仅使用EDS</summary>

EDS提供的端点发现功能可有效规避DNS的限制（响应中的最大记录数等），只有Endpoint是动态生成的，Cluster及Listener都是静态配置的。

</details>

<details>

<summary>使用EDS和CDS</summary>

CDS能够让Envoy以优雅的方式添加、更新和删除上游集群，于是，初始配置时，Envoy无需事先了解所有上游集群，Endpoint和Cluster是动态生成的，Listener是静态配置的。

</details>

<details>

<summary>EDS、CDS和RDS</summary>

动态发现路由配置，RDS与EDS、CDS一起使用时，为用户提供了构建复杂路由拓扑的能力（流量转移、蓝/绿部署等），Endpoint、Cluster、Router都是动态的，Listener是静态配置的。

</details>
{% endtab %}

{% tab title="添加动态配置机制" %}
适用于复杂场景

* EDS、CDS、RDS和LDS：动态发现侦听器配置、包括内嵌的过滤器链；启用此四种服务发现后，除了较罕见的配置变动、证书轮替或更新Envoy程序之外，几乎无需再热重启Envoy
* EDS、CDS、RDS、LDS和SDS:动态发现侦听器密钥相关的证书、私钥及TLS会话票据，以及对证书验证逻辑的配置（受信任的根证书和撤销机制等）
{% endtab %}
{% endtabs %}
{% endhint %}
