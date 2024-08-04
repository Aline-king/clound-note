# stats相关配置

Envoy 的 stats 配置参数位于 Bootstrap 配置文件的顶级配置段。



```
{
  ...
  "stats_sinks": [], // 用于配置连接到 statsD
  "stats_config": "{...}", // stats 内部处理机制；
  "stats_flush_interval": "{...}", // stats 数据刷写至 sinks 的频率，出于性能考虑，Envoy 仅周期性刷写 counters 和 gauges，默认时长为 5000ms；
  "stats_flush_on_admin": "..." // 仅在 admin 接口上收到查询请求时才刷写数据；
  ...
}
```

`stats_sinks`：可选配置。统计数据默认没有配置任何暴露机制，但如果需要存储长期的指标数据，则应手动定制此配置。

**stats\_sinks 配置示例**

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

