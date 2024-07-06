# 侦听器工作原理

## 侦听器部署说明及配置详情

* Envoy 配置支持在单个进程中运行任意数量的侦听器
  * 独立部署时，建议每个主机仅部署单个Envoy实例，并在必要时于此实例上运行一到多个侦听器；
  * Enovy支持TCP和UDP两种类型的侦听器；
* 每个侦听器都独立配置有一定数量的网络层级（L3/L4）过滤器
  * 侦听器收到的连接请求将由其过滤器链中的各过滤器进行处理；
  * 通用侦听器架构用于执行Envoy所用于的绝大多数不同的代理任务
    * 速率限制
    * TLS客户端认证
    * HTTP连接管理
    * 原始TCP代理

以下配置

{% tabs %}
{% tab title="定义了一个Envoy侦听器配置详情" %}
```
{
"name": "...",
"address": "{...}",
"stat_prefix": "...",
"filter_chains": [],
"use_original_dst": "{...}",
"default_filter_chain": "{...}",
"per_connection_buffer_limit_bytes": "{...}",
"metadata": "{...}",
"drain_type": "...",
"listener_filters": [],
"listener_filters_timeout": "{...}",
"continue_on_listener_filters_timeout": "...",
"transparent": "{...}",
"freebind": "{...}",
"socket_options": [],
"tcp_fast_open_queue_length": "{...}",
"traffic_direction": "...",
"udp_listener_config": "{...}",
"api_listener": "{...}",
"connection_balance_config": "{...}",
"reuse_port": "...",
"access_log": [],
"tcp_backlog_size": "{...}",
"bind_to_port": "{...}"
}

```
{% endtab %}

{% tab title="解释" %}
```
- `name`: 监听器的名称，用于标识。
- `address`: 监听地址配置，包括IP地址和端口号。
- `stat_prefix`: 统计前缀，用于生成监听器相关的统计指标。
- `filter_chains`: 空列表表示未配置，通常包含一系列网络过滤器，用于处理不同类型的连接和请求。
- `use_original_dst`: 是否使用连接最初的目标地址作为监听目标。
- `default_filter_chain`: 默认的过滤器链，当没有其他匹配的过滤器链时使用。
- `per_connection_buffer_limit_bytes`: 每个连接的缓冲区大小限制。
- `metadata`: 附加的元数据，可用于传递给过滤器或其他配置。
- `drain_type`: 优雅关闭的类型，指示如何处理已建立的连接。
- `listener_filters`: 监听器级别的过滤器列表，这些过滤器在连接被任何filter_chain处理前执行。
- `listener_filters_timeout`: 监听器过滤器的超时时间。
- `continue_on_listener_filters_timeout`: 是否在监听器过滤器超时时继续处理连接。
- `transparent`: 是否启用透明代理模式。
- `freebind`: 允许监听socket绑定到未分配的IP地址。
- `socket_options`: Socket选项的列表，用于定制化网络行为。
- `tcp_fast_open_queue_length`: TCP Fast Open队列的长度。
- `traffic_direction`: 流量方向，例如INBOUND或OUTBOUND，影响统计和过滤逻辑。
- `udp_listener_config`: UDP监听器的特定配置。
- `api_listener`: 与xDS API交互的配置，动态更新监听器。
- `connection_balance_config`: 连接平衡策略的配置。
- `reuse_port`: 是否允许端口复用。
- `access_log`: 访问日志的配置列表。
- `tcp_backlog_size`: TCP监听的 backlog（待处理连接队列）大小。
- `bind_to_port`: 是否实际绑定到端口，对于某些场景可能需要设置为false以避免立即绑定。

此配置展示了Envoy监听器高度可配置性和灵活性，支持多种网络协议、过滤逻辑和管理功能。

```
{% endtab %}
{% endtabs %}
