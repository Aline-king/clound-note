# 常见stats sink类型



<details>

<summary><strong>Prometheus</strong></summary>

Prometheus 是一个开源的系统监控和报警工具，广泛用于容器化和微服务架构中。Envoy 可以通过 Prometheus Sink 导出统计数据，使 Prometheus 能够抓取并存储这些数据。

```
stats_sinks:
- name: envoy.stat_sinks.prometheus
  typed_config:
    "@type": type.googleapis.com/envoy.config.metrics.v3.PrometheusSink
    emit_tags_as_labels: true
```

Envoy 代理 收集指标 并导出和存储到 Prometheus 使用 Grafana进行可视化仪表盘，最后使用仪表盘进行展示

<img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

**数据流：**

1. **Envoy 代理收集指标并发送到 Prometheus Sink。**
2. **Prometheus Sink 将指标发送到 Prometheus 服务器。**
3. **Prometheus 服务器处理并存储这些指标。**
4. **Grafana 从 Prometheus 服务器获取数据，用于创建可视化仪表盘和设置警报。**
5. **用户通过 Grafana 监控和分析指标。**

**这个架构确保 Envoy 的指标被有效地收集、处理和可视化，提供全面的监控和警报功能。**



Envoy 的 statsd sink 不支持带标签的指标，而 Prometheus 需要带标签的指标来存储时序数据。因此，若要使用 Prometheus 收集数据，需要一个支持标签化指标的存储系统。

StatsD 是一种轻量级的统计数据收集工具，但它的设计比较简单，不支持带标签的指标。这意味着无法为指标附加额外的信息（如来源、类型等），这在复杂系统的监控中可能会有局限性。

</details>

<details>

<summary>StatsD</summary>

StatsD 是一个简单的、基于 UDP 协议的统计数据收集和聚合服务。Envoy 可以通过 StatsD Sink 将统计数据发送到 StatsD 服务器。

```
stats_sinks:
 - name: envoy.stat_sinks.statsd
   typed_config:
     "@type": type.googleapis.com/envoy.config.metrics.v3.StatsdSink
     address:
       socket_address:
         address: 127.0.0.1
         port_value: 8125
```

![](<../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png>)



**数据流：**

1. **Envoy 代理**：
   * 从传入和传出的流量中收集指标。
   * 将指标发送到 StatsD sink。
2. **StatsD Sink**：
   * 将指标转发到 StatsD 服务器。
3. **StatsD 服务器**：
   * 从 StatsD sink 接收指标。
   * 处理和聚合数据。
4. **监控和可视化工具（Grafana）**：
   * 从 StatsD 服务器获取聚合数据。
   * 在可定制的仪表盘上显示数据。
   * 根据指标设置警报。
5. **用户**：
   * 监控仪表盘并响应警报。

</details>



<details>

<summary><strong>DogStatsD</strong>：</summary>

```
stats_sinks:
- name: envoy.stat_sinks.dog_statsd
  typed_config:
    "@type": type.googleapis.com/envoy.config.metrics.v3.DogStatsdSink
    address:
      socket_address:
        address: 127.0.0.1
        port_value: 8125
```

Stats Sink 是 Envoy 用于导出统计数据的关键组件，主要作用是将 Envoy 内部的性能和运行状况数据发送到外部监控系统，以便进行实时监控、历史分析、可视化和报警。通过配置合适的 Stats Sink，用户可以更好地了解和管理 Envoy 代理及其所处理的流量。

</details>
