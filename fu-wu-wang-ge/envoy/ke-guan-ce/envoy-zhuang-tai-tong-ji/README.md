# Envoy状态统计

## 涉及方面

Envoy 是一个高性能的分布式代理，广泛用于微服务架构中，提供负载均衡、服务发现、流量管理和观察等功能。Envoy 通过丰富的状态统计信息帮助用户监控和分析流量行为及系统性能。

以下是 Envoy 状态统计的几个主要方面：

{% tabs %}
{% tab title="HTTP 请求统计" %}
* **请求计数**：总请求数、成功请求数、失败请求数等。
* **请求延迟**：请求处理时间的分布（如平均延迟、P95延迟、P99延迟等）。
* **请求大小**：请求和响应的大小分布。
{% endtab %}

{% tab title="TCP 连接统计" %}
* **连接数**：总连接数、活动连接数、关闭连接数等。
* **连接时长**：连接存续时间的分布。
{% endtab %}

{% tab title="健康检查统计" %}
* **成功率**：健康检查的成功次数和失败次数。
* **失败原因**：健康检查失败的具体原因统计。
{% endtab %}

{% tab title="缓存统计" %}
* **命中率**：缓存命中次数和未命中次数。
* **缓存大小**：缓存的当前大小和最大大小。
{% endtab %}

{% tab title="上游服务统计" %}
* **请求数**：每个上游服务的请求总数和错误请求数。
* **延迟**：请求到上游服务的延迟分布。
* **重试次数**：请求的重试次数统计。
{% endtab %}

{% tab title="流量管理统计" %}
* **流量控制**：限流次数、熔断次数等。
* **重定向**：请求重定向次数和失败次数。
{% endtab %}

{% tab title="错误统计" %}
* **客户端错误**：如4xx状态码的请求数。
* **服务端错误**：如5xx状态码的请求数。
* **网络错误**：如连接超时、连接失败等。
{% endtab %}

{% tab title="资源使用统计" %}
* **内存使用**：Envoy 的内存使用情况。
* **CPU 使用**：Envoy 的 CPU 使用情况。
{% endtab %}
{% endtabs %}

Envoy 提供的状态统计信息非常丰富且细致，用户可以通过这些数据监控系统的健康状态、优化性能、快速定位问题，并做出相应的调整以提升系统的稳定性和可靠性。这些统计数据可以通过 Envoy 自带的管理接口访问，或者集成到监控系统（如 Prometheus、Grafana）中进行可视化和分析。

## 数据类型

<details>

<summary>Envoy统计数据分类</summary>

* **下游数据**：
  * 这些数据与进入 Envoy 的连接有关。
  * 主要由侦听器、HTTP 连接管理器和 TCP 代理过滤器生成。
  * 示例：用户请求进入你的系统，Envoy 会记录这些请求的数量和其他相关信息。

<!---->

* **上游数据**：
  * 这些数据与离开 Envoy 的连接有关。
  * 主要由连接池、路由器过滤器和 TCP 代理过滤器生成。
  * 示例：当你的系统向其他服务发送请求时，Envoy 会记录这些请求的数量和其他相关信息。

<!---->

* **Envoy 服务器数据**：
  * 这些数据记录了 Envoy 服务器实例的工作细节。
  * 包括服务器运行时间、分配的内存量等。
  * 示例：Envoy 运行了多长时间，使用了多少内存等信息。

</details>

<details>

<summary>Envoy 数据类型</summary>

Envoy 统计数据主要有三种类型，所有数据都是无符号整数：

1. **计数器（Counter）**：
   * 这是累加型的数据，只会增加不会减少。
   * 示例：总请求数（total requests）。
   * 用途：在一段时间内查看某个值的变化情况。
2. **指标（Gauge）**：
   * 这是常规的指标数据，可以增加也可以减少。
   * 示例：当前活动请求数（current active requests）。
   * 用途：查看某个时刻的状态。
3. **柱状图（Histogram）**：
   * 这用于统计数据的分布情况。
   * 示例：向上游服务器请求的时间（upstream request time）。
   * 用途：了解请求时长的分布情况，比单纯的平均值更详细。

通过这些统计数据，用户可以更好地监控系统的性能、识别问题并进行优化。

</details>

