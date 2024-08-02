# Envoy HTTP高级路由

## 高级路由 <a href="#liu-envoyhttp-gao-ji-lu-you-182" id="liu-envoyhttp-gao-ji-lu-you-182"></a>

Envoy的HTTP高级路由功能提供了强大的工具和配置选项，以便灵活地管理和处理HTTP请求。以下是Envoy的HTTP高级路由的主要内容和功能：

{% tabs %}
{% tab title="路由匹配" %}
### （Route Matching） <a href="#id-61-lu-you-pi-pei-routematching184" id="id-61-lu-you-pi-pei-routematching184"></a>

路由匹配是Envoy路由的核心部分，通过配置不同的匹配条件，将请求路由到合适的上游服务。匹配条件包括：

* **前缀匹配（prefix match）**：匹配请求路径的前缀。
* **完全路径匹配（path match）**：匹配完整的请求路径。
* **正则表达式匹配（regex match）**：使用正则表达式匹配请求路径。
* **请求头匹配（header match）**：基于请求头的值进行匹配。
* **查询参数匹配（query parameter match）**：基于查询参数的值进行匹配。
{% endtab %}

{% tab title="路由目标" %}
定义匹配请求后的目标服务和行为，包括：

* **集群（cluster）**：将请求转发到特定的上游集群。
* **集群权重（weighted clusters）**：将请求按权重分配到多个集群，实现流量分配和负载均衡。
* **直连响应（direct response）**：直接由Envoy响应请求，而不是转发到上游服务。
* **重定向（redirect）**：将请求重定向到另一个URL。
{% endtab %}

{% tab title="路由策略" %}
通过路由策略（Route Policies）对请求进行进一步的处理和控制：

* **请求/响应修改**：修改请求和响应的头信息、路径等。
* **重试策略（retry policy）**：定义请求失败后的重试行为，包括重试次数、条件等。
* **超时策略（timeout policy）**：定义请求的超时时间，包括总超时、空闲超时等。
* **流量镜像（traffic mirroring）**：将请求复制并发送到其他服务进行调试和测试。
{% endtab %}

{% tab title="路由级别的特性和操作" %}
* **健康检查（health checks）**：确保请求仅被路由到健康的上游服务实例。
* **负载均衡策略（load balancing policy）**：定义请求在上游集群中的负载均衡行为，例如轮询、最少连接、随机等。
* **请求限速（rate limiting）**：对请求进行限速控制，防止过载。
* **服务质量（QoS）**：配置不同的优先级和延迟管理策略。
{% endtab %}

{% tab title="高级匹配条件" %}
* **复合匹配（composite matching）**：组合多种匹配条件进行复杂匹配，例如同时基于路径、请求头和查询参数进行匹配。
* **谓词匹配（predicate matching）**：使用自定义条件进行匹配，例如基于请求的某些特征进行高级匹配。
{% endtab %}

{% tab title="动态路由" %}
* **xDS API**：通过xDS API动态更新路由配置，包括CDS（Cluster Discovery Service）、EDS（Endpoint Discovery Service）、LDS（Listener Discovery Service）和RDS（Route Discovery Service）。
* **实时配置更新**：支持路由配置的实时更新，确保系统的高灵活性和可用性。
{% endtab %}
{% endtabs %}

6.8 扩展内容

**域名映射到虚拟主机**

* **域名映射到虚拟主机**：Envoy使用虚拟主机（Virtual Host）来根据域名匹配请求。每个虚拟主机可以管理一个或多个域名，并包含相应的路由规则。

**路由匹配**

* **路径前缀匹配（prefix match）**：根据路径前缀匹配请求。
* **精确匹配（exact match）**：根据完整路径精确匹配请求。
* **正则表达式匹配（regex match）**：使用正则表达式匹配路径。

**重定向**

* **虚拟主机级别的TLS重定向**：在虚拟主机级别进行TLS重定向，将HTTP请求重定向到HTTPS。
* **路径级别的重定向（path/host redirect）**：根据路径或主机头进行重定向，将请求重定向到另一个路径或主机。

**直接生成响应**

* **直接响应**：Envoy可以直接生成HTTP响应，而无需将请求转发到上游服务。

**主机重写和前缀重写**

* **主机重写（host rewrite）**：将请求的主机头重写为另一个主机名，例如将hostA重写为hostB。
* **前缀重写（prefix rewrite）**：将请求路径的前缀重写为另一个前缀。

**请求重试和超时**

* **请求重试（retry）**：基于HTTP头或路由配置进行请求重试，可以配置重试条件、重试次数和重试策略。
* **请求超时（timeout）**：配置请求的超时时间，包括总超时和每次尝试的超时。

**流量迁移和权重路由**

* **流量迁移**：基于运行时参数进行流量迁移，动态调整流量的路由目标。
* **权重或百分比流量分割**：基于权重或百分比将流量分割到多个上游集群，实现流量分配和负载均衡。

**基于标头和优先级的路由**

* **基于标头的路由**：根据HTTP请求头的值进行路由匹配。
* **基于优先级的路由**：配置请求的优先级，基于优先级进行路由选择。

**基于哈希的路由**

* **哈希策略（hash-based routing）**：基于请求的某些特征（例如请求头、URL参数）进行哈希计算，确保同一特征的请求路由到相同的上游服务。

**配置示例及解析**

下面我们来看一个Envoy高级路由配置的示例，展示上述功能的实现：

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
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains:
              - "example.com"
              routes:
              - match:
                  prefix: "/service1"
                route:
                  cluster: service1_cluster
                  host_rewrite_literal: new-service1.example.com
                  timeout: 5s
                  retry_policy:
                    retry_on: "5xx"
                    num_retries: 3
                    per_try_timeout: 1s
              - match:
                  prefix: "/service2"
                route:
                  weighted_clusters:
                    clusters:
                    - name: service2_cluster
                      weight: 80
                    - name: service2_canary_cluster
                      weight: 20
              - match:
                  prefix: "/health"
                direct_response:
                  status: 200
                  body:
                    inline_string: "OK"
              - match:
                  prefix: "/old-path"
                redirect:
                  path_redirect: "/new-path"
                  response_code: 301
              - match:
                  prefix: "/test"
                route:
                  cluster: test_cluster
                  prefix_rewrite: "/new-test"
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
          
  clusters:
  - name: service1_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service1_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 8081
            
  - name: service2_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service2_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.2
                port_value: 8082
            
  - name: service2_canary_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service2_canary_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.3
                port_value: 8083
            
  - name: test_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: test_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.4
                port_value: 8084
```
