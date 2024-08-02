# 负载均衡器策略

## 一、 负载均衡器策略 <a href="#yi-fu-zai-jun-heng-qi-ce-le-1" id="yi-fu-zai-jun-heng-qi-ce-le-1"></a>

在Envoy中，负载均衡（Load Balancing）策略用于决定如何在多个上游（upstream）服务实例之间分配传入的请求。Envoy支持多种负载均衡策略，每种策略都有其独特的行为和应用场景。以下是Envoy支持的一些主要负载均衡策略：

<details>

<summary>ROUND_ROBIN（轮询）</summary>

**Round Robin**策略按顺序循环地将请求分配给每个上游实例。它是一种简单且常见的负载均衡策略，适用于请求量均匀且服务实例性能相似的场景。

```
lb_policy: ROUND_ROBIN
```

</details>

<details>

<summary>LEAST_REQUEST（最少请求）</summary>

**Least Request**策略将请求分配给当前处理请求数量最少的上游实例。这种策略适用于负载分布不均匀的场景，有助于平衡负载并提高整体性能。

```
lb_policy: LEAST_REQUEST
```

</details>

<details>

<summary>RANDOM（随机）</summary>

**Random**策略随机选择一个上游实例来处理请求。这种策略适用于简单的负载均衡需求，不需要任何复杂计算。

```
lb_policy: RANDOM
```

</details>

<details>

<summary>RING_HASH（环哈希）</summary>

**Ring Hash**策略使用一致性哈希将请求分配给上游实例，适用于需要会话粘性或状态持久性的场景。该策略常用于缓存服务，以确保相同的请求始终路由到相同的上游实例

```
lb_policy: RING_HASH
```

</details>

<details>

<summary><strong>Maglev</strong></summary>

**Maglev**策略也是一种基于哈希的负载均衡算法，设计用于在更改上游集群时提供较低的请求重新分配率。这有助于实现更平滑的负载转移。

其名称源自Google的Maglev项目，该项目旨在提供高性能、低延迟的负载均衡。Maglev算法通过优化一致性哈希的实现，减少了节点变更时的请求重新分配率，从而提高了负载均衡的稳定性和效率。

**Maglev负载均衡策略的特点**

1. **低重新分配率**：在上游节点发生变化（增加或减少）时，Maglev算法能最大限度地减少请求的重新分配。
2. **高效率**：Maglev能够快速计算目标节点，从而实现高性能的请求分配。
3. **一致性哈希**：Maglev采用一致性哈希技术，确保相同的请求尽量路由到相同的上游节点。

以下是Envoy中使用Maglev负载均衡策略的配置示例：

```yaml
static_resources:
  clusters:
  - name: maglev_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: MAGLEV
    load_assignment:
      cluster_name: maglev_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service1.example.com
                port_value: 80
        - endpoint:
            address:
              socket_address:
                address: service2.example.com
                port_value: 80
```

在这个示例中，我们配置了一个名为 `maglev_service_cluster`的集群，并指定其负载均衡策略为 `MAGLEV`。Envoy会根据Maglev算法在 `service1.example.com`和 `service2.example.com`之间分配流量。

**Maglev算法的工作原理**

Maglev算法的核心在于创建一个哈希环并将请求分配到环上的节点。它通过以下步骤工作：

1. **初始化哈希环**：在启动时，根据上游节点的数量和配置的哈希函数初始化一个哈希环。
2. **哈希映射**：每个上游节点在环上占据多个位置，通过哈希函数将这些位置映射到节点。
3. **请求分配**：对于每个传入的请求，计算其哈希值，并在环上找到最近的节点位置，将请求分配给该节点。

Maglev的关键在于减少节点变更时的请求重新分配。当有节点加入或离开时，只会重新分配受影响的部分请求，而不是全部请求。

**使用场景**

Maglev负载均衡策略适用于以下场景：

1. **大规模分布式系统**：需要在多个数据中心或区域之间均匀分配负载，且节点数量较多。
2. **高稳定性要求**：需要在节点变更时保持较低的请求重新分配率，以减少服务中断。
3. **一致性需求**：需要确保相同的请求尽量路由到相同的上游节点，适用于缓存服务、会话保持等场景。

Maglev负载均衡策略通过优化一致性哈希算法，提供了高效、低延迟、低重新分配率的负载均衡方案。它在大规模分布式系统和高稳定性、高一致性要求的场景中表现尤为出色。在Envoy中，通过简单的配置即可使用Maglev策略，实现稳定高效的请求分配。

```
lb_policy: MAGLEV
```

</details>

<details>

<summary><strong>WEIGHTED_LEAST_REQUEST（加权最少请求）</strong></summary>

**Weighted Least Request**策略是在最少请求策略的基础上增加了权重因素。上游实例可以根据其权重和当前负载来决定请求的分配。适用于实例性能不同的场景，通过权重调整分配比例。

```
lb_policy: LEAST_REQUEST
least_request_lb_config:
  choice_count: 2
  active_request_bias:
    default_value: 1.0
    runtime_key: "new_active_request_bias"
```

</details>

<details>

<summary><strong>ORIGINAL_DST（原始目标）</strong></summary>

**Original Destination**策略将请求直接路由到原始目标地址，而不进行任何负载均衡。这种策略适用于需要保留原始目标地址的场景。

```
lb_policy: ORIGINAL_DST_LB
original_dst_lb_config:
  use_http_header: true
```

</details>

<details>

<summary><strong>CLUSTER_PROVIDED（集群提供）</strong></summary>

**Cluster Provided**策略允许集群自己决定负载均衡策略，通常用于集成自定义负载均衡逻辑的场景。

```
lb_policy: CLUSTER_PROVIDED
```

</details>

<details>

<summary><strong>LOAD_BALANCING_POLICY（动态负载均衡策略）</strong></summary>

Envoy还支持通过LoadBalancingPolicy配置动态选择负载均衡策略，允许在运行时调整策略。

**配置示例**

```yaml
load_balancing_policy:
  policies:
    - policy:
        typed_extension_config:
          name: envoy.load_balancing_policies.round_robin
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.load_balancing_policies.round_robin.v3.RoundRobin
```

**完整示例配置**

```yaml
static_resources:
  clusters:
  - name: example_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: LEAST_REQUEST
    load_assignment:
      cluster_name: example_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service.example.com
                port_value: 80
    least_request_lb_config:
      choice_count: 2
      active_request_bias:
        default_value: 1.0
        runtime_key: "new_active_request_bias"
```

Envoy提供了多种负载均衡策略，以满足不同的应用场景和需求。从简单的轮询和随机策略，到复杂的哈希和最少请求策略，每种策略都有其特定的优势和适用场景。通过灵活选择和配置负载均衡策略，Envoy能够优化请求分配，提高服务的可靠性和性能。

</details>

## 二、 全局负载均衡及分布式负载均衡 <a href="#er-quan-ju-fu-zai-jun-heng-ji-fen-bu-shi-fu-zai-jun-heng-56" id="er-quan-ju-fu-zai-jun-heng-ji-fen-bu-shi-fu-zai-jun-heng-56"></a>

在Envoy中，全局负载均衡和分布式负载均衡是两种不同的负载均衡策略，它们在工作机制和适用场景上有所不同。

{% tabs %}
{% tab title="全局负载均衡（Global Load Balancing）" %}
全局负载均衡是指在整个服务网格中实现跨多个区域或数据中心的负载均衡。它关注的是如何在多个地理位置上的实例之间分配流量，以实现高可用性和灾备能力。

**特点：**

* **跨区域**：流量可以在不同的区域或数据中心之间分配。
* **高可用性**：在某个区域或数据中心出现故障时，可以将流量分配到其他区域。
* **灾备能力**：确保即使在大规模故障发生时，服务仍然可用。
{% endtab %}

{% tab title="分布式负载均衡（Local Load Balancing）" %}
分布式负载均衡是指在本地服务实例之间进行流量分配。它主要关注的是如何在单个区域或数据中心内部的多个实例之间均匀分配流量，以优化资源利用和性能。

**特点：**

* **本地性**：流量在本地实例之间分配。
* **低延迟**：由于负载均衡发生在同一区域或数据中心，网络延迟较低。
* **资源优化**：在本地实例之间均匀分配负载，优化资源利用。
{% endtab %}
{% endtabs %}

### 2.3 Envoy的负载均衡策略分类 <a href="#id-23envoy-de-fu-zai-jun-heng-ce-le-fen-lei-66" id="id-23envoy-de-fu-zai-jun-heng-ce-le-fen-lei-66"></a>

Envoy提供的负载均衡策略可以分为全局负载均衡和分布式负载均衡。以下是对上述负载均衡算法的分类：

{% tabs %}
{% tab title="全局负载均衡策略" %}
* **RING\_HASH**：环哈希策略，适用于需要会话粘性或跨数据中心的一致性哈希。
* **MAGLEV**：Maglev负载均衡策略，设计用于低重新分配率的跨区域流量分配。
{% endtab %}

{% tab title="分布式负载均衡策略" %}
* **ROUND\_ROBIN**：轮询策略，适用于简单的本地负载均衡。
* **LEAST\_REQUEST**：最少请求策略，适用于需要优化本地负载的场景。
* **RANDOM**：随机策略，适用于简单且均匀分布的本地负载均衡。
* **WEIGHTED\_LEAST\_REQUEST**：加权最少请求策略，结合权重和最少请求进行本地负载均衡。
* **ORIGINAL\_DST**：原始目标策略，保持请求的原始目标地址，用于本地的特殊场景。
* **CLUSTER\_PROVIDED**：集群提供策略，允许集群自行决定本地负载均衡逻辑。
{% endtab %}
{% endtabs %}

**具体示例与理解**



{% tabs %}
{% tab title="全局负载均衡示例" %}
假设有一个服务需要在多个区域进行负载均衡，可以使用RING\_HASH策略：

```yaml
static_resources:
  clusters:
  - name: global_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: RING_HASH
    load_assignment:
      cluster_name: global_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: global-service-region1.example.com
                port_value: 80
        - endpoint:
            address:
              socket_address:
                address: global-service-region2.example.com
                port_value: 80
```


{% endtab %}

{% tab title="分布式负载均衡示例" %}
对于在单个数据中心内进行负载均衡，可以使用LEAST\_REQUEST策略：

```yaml
static_resources:
  clusters:
  - name: local_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: LEAST_REQUEST
    load_assignment:
      cluster_name: local_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: local-service.example.com
                port_value: 80
```


{% endtab %}
{% endtabs %}

* **全局负载均衡**关注跨区域或数据中心的流量分配，常用的策略有RING\_HASH和MAGLEV。
* **分布式负载均衡**关注本地实例之间的流量分配，常用的策略有ROUND\_ROBIN、LEAST\_REQUEST、RANDOM等。

通过选择合适的负载均衡策略，Envoy能够实现高效的流量分配，满足不同的业务需求和性能要求。在实际应用中，可以根据具体的场景和需求，灵活配置和组合这些负载均衡策略
