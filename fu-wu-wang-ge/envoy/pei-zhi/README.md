# 配置

## 加载配置方法

{% hint style="success" %}
**提示块标题**

{% tabs %}
{% tab title="从Bootstrap文件加载" %}
启动时从Bootstrap配置文件中加载初始配置，支持动态配置

<details>

<summary>xDS API</summary>



</details>
{% endtab %}

{% tab title="启用全动态配置机制" %}
启用全动态配置机制后，仅极少数场景需要重新启动Envoy进程

* 侦听器、集群等信息大部分使用动态加载配置，由于管理服务器不能动态配置，所以说是大部分动态加载配置
* 支持热重启
{% endtab %}
{% endtabs %}
{% endhint %}

*
*
  *
    * 从配置文件加载配置
    * 从管理服务器(Management Server)基于xds协议加载配置
  * runtime
    * 轻量化配置，小功能打开或关闭使用
    * 某些关键特性保存为key/value数据，通过runtime接口为key赋予不同的值，从而达到调整配置的目的。
    * 支持多层配置和覆盖机制
*
