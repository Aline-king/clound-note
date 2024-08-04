# 位置加权负载均衡

## 一、位置加权负载均衡介绍 <a href="#yi-wei-zhi-jia-quan-fu-zai-jun-heng-jie-shao-1" id="yi-wei-zhi-jia-quan-fu-zai-jun-heng-jie-shao-1"></a>

在Envoy中，集群位置加权负载均衡（Weighted Load Balancing）是一种策略，用于根据各个上游节点的权重分配流量。通过配置不同的权重，Envoy能够优先将更多的流量分配给资源更充足或性能更高的节点。这种策略在实际应用中可以帮助优化资源利用，确保高效和可靠的服务。

Envoy中，通过设置每个上游节点的权重，集群位置加权负载均衡能够实现更细粒度的流量控制。权重越高的节点会接收到更多的流量。

以下是一个具体的配置示例，展示了如何在Envoy中设置集群位置加权负载均衡：

```yaml
static_resources:
  clusters:
  - name: weighted_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN  # 选择基本的负载均衡策略
    load_assignment:
      cluster_name: weighted_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service1.example.com
                port_value: 80
          load_balancing_weight:
            value: 80  # 权重80
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service2.example.com
                port_value: 80
          load_balancing_weight:
            value: 20  # 权重20
```

**配置项解释**

* **lb\_policy**：指定基本的负载均衡策略，这里使用 `ROUND_ROBIN`。
* **lb\_endpoints**：定义上游服务的节点。
* **load\_balancing\_weight**：设置节点的负载均衡权重，数值越大，分配的流量越多。

## 二、位置加权负载均衡机制 <a href="#er-wei-zhi-jia-quan-fu-zai-jun-heng-ji-zhi-8" id="er-wei-zhi-jia-quan-fu-zai-jun-heng-ji-zhi-8"></a>

位置加权负载均衡(Locality weighted load balancing) 的确是为特定的Locality及其相关的LbEndpoints分配权重，并根据这些权重在各Locality之间分配流量。以下是更详细的解释：

1. **定义权重**：为每个Locality及其相关的LbEndpoints显式赋予权重。
2. **流量分配**：在所有Locality的所有Endpoint都处于健康状态时，根据位置权重在各Locality之间进行加权轮询。

例如：

* 假设有两个region：cn-north-1和cn-north-2，它们的权重分别为1和2。
* 如果这两个region内的所有端点均处于健康状态，那么流量将根据权重进行分配。
* 在这个例子中，cn-north-1将接收33%的流量，而cn-north-2将接收67%的流量（比例为1:2）。

**启用位置加权负载均衡及位置权重定义的方法**

```powershell
cluster:
- name: ...
load assignment:
endpoints:
  locality:" {....}"
  Ib_endpoints":[]
  load_balancing_weight:“{}”# 整数值，定义当前位置或优先级的权重，最小值为1;
  priority:"..."
```

当某Locality的某些Endpoint不可用时，Envov则按比例动态调整该Locality的权重

当某个Locality（位置）的某些Endpoint（端点）不可用时，Envoy会按比例动态调整该Locality的权重。

位置加权负载均衡还支持为LbEndpoint（负载均衡端点）配置超配因子（默认为1.4）。Locality（位置）X的有效权重计算方式如下：

1. **健康比例计算**：\
   健康比例的计算公式是：\
   health(L\_X) = 1.4 \* (健康的X端点数量 / X端点的总数量)\
   这里，健康比例 = 超配因子 × 健康节点数 / 总节点数。
2. **有效权重计算**：\
   有效权重的计算公式是：\
   effective weight(L\_X) = locality\_weight\_X \* min(100, health(L\_X))\
   这里，有效权重 = 位置权重 × （100或健康比例，取较小值）。
3. **流量分配计算**：\
   流量分配的计算公式是：\
   load to L\_X = effective weight(L\_X) / (所有位置的有效权重之和)\
   这里，可调度的位置 = 有效权重 / 总权重。

**举个例子**

假设我们有两个集群，一个集群有2个节点，另一个集群有3个节点，初始权重比为1：2。如果3节点的集群中有1个节点出现故障，健康节点比例为2/3。使用超配因子1.4来计算：

1. **健康比例**：\
   health(L\_Y) = 1.4 \* (2 / 3) ≈ 0.93 或 93%
2. **有效权重**：\
   effective weight(L\_Y) = locality\_weight\_Y \* 0.93\
   如果初始权重是2，那么有效权重就是2 \* 0.93 = 1.86。
3. **流量分配**：\
   load to L\_Y = 1.86 / (1 + 1.86) ≈ 65%\
   load to L\_X = 1 / (1 + 1.86) ≈ 35%

所以，原本流量分配比例是1:2（33%:67%），但由于Y集群有一个节点故障，现在的比例调整为35%:65%。

**动态调整权重的例子**

假设位置X集群和Y集群的初始权重分别为1和2。如果Y集群的健康端点比例只有50%，则其健康比例计算为：

1. **健康比例**：\
   health(L\_Y) = 1.4 \* 0.5 = 0.7 或 70%
2. **有效权重**：\
   effective weight(L\_Y) = 2 \* 0.7 = 1.4
3. **流量分配**：\
   load to L\_Y = 1.4 / (1 + 1.4) ≈ 58%\
   load to L\_X = 1 / (1 + 1.4) ≈ 42%

流量分配比例从1:2（33%:67%）变为1:1.4（42%:58%）。

**配置优先级和权重**

如果同时配置了优先级和权重，负载均衡器将按照以下步骤进行调度选择：

1. 选择优先级。
2. 从选出的优先级中选择Locality。
3. 从选出的Locality中选择Endpoint。

这样确保流量优先分配到更高优先级、健康的节点上。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719843371035/9940a52e8ebe42aaa5875da4ddfbc9ef.png)

## 三、 位置加权负载均衡案例 <a href="#san-wei-zhi-jia-quan-fu-zai-jun-heng-an-li-32" id="san-wei-zhi-jia-quan-fu-zai-jun-heng-an-li-32"></a>

位置加权负载均衡 在Envoy配置中，位置加权负载均衡通常是通过在`clusters`部分`load_assignment`字段中配置`locality`来实现的。

这使得Envoy能够根据地理位置或数据中心的不同权重和优先级来分配流量，从而优化响应时间和资源利用率。 在提供的配置中，两个地理位置被配置为具有不同的权重，这些设置直接影响到负载均衡的决策：

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
