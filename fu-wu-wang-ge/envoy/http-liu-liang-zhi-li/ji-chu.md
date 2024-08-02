# 基础

在Envoy中，虚拟主机（Virtual Host）和路由（Route）是配置HTTP请求路由的重要概念。

## Envoy HTTP路由配置框架

虚拟主机是Envoy中用于根据请求的主机头（Host Header）来匹配请求的一种机制。它允许你在同一个Envoy实例中处理多个域名或服务。

在Envoy中，HTTP路由是通过路由配置来实现的，这是Envoy的核心功能之一。Envoy作为一个边缘和服务代理，提供了丰富的路由功能，能够管理微服务架构中的流量。下面是Envoy中HTTP路由及配置框架的一些关键组件和概念：

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<table data-header-hidden><thead><tr><th width="164"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td><strong>路由配置 (Route Configuration)</strong></td><td>Envoy的路由配置通常在其配置文件中定义，主要包括虚拟主机（virtual hosts）、路由条目（route entries）和路由表（route tables）。这些配置可以静态定义，也可以通过xDS API动态获取。</td><td></td></tr><tr><td><strong>虚拟主机 (Virtual Hosts)</strong></td><td>虚拟主机是一组路由规则的集合，它们基于请求的域名（例如 <code>Host</code>头部）来进行匹配。每个虚拟主机包含一个或多个路由规则，用于根据路径、请求头等信息将流量路由到不同的后端服务。</td><td></td></tr><tr><td><strong>路由条目 (Route Entries)</strong></td><td>每个路由条目定义了单个路由规则。它包含匹配条件和路由行为，如重定向、直接响应或将请求转发到指定的集群。路由条目可以根据请求路径、请求头、HTTP方法等条件进行匹配。</td><td></td></tr><tr><td><strong>集群 (Clusters)</strong></td><td><p></p><p>集群是Envoy用来管理后端服务的节点。每个集群包含一个或多个服务实例，Envoy将根据路由配置将流量分发到这些实例。集群支持多种发现和健康检查机制。</p></td><td></td></tr></tbody></table>



**示例配置**

这是一个简单的Envoy路由配置示例：

```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 80
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: backend
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: backend_service
  clusters:
  - name: backend_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    hosts:
    - socket_address:
        address: backend
        port_value: 80
```

在这个配置中，所有到达端口80的HTTP请求都将由 `envoy.http_connection_manager`处理，并根据定义的路由规则路由到后端服务。

Envoy的HTTP路由配置非常灵活且功能强大，能够应对各种复杂的路由需求，并支持高度的动态配置和扩展。

### 1.2 虚拟主机在Envoy路由中的作用和行为 <a href="#id-12-xu-ni-zhu-ji-zai-envoy-lu-you-zhong-de-zuo-yong-he-xing-wei-19" id="id-12-xu-ni-zhu-ji-zai-envoy-lu-you-zhong-de-zuo-yong-he-xing-wei-19"></a>

在Envoy的HTTP路由配置中，顶级元素通常是虚拟主机（Virtual Host）。以下描述了虚拟主机在Envoy路由中的作用和行为：

**虚拟主机的定义和作用**

* **逻辑名称与域名集合**：每个虚拟主机在配置中有一个唯一的逻辑名称（`name`），这是为了在配置文件中标识不同的虚拟主机。此外，虚拟主机还定义了一组域名（`domains`），这些域名用于匹配进入Envoy的请求中的 `Host`头部。这意味着，当一个HTTP请求到达Envoy时，Envoy会查看请求的 `Host`头部，并将其与虚拟主机配置的域名进行比较，以决定该请求应该由哪个虚拟主机处理。

**路由决策过程**

* **域名匹配后的路由处理**：一旦基于域名匹配到一个虚拟主机，接下来的处理就是根据该虚拟主机下配置的路由规则（`routes`）来决定如何处理请求。这些路由规则可以定义多种行为，包括：
  * **路由到后端集群**：最常见的行为是将请求路由到一个后端服务或集群。这通常涉及到更细粒度的匹配条件，如请求的路径（`prefix`、`path`或使用正则表达式等），请求方法等。
  * **重定向**：路由规则还可以配置为对某些请求执行HTTP重定向，例如从HTTP自动重定向到HTTPS，或者从一个旧的URL重定向到新的URL。

这种基于虚拟主机的路由机制使得Envoy能够非常灵活地处理来自不同域的请求，并且可以在同一个Envoy实例上配置多个虚拟主机，每个主机都有自己独立的路由策略。这对于运行多个服务或应用在同一物理服务器上的情况非常有用，也支持微服务架构中服务的隔离和独立路由决策。

这是一个结构化的Envoy配置示例，包含监听器、过滤器链、HTTP连接管理器以及路由配置。

```yaml
listeners:
  - name: <listener_name>  # 监听器的名称
    address: {...}  # 监听器的地址配置
    filter_chains:
      - filters:
          - name: envoy.filters.network.http_connection_manager  # 使用的过滤器名称
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
              stat_prefix: ingress_http  # 统计数据前缀
              codec_type: AUTO  # 编解码器类型
              route_config:
                name: <route_config_name>  # 路由配置的名称
                virtual_hosts:
                  - name: <virtual_host_name>  # 虚拟主机的名称
                    domains: []  # 与此虚拟主机相关联的域名列表
                    routes:
                      - name: <route_name>  # 路由条目的名称
                        match: 
                          {...}  # 路由匹配条件，可以基于路径前缀、完整路径、正则表达式或连接匹配器
                        route:
                          {...}  # 路由目标，可以是集群、基于请求头的集群或加权集群
                        redirect:
                          {...}  # 重定向配置
                        direct_response:
                          {...}  # 直接响应配置
                        virtual_clusters: []  # 虚拟集群列表，用于统计信息
```

**关键元素说明：**

* **listeners**: 这是配置文件的顶级元素，其中包括所有监听器的定义。每个监听器负责在特定的网络地址上监听入站连接。
* **filter\_chains**: 每个监听器可以有多个过滤器链，用于处理通过该监听器的数据流。
* **filters**: 指定使用的过滤器类型，这里使用的是 `envoy.filters.network.http_connection_manager`，专门处理HTTP连接。
* **typed\_config**: 具体到HTTP连接管理器的配置，包括路由配置。
* **route\_config**: 定义了如何处理通过HTTP连接管理器的请求，包括一个或多个虚拟主机。
* **virtual\_hosts**: 每个虚拟主机根据请求的 `Host`头部匹配特定的域名，并定义一系列的路由规则。
* **routes**: 每个路由定义了匹配条件和处理动作，包括直接路由到后端集群、重定向或直接响应。

这种配置方式为Envoy提供了高度的灵活性和强大的流量管理能力，使其成为处理复杂网络环境中的请求的理想选择。

**实际应用配置示例**

```powershell
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
              - "www.example.com"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: service_cluster
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
        
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

在上述配置中，虚拟主机 `local_service` 用于匹配请求主机头为 `example.com` 或 `www.example.com` 的请求。







