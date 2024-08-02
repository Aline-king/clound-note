# Page 3

Envoy的HTTP连接管理（HTTP Connection Management）是Envoy代理的重要组成部分，负责管理HTTP连接的生命周期，包括HTTP请求的接收、处理和响应发送。以下是HTTP连接管理的主要内容：

### 5.1 HTTP连接管理器（HTTP Connection Manager）作用 <a href="#id-51http-lian-jie-guan-li-qi-httpconnectionmanager-zuo-yong-139" id="id-51http-lian-jie-guan-li-qi-httpconnectionmanager-zuo-yong-139"></a>

**HTTP连接管理器**是Envoy中处理HTTP流量的核心组件。它提供了全面的功能来管理HTTP连接和流量。

{% tabs %}
{% tab title="配置结构" %}
HTTP连接管理器的配置通常包括以下几个部分：

* **stat\_prefix**: 统计前缀，用于生成统计数据的前缀。
* **codec\_type**: 编解码类型，支持HTTP1、HTTP2和自动检测（AUTO）。
* **route\_config**: 路由配置，定义如何将请求路由到上游服务。
* **http\_filters**: HTTP过滤器，定义在请求和响应过程中应用的过滤器链。
* **access\_log**: 访问日志配置，定义如何记录访问日志。
{% endtab %}

{% tab title="路由配置" %}
**路由配置**（Route Configuration）定义了如何将HTTP请求路由到适当的上游服务。它包括虚拟主机和路由规则：

* **virtual\_hosts**: 虚拟主机，包含域名和对应的路由规则。
* **routes**: 路由规则，定义如何匹配请求并将其转发到后端集群。
{% endtab %}

{% tab title="HTTP过滤器" %}
HTTP过滤器（HTTP Filters）是在请求和响应过程中应用的插件。它们可以执行多种功能，如：

* **认证和授权**: 验证请求的合法性并检查访问权限。
* **修改请求和响应**: 修改HTTP头、请求体或响应体。
* **统计和监控**: 收集和报告请求处理的统计数据。

常见的HTTP过滤器包括：

* **envoy.filters.http.router**: 路由过滤器，负责将请求转发到上游服务。
* **envoy.filters.http.jwt\_authn**: JWT认证过滤器，用于验证JWT令牌。
* **envoy.filters.http.rbac**: 角色访问控制过滤器，用于实施访问控制策略。
{% endtab %}

{% tab title="访问日志" %}
访问日志（Access Logging）记录每个请求的详细信息，用于监控和审计。配置项包括：

* **path**: 日志文件路径。
* **format**: 日志格式，可以是预定义的格式或自定义格式。
* **filters**: 日志过滤器，控制哪些请求被记录。
{% endtab %}

{% tab title="连接管理和超时设置" %}
Envoy提供多种选项来管理HTTP连接和超时：

* **idle\_timeout**: 空闲连接超时时间，超过该时间未活动的连接将被关闭。
* **request\_timeout**: 请求超时时间，单个请求的处理时间超过该值将被终止。
* **drain\_timeout**: 连接排空超时时间，用于优雅关闭连接。
{% endtab %}
{% endtabs %}

### 5.7 示例配置 <a href="#id-57-shi-li-pei-zhi-158" id="id-57-shi-li-pei-zhi-158"></a>

以下是一个示例配置，展示了HTTP连接管理器的基本配置：

```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 8080
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
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: service_cluster
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
        
          access_log:
          - name: envoy.access_loggers.file
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
              path: "/var/log/envoy/access.log"
              format: "[%START_TIME%] \"%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%\" %RESPONSE_CODE% %RESPONSE_FLAGS% \"%REQ(USER-AGENT)%\" \"%REQ(X-FORWARDED-FOR)%\" \"%REQ(REQUEST_ID)%\" \"%REQ(AUTHORITY)%\" \"%UPSTREAM_HOST%\"\n"
        
  clusters:
  - name: service_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 8081
```

Envoy的HTTP连接管理器提供了强大的功能来管理HTTP连接和请求处理。它支持灵活的路由配置、丰富的HTTP过滤器、多种超时设置和详细的访问日志记录。这些功能使Envoy能够高效地处理复杂的HTTP流量管理需求，并提供高性能和可扩展的解决方案。

### 5.8 HTTP连接管理的扩展功能 <a href="#id-58http-lian-jie-guan-li-de-kuo-zhan-gong-neng-162" id="id-58http-lian-jie-guan-li-de-kuo-zhan-gong-neng-162"></a>

<table><thead><tr><th width="144"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>HTTP连接管理器的基本功能</td><td><p><strong>字节到消息转换</strong>：通过内置的L4过滤器，HTTP连接管理器将原始字节流转换为HTTP应用层协议级别的消息和事件，例如接收到的标头和主体。</p><p><strong>HTTP连接和请求处理</strong>：处理所有HTTP连接和请求的通用功能，包括访问日志、生成跟踪连接ID、请求/响应头处理、路由表管理和统计信息收集</p></td><td></td></tr><tr><td>支持的协议</td><td><strong>HTTP/1.1、WebSockets和HTTP/2</strong>：Envoy支持这些协议的处理，但不支持SPDY。</td><td></td></tr><tr><td>路由表管理</td><td><strong>静态和动态路由表</strong>：路由表可以通过静态配置或者通过xDS API中的RDS（Route Discovery Service）动态生成。</td><td></td></tr><tr><td>重试插件</td><td><strong>内建重试插件</strong>：可以配置重试行为，包括Host Predicates和Priority Predicates，用于定义重试的条件和优先级。</td><td></td></tr><tr><td>302重定向支持</td><td><strong>内建302重定向支持</strong>：Envoy可以捕获302重定向响应，合成新请求后将其发送到路由匹配指定的上游端，并将收到的响应返回给原始客户端。</td><td></td></tr><tr><td>超时机制</td><td><p><strong>连接级别超时</strong>：</p><ul><li><strong>空闲超时</strong>：处理连接空闲时间超过设定值时关闭连接。</li><li><strong>排空超时（GOAWAY）</strong>：在连接关闭前，发送GOAWAY帧通知客户端不再接受新请求。</li></ul><p><strong>流级别超时</strong>：</p><ul><li><strong>空闲超时</strong>：处理HTTP流空闲时间超过设定值时关闭流。</li><li><strong>每路由相关的上游端点超时</strong>：针对特定路由配置的超时时间。</li><li><strong>gRPC最大超时</strong>：处理gRPC请求的最大超时时间。</li></ul></td><td></td></tr><tr><td>动态转发代理</td><td><strong>基于自定义集群的动态转发代理</strong>：Envoy支持基于自定义集群的动态转发，这允许Envoy根据实时情况动态调整请求的转发目标。</td><td></td></tr><tr><td>HTTP协议相关功能通过HTTP过滤器实现</td><td><p>HTTP协议相关的功能通过各类HTTP过滤器实现，这些过滤器大体可分为编码器和解码器两类：</p><p><strong>常用过滤器</strong></p><ul><li><p><strong>Router过滤器（envoy.router）</strong>：</p><ul><li><strong>请求转发和重定向</strong>：基于路由表完成请求的转发或重定向。</li><li><strong>重试操作</strong>：处理重试逻辑，确保请求在失败后按照配置进行重试。</li><li><strong>统计信息生成</strong>：收集和生成有关请求处理的统计信息。</li></ul></li></ul></td><td></td></tr></tbody></table>

5.8.8&#x20;



Envoy的HTTP连接管理器不仅负责基础的HTTP连接和请求处理，还包含丰富的功能来增强HTTP协议的支持和管理能力。这些功能包括多协议支持、静态和动态路由配置、内建重试和重定向支持、详细的超时管理以及动态转发代理能力。通过这些功能，Envoy能够灵活、高效地管理和处理复杂的HTTP流量，使其成为一个强大的边缘代理和服务网格组件。
