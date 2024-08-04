# 集群管理器 cluster manager



{% tabs %}
{% tab title="介绍" %}
<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Envoy中的集群管理器（Cluster Manager）是一个关键组件，用于管理和维护所有上游集群（upstream clusters）。它负责维护集群的健康状态、执行负载均衡策略、路由请求等。
{% endtab %}

{% tab title="主要作用" %}
1. **集群发现和维护**：\
   集群管理器负责发现和维护集群。这包括动态添加和删除集群，以及维护集群的健康状态。
2. **负载均衡**：\
   集群管理器实现多种负载均衡策略，如轮询、最小请求、随机等。它负责在多个上游节点之间分发请求，以实现高可用性和优化资源利用。
3. **健康检查**：\
   集群管理器定期对集群中的各个节点进行健康检查，以确保只有健康的节点参与流量处理。健康检查可以是主动的（如发送ping请求）或被动的（如根据请求失败率判断）。
4. **路由请求**：\
   根据配置，集群管理器将传入的请求路由到适当的上游集群。这通常结合Envoy的路由规则和负载均衡策略来实现。
5. **熔断器（Circuit Breakers）**：\
   集群管理器实现熔断器机制，当检测到某个集群或节点故障率过高时，临时停止向其发送请求，以防止系统过载和雪崩效应。
{% endtab %}

{% tab title="使用方法" %}
要使用集群管理器，需要在Envoy的配置文件中定义上游集群。以下是一个简单的配置示例：

```yaml
static_resources:
  clusters:
  - name: service_cluster  # 定义集群的名称。
    connect_timeout: 0.25s # 接超时时间。
    type: STRICT_DNS # 集群类型，可以是 STATIC、STRICT_DNS、LOGICAL_DNS或 EDS。STRICT_DNS表示通过DNS解析来动态发现集群成员。
    lb_policy: ROUND_ROBIN
    load_assignment: # 定义集群成员的地址信息。对于 STRICT_DNS类型，这里通常是域名和端口。
      cluster_name: service_cluster
      endpoints:
      - lb_endpoints: # 负载均衡策略，例如 ROUND_ROBIN（轮询）、LEAST_REQUEST（最小请求）、RANDOM（随机）等。
        - endpoint:
            address:
              socket_address:
                address: service1.example.com
                port_value: 80
```
{% endtab %}

{% tab title="动态集群管理" %}
Envoy支持通过xDS API（如CDS，Cluster Discovery Service）进行动态集群管理。使用xDS API，可以在运行时动态添加、更新或删除集群，而无需重启Envoy实例。



```yaml
dynamic_resources:
  cds_config:
    api_config_source:
      api_type: GRPC
      grpc_services:
        envoy_grpc:
          cluster_name: xds_cluster

static_resources:
  clusters:
  - name: xds_cluster
    type: STATIC
    connect_timeout: 0.25s
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: xds_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: xdsserver
                port_value: 18000
```
{% endtab %}
{% endtabs %}



**动态资源配置解释**

* `cds_config`: 配置CDS，用于动态管理集群。
* `api_config_source`: 指定API类型（如 `GRPC`），以及用于与xDS服务通信的集群名称。
* `static_resources`中的 `clusters`: 配置用于与xDS服务通信的集群。

通过这些配置，Envoy可以在运行时从xDS服务获取最新的集群信息，动态调整上游集群配置。

### 1.4 监控和调试 <a href="#id-14-jian-kong-he-tiao-shi-18" id="id-14-jian-kong-he-tiao-shi-18"></a>

Envoy提供了一些工具和接口来监控和调试集群管理器的状态，例如：

* **Admin接口**：可以通过/admin接口查看集群状态、健康检查结果等。
* **统计数据**：Envoy会生成丰富的统计数据（metrics），包括集群级别的请求数、失败率、健康状态等，可以通过Prometheus等监控系统进行收集和展示。

总的来说，Envoy的集群管理器通过灵活的配置和强大的动态管理能力，帮助用户实现高效、可靠的上游集群管理，从而提升整个服务网格的可用性和性能。

## 二、 集群预热 <a href="#er-ji-qun-yu-re-22" id="er-ji-qun-yu-re-22"></a>

在Envoy中，<mark style="color:blue;">**集群预热**</mark>（Cluster Warming）是指在将新集群或更新后的集群正式投入使用之前，确保它们已经成功建立连接并准备好接收流量的过程。

集群预热的目的是<mark style="color:orange;">**避免将请求发送到尚未完全准备好的集群节点，**</mark>从而提高服务的可靠性和稳定性。

### 2.1 集群预热的工作机制 <a href="#id-21-ji-qun-yu-re-de-gong-zuo-ji-zhi-24" id="id-21-ji-qun-yu-re-de-gong-zuo-ji-zhi-24"></a>

当Envoy检测到新集群或集群配置更新时，它不会立即开始将流量路由到该集群，而是首先执行以下步骤：

1. **连接建立**：Envoy尝试与集群中的所有上游节点建立连接。这包括执行必要的DNS解析、TCP连接建立以及TLS握手等过程。
2. **健康检查**：如果配置了健康检查，Envoy会对新集群中的节点执行健康检查，确保它们处于健康状态。
3. **等待足够的节点准备好**：Envoy会等待一定数量的节点成功建立连接并通过健康检查，这个数量由配置中的 `preconnect`参数或默认策略决定。
4. **预热完成**：当上述条件满足后，Envoy会认为集群已经预热完成，并开始将流量路由到该集群。

### 2.2 配置示例 <a href="#id-22-pei-zhi-shi-li-27" id="id-22-pei-zhi-shi-li-27"></a>

下面是一个包含集群预热配置的示例：

> 集群预热是Envoy提供的一个重要功能，帮助用户在动态和复杂的服务网格环境中，确保流量路由的稳定性和可靠性。

```yaml
static_resources:
  clusters:
  - name: new_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: new_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: newservice.example.com
                port_value: 80
    health_checks:
    - timeout: 1s
      interval: 10s
      unhealthy_threshold: 2
      healthy_threshold: 3
      tcp_health_check: {}
```

在这个示例中，集群 `new_service_cluster`配置了健康检查。在预热过程中，Envoy会尝试与 `newservice.example.com`的节点建立连接并执行健康检查。在足够多的节点准备好之前，Envoy不会将流量路由到这个集群。

**优势**

集群预热的主要优势包括：

* **避免请求失败**：在集群节点尚未完全准备好之前不会接收请求，减少因节点未准备好导致的请求失败。
* **提升服务可靠性**：确保只有健康的节点参与流量处理，提高整体服务的可靠性和稳定性。
* **平滑的配置更新**：在动态集群管理中，预热可以确保配置更新不会立即影响流量，使得新配置逐步生效。

## 三、集群配置框架 <a href="#san-ji-qun-pei-zhi-kuang-jia-35" id="san-ji-qun-pei-zhi-kuang-jia-35"></a>

> 单个集群配置，v3格式集群配置内容

在Envoy的v3配置中，集群配置框架涉及多个重要部分，包括集群的基本信息、负载均衡策略、健康检查、连接设置等。

下面是一个使用v3格式配置单个集群的完整示例，并解释了每个配置项的作用：

> 这个配置示例展示了如何使用Envoy的v3配置格式来定义一个集群的详细信息。通过调整这些配置项，可以实现对集群的精细管理和优化。

```yaml
static_resources:
  clusters:
  - name: example_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: example_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: example.com
                port_value: 80
    health_checks:
    - timeout: 1s
      interval: 10s
      unhealthy_threshold: 2
      healthy_threshold: 3
      tcp_health_check: {}
    circuit_breakers:
      thresholds:
      - priority: DEFAULT
        max_connections: 100
        max_pending_requests: 1000
        max_requests: 1000
        max_retries: 3
    outlier_detection:
      consecutive_5xx: 5
      interval: 10s
      base_ejection_time: 30s
      max_ejection_percent: 50
    dns_refresh_rate: 5s
    dns_lookup_family: V4_ONLY
    http2_protocol_options: {}  # 如果使用HTTP/2，请配置此项
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
