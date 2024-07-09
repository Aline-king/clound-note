# 统计数据获取方式



{% tabs %}
{% tab title="管理接口" %}
Envoy 提供了一个管理接口，可以通过 HTTP API 直接获取统计数据。

**示例：**

```sh
curl http://<envoy-admin-address>:<port>/stats
```

这个命令会返回所有的统计数据，包括下游、上游和服务器数据。
{% endtab %}

{% tab title="Prometheus 集成" %}
Envoy 可以与 Prometheus 集成，将统计数据导出到 Prometheus，然后使用 Grafana 等工具进行可视化和监控。

**配置步骤：**

**在 Envoy 配置中启用 Prometheus 统计导出**：

该配置文件定义了 Envoy 的管理接口、监听器和集群，同时配置了 Prometheus 作为统计数据的导出方式。通过这些配置，Envoy 可以接受来自 `0.0.0.0:10000` 的请求，并将请求路由到 `service_0` 集群，同时将统计数据导出到 Prometheus，用于监控和分析。

```
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: service_0 }
          http_filters:
          - name: envoy.filters.http.router
  clusters:
  - name: service_0
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_0
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service_0
                port_value: 80

stats_sinks:
- name: envoy.stat_sinks.prometheus
  typed_config:
    "@type": type.googleapis.com/envoy.config.metrics.v3.PrometheusSink
    emit_tags_as_labels: true

```

**在 Prometheus 配置中添加 Envoy 的抓取配置**：

```yaml
scrape_configs:
  - job_name: 'envoy'
    static_configs:
      - targets: ['<envoy-admin-address>:9901']
        labels:
          group: 'envoy'
```
{% endtab %}

{% tab title="使用第三方工具" %}
许多第三方监控和分析工具可以集成 Envoy，例如 Datadog、New Relic 等。这些工具通常会提供现成的插件或集成方式，可以自动收集和可视化 Envoy 的统计数据。

**示例：**

* **Datadog**：通过 Datadog Agent 配置 Envoy 集成。
* **New Relic**：通过 New Relic APM 配置 Envoy 监控。
{% endtab %}

{% tab title="使用 Envoy 的本地日志" %}
Envoy 还可以将统计数据记录到本地日志文件中，然后通过日志分析工具进行处理和分析。

在管理接口配置中指定 `access_log_path`，该路径用于记录管理接口的访问日志。

```
admin:
  access_log_path: /tmp/admin_access.log # 管理接口的访问日志将记录到 `/tmp/admin_access.log` 文件中。
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: service_0 }
          http_filters:
          - name: envoy.filters.http.router
          access_log:
          - name: envoy.access_loggers.file
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
              path: /var/log/envoy/access.log # 访问日志将记录到 `/var/log/envoy/access.log` 文件中。
```


{% endtab %}

{% tab title="实时查看管理接口" %}
通过访问管理接口的 `/stats` 路径，可以在浏览器中实时查看统计数据：

```sh
http://<envoy-admin-address>:<port>/stats
```

获取 Envoy 的统计数据有多种方法，包括直接通过管理接口获取、与 Prometheus 集成、使用第三方工具和本地日志记录等。根据具体需求选择合适的方式，可以帮助用户有效地监控和优化系统性能。
{% endtab %}
{% endtabs %}

Envoy 本身不直接支持将统计数据记录到本地日志文件中。统计数据通常通过管理接口（例如 `http://<envoy-admin-address>:<port>/stats`）进行访问和收集。如果需要将统计数据记录到本地日志文件中，可以编写脚本定期抓取管理接口的统计数据并将其写入日志文件。

## 制作脚本

通过上述方法，可以将 Envoy 的统计数据定期记录到本地日志文件 `/var/log/envoy/stats.log` 中。之后，可以使用日志分析工具（如 Splunk、ELK Stack）进行处理和分析。

{% tabs %}
{% tab title="创建一个脚本来抓取统计数据" %}
```
#!/bin/bash
   curl -s http://localhost:9901/stats > /var/log/envoy/stats.log
```
{% endtab %}

{% tab title="保存脚本赋予权限" %}
```
chmod +x /usr/local/bin/fetch_envoy_stats.sh
```
{% endtab %}

{% tab title="设置定时任务" %}
每分钟抓取一次统计数据：

```
 * * * * * /usr/local/bin/fetch_envoy_stats.sh
```
{% endtab %}
{% endtabs %}
