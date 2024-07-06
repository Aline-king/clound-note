# 资源对象

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\


资源对象有两种形态：&#x20;

* 文件形态 - 编写出来的资源对象文件，有大量属性组成。&#x20;
* 对象形态 - 基于资源对象文件初始化后出来的应用对象。

资源对象初始化的方法主要有两大类：&#x20;

1.  命令行工具(kubectl)

    \- 通过纯粹的 "k8s命令及其选项" 来实现资源的创建&#x20;
2.  文件方式(API)&#x20;

    \- 基于 "k8s命令 + 配置文件" 来实现资源的创建

    \- 基于 "声明式的配置文件 + kubectl" 来实现资源的创建



1.  最基础资源对象

    pod/pods（po）
2.
3. **其他资源对象**

*

{% tabs %}
{% tab title="资源对象" %}
* pod/pods (po)
* node/nodes(no)
* replication controllers  rc
* horizontal pod autoscalers  hpa
* replica sets rs
* deployment deploy
* services svc
* namespaces ns
* daemon sets ds
* endpoints ep
* events ev
* ingresses  ing
* clusters
* stateful sets
* jobs
{% endtab %}

{% tab title="存储对象" %}
persistent volume pv

persistent volume claims pvc

storage classes sc

config maps cm

secrets
{% endtab %}

{% tab title="策略对象" %}
1. SecurityContext ResourceQuota LimitRange
2.
{% endtab %}

{% tab title="身份对象" %}
1. ServiceAccount
2. Role
3. ClusterRole
{% endtab %}
{% endtabs %}
