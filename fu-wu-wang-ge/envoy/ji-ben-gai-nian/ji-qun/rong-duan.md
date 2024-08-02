# 熔断

## 一、熔断介绍 <a href="#yi-rong-duan-jie-shao-1" id="yi-rong-duan-jie-shao-1"></a>

### 1.1 什么是熔断 <a href="#id-11-shen-me-shi-rong-duan-2" id="id-11-shen-me-shi-rong-duan-2"></a>

在Envoy中，熔断（Circuit Breaking）是一种保护机制，用于防止上游服务（upstream services）过载，从而确保系统的整体稳定性和可用性。熔断器会在检测到上游服务异常负载或故障率过高时，自动限制或中断对该服务的请求，以防止故障蔓延和资源耗尽。

### 1.2 熔断的工作原理 <a href="#id-12-rong-duan-de-gong-zuo-yuan-li-4" id="id-12-rong-duan-de-gong-zuo-yuan-li-4"></a>

熔断器的工作原理类似于电路中的保险丝，当检测到异常情况（如请求失败率高、响应时间长等）时，熔断器会“跳闸”，暂时停止向故障的上游服务发送请求。在一段时间后，熔断器会尝试恢复对上游服务的请求，如果服务恢复正常，则重新开启流量，否则继续保持熔断状态。

### 1.3 配置熔断器 <a href="#id-13-pei-zhi-rong-duan-qi-6" id="id-13-pei-zhi-rong-duan-qi-6"></a>

在Envoy中，熔断器的配置可以在集群级别进行。配置项包括最大连接数、最大并发请求数、最大请求数、最大重试次数等。以下是配置熔断器的示例：

**示例配置**

```yaml
static_resources:
  clusters:
  - name: example_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: example_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: example.com
                port_value: 80
    circuit_breakers:
      thresholds:
      - priority: DEFAULT
        max_connections: 100
        max_pending_requests: 1000
        max_requests: 5000
        max_retries: 3
      - priority: HIGH
        max_connections: 200
        max_pending_requests: 2000
        max_requests: 10000
        max_retries: 5
```

**配置项解释**

* **priority**：定义优先级，Envoy支持两种优先级：`DEFAULT` 和 `HIGH`。可以为不同的优先级配置不同的熔断策略。
  * **默认流量（DEFAULT）**：适用于大多数普通请求，保证在常规负载下的服务稳定性。
  * **高优先级流量（HIGH）**：适用于关键任务或实时请求，提供更高的资源限制，确保这些关键请求在高负载情况下仍然能够得到处理。
* **max\_connections**：允许的最大连接数。当达到此限制时，新的连接请求将被拒绝。
* **max\_pending\_requests**：允许的最大挂起请求数。当达到此限制时，新的请求将被拒绝。
* **max\_requests**：允许的最大请求数。当达到此限制时，新的请求将被拒绝。
* **max\_retries**：允许的最大重试次数。当达到此限制时，将不会再尝试重试失败的请求。

### 1.4 熔断的优势 <a href="#id-14-rong-duan-de-you-shi-12" id="id-14-rong-duan-de-you-shi-12"></a>

1. **防止服务过载**：通过限制请求和连接数，防止上游服务因过载而崩溃。
2. **提高系统稳定性**：避免故障蔓延，确保其他服务的正常运行。
3. **自动恢复**：熔断器在一段时间后会尝试恢复对上游服务的请求，确保服务恢复正常后重新提供服务。

### 1.5 监控和调试 <a href="#id-15-jian-kong-he-tiao-shi-14" id="id-15-jian-kong-he-tiao-shi-14"></a>

Envoy提供了丰富的监控指标，可以通过/admin接口查看熔断器的状态和统计信息。例如：

* **`cluster.<cluster_name>.circuit_breakers.default.cx_open`**：当前打开的连接数。
* **`cluster.<cluster_name>.circuit_breakers.default.rq_pending_open`**：当前打开的挂起请求数。
* **`cluster.<cluster_name>.circuit_breakers.default.rq_open`**：当前打开的请求数。
* **`cluster.<cluster_name>.circuit_breakers.default.rq_retry_open`**：当前打开的重试请求数。

这些统计数据可以通过访问Envoy的admin接口获取，例如 `http://localhost:9901/stats`。

熔断器是Envoy中一项关键的保护机制，通过限制上游服务的请求和连接数，防止服务过载和故障蔓延。通过配置熔断器，可以提高系统的稳定性和可靠性，确保在上游服务发生异常时，整个系统仍能正常运行。

### 1.6 连接池 <a href="#id-16-lian-jie-chi-19" id="id-16-lian-jie-chi-19"></a>

在Envoy中，集群连接池（Cluster Connection Pool）是一个管理与上游服务（upstream services）之间连接的组件。连接池用于重用现有连接，以减少连接建立和拆除的开销，从而提高性能和效率。连接池可以配置为不同类型的协议，如HTTP1、HTTP2和TCP。

#### 1.6.1 连接池的作用 <a href="#id-161-lian-jie-chi-de-zuo-yong-21" id="id-161-lian-jie-chi-de-zuo-yong-21"></a>

1. **减少连接开销**：通过重用现有连接，减少连接建立和拆除的频率，降低延迟和资源消耗。
2. **提高吞吐量**：连接池可以同时维护多个连接，提升并发处理能力。
3. **优化资源使用**：通过限制连接数，防止资源过度使用和服务过载。

#### 1.6.2 配置连接池 <a href="#id-162-pei-zhi-lian-jie-chi-23" id="id-162-pei-zhi-lian-jie-chi-23"></a>

连接池配置通常在集群配置中指定。下面是一些常见的连接池配置示例，包括HTTP1、HTTP2和TCP连接池的配置。



{% tabs %}
{% tab title="HTTP1 连接池配置" %}
```yaml
static_resources:
  clusters:
  - name: http1_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: http1_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: http1-service.example.com
                port_value: 80
    http_protocol_options:
      max_connections: 100
      max_pending_requests: 1000
      max_requests_per_connection: 50
      idle_timeout: 1s
```


{% endtab %}

{% tab title="HTTP2 连接池配置" %}
```yaml
static_resources:
  clusters:
  - name: http2_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: http2_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: http2-service.example.com
                port_value: 80
    http2_protocol_options: {}  # 启用HTTP2
    circuit_breakers:
      thresholds:
      - priority: DEFAULT
        max_connections: 100
        max_pending_requests: 1000
        max_requests: 5000
```


{% endtab %}

{% tab title="TCP 连接池配置" %}
```yaml
static_resources:
  clusters:
  - name: tcp_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: tcp_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: tcp-service.example.com
                port_value: 9000
    upstream_connection_options:
      tcp_keepalive:
        keepalive_time: 300
        keepalive_interval: 60
        keepalive_probes: 5
```


{% endtab %}
{% endtabs %}

**配置项解释**

* **max\_connections**：连接池中允许的最大连接数。
* **max\_pending\_requests**：连接池中允许的最大挂起请求数。
* **max\_requests\_per\_connection**：每个连接允许处理的最大请求数。适用于HTTP1连接池。
* **idle\_timeout**：连接的空闲超时时间。如果连接在此时间内没有活动，会被关闭。
* **http2\_protocol\_options**：启用HTTP2协议的连接池配置。
* **tcp\_keepalive**：TCP连接的keep-alive选项，包括保持时间、间隔和探测次数。
