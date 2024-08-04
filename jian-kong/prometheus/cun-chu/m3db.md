# m3db

> [github上的m3db](https://github.com/m3db/m3)
>
> [官方网站](https://m3db.io/)

一款具有反向索引的时间序列数据库

**历史**

* 2014年，经过数年与基于Graphite和WhisperDB的传统指标平台的对抗之后，Uber的Observability团队开始开发一个开源指标平台
* 该平台将被称为M3。该项目的目标是为团队提供一个高度可用的集中式指标平台，该平台可以轻松地与现有工具集成。

**存储发展过程**存储层的第一个迭代以Cassandra和ElasticSearch为特色，但是随着打车业务进入高速增长阶段，在Graphite和WhisperDB年代曾困扰团队的交火重新浮出水面。然后，团队决定从头开始构建M3DB，这是一个具有嵌入式**反向索引（倒排索引）**的自定义时间序列数据库，这成为了平台的基础。在大约一年半的时间里，M3DB是在Uber上开发和推出的。

## 架构图

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

组件介绍

{% tabs %}
{% tab title="M3 Coordinator" %}
* M3协调器是一项服务，用于协调上游系统（例如Prometheus）和M3DB之间的读写
* 它是用户可以部署以访问M3DB好处的桥梁，例如长期存储和与其他监视系统（例如Prometheus）的多DC设置
{% endtab %}

{% tab title="m3db" %}
* M3DB是一个分布式时间序列数据库，提供可伸缩的存储和时间序列的反向索引
* 它被优化为具有成本效益的，可靠的实时和长期保留指标存储和索引
{% endtab %}

{% tab title="m3query" %}
M3 Query是一个分布式查询引擎，用于查询存储在M3DB节点中的实时和历史数据，支持多种查询语言

它旨在支持低延迟的实时查询和执行时间可能更长的查询，这些数据可以聚合用于分析用例的较大数据集

例如，如果您将Prometheus远程写端点与M3 Coordinator一起使用，则可以使用M3 Query代替Prometheus远程读端点

这样，您可以获得M3 Query引擎的所有好处，例如块处理。由于M3 Query提供了与Prometheus兼容的API，因此您可以使用第三方制图和警报解决方案，例如Grafana
{% endtab %}

{% tab title="m3aggregator" %}
M3聚合器是专用的**度量标准聚合器**，可在将度量标准存储在M3DB节点中之前提供基于状态的基于流的下采样。它使用存储在etcd中的动态规则。

它使用领导者选举和聚集窗口跟踪，利用etcd来管理此状态，以可靠的方式将一次至少一次的聚集可靠地发送给长期存储的降采样指标。这提供了具有成本效益的，可靠的下采样和指标汇总。

M3协调器也可以执行此角色，但是M3聚合器可以分片和复制指标，而M3协调器则不是，并且需要谨慎地以高度可用的方式进行部署和运行。

M3DB相似，M3 Aggregator默认情况下支持集群和复制。这意味着度量标准已正确路由到负责聚合每个度量标准的实例，并且您可以配置多个M3 Aggregator副本，以使聚合没有单点故障。
{% endtab %}
{% endtabs %}
