# 主动健康状态检测

## 分布式系统中的一致性模型 <a href="#yi-fen-bu-shi-xi-tong-zhong-de-yi-zhi-xing-mo-xing-1" id="yi-fen-bu-shi-xi-tong-zhong-de-yi-zhi-xing-mo-xing-1"></a>

在分布式系统中，一致性模型定义了多个副本之间的数据同步程度。主要有两种模型：

1. **强一致性**：所有副本在任何时间点上都保证一致的状态。
2. **最终一致性**：允许短期的不一致，但在没有新的更新操作后，系统最终会达到一致的状态。

## Envoy的服务发现和健康检查 <a href="#er-envoy-de-fu-wu-fa-xian-he-jian-kang-jian-cha-4" id="er-envoy-de-fu-wu-fa-xian-he-jian-kang-jian-cha-4"></a>

Envoy的服务发现机制是基于最终一致性模型的。这意味着在某一时刻，Envoy实例可能会对上游服务的成员列表有不同的看法，但随着时间推移，这些视图会趋于一致。

**服务发现的最终一致性**

Envoy通过xDS（如CDS、EDS）协议与控制平面（如Istio、Consul）通信来获取服务的最新信息。在实际操作中：

1. **订阅和推送**：Envoy实例订阅控制平面发布的服务信息更新。当服务实例加入或离开网格时，控制平面会将这些变化推送给订阅的Envoy实例。
2. **传播延迟**：由于网络延迟、处理时间等因素，不同Envoy实例接收到更新的时间点可能不同。因此，在短期内，Envoy实例之间的服务信息可能不一致。
3. **最终一致性**：随着控制平面不断推送更新，并且所有Envoy实例定期刷新服务信息，所有实例最终会达到一致的视图。

**主动健康检查**

为了确保服务的可靠性，Envoy结合了主动健康检查机制来判定集群的健康状态：

1. **健康检查类型**：Envoy支持多种健康检查方式，包括HTTP、TCP和GRPC健康检查。
2. **周期性检查**：Envoy定期向上游服务实例发送健康检查请求，以确定它们是否健康。
3. **状态报告**：根据健康检查的结果，Envoy可以动态调整负载均衡策略，例如只将流量发送到健康的实例，避免不健康的实例。

**具体场景理解**

假设在一个服务网格中，有多个Envoy实例和一个控制平面。控制平面管理着一个名为 `example_service`的服务，该服务有多个实例。

1. **实例加入网格**：当新的服务实例 `example_service_3`加入网格时，控制平面会更新其服务信息并推送给所有订阅的Envoy实例。
2. **传播和更新**：由于传播延迟，不同Envoy实例接收到该更新的时间点不同。一段时间内，某些Envoy实例可能还不知道 `example_service_3`的存在。
3. **最终一致**：随着时间推移，所有Envoy实例都会收到该更新并更新其内部状态，最终达到一致。
4. **健康检查**：在这一过程中，Envoy实例会持续对 `example_service`的所有实例进行健康检查。如果某个实例（如 `example_service_2`）变得不健康，Envoy会将其标记为不健康，并停止将流量路由到该实例，直到其恢复健康状态。

Envoy的服务发现采用最终一致性模型，而不是强一致性模型。这意味着它允许短期的不一致，但最终会达到一致的状态。通过结合主动健康检查机制，Envoy能够确保尽可能多地将流量路由到健康的上游服务实例，从而提高整个系统的可靠性和稳定性。这种设计既保证了系统的灵活性和扩展性，又通过健康检查机制维护了服务的可用性。

## 三、主动健康检测类型及示例 <a href="#san-zhu-dong-jian-kang-jian-ce-lei-xing-ji-shi-li-16" id="san-zhu-dong-jian-kang-jian-ce-lei-xing-ji-shi-li-16"></a>

在Envoy中，主动健康检查（Active Health Checking）是一种机制，用于定期向上游服务实例发送健康检查请求，以确定它们是否可以正常处理请求。通过主动健康检查，Envoy可以确保仅将流量路由到健康的实例，从而提高服务的可靠性和可用性。

### 3.1 健康检查类型 <a href="#id-31-jian-kang-jian-cha-lei-xing-18" id="id-31-jian-kang-jian-cha-lei-xing-18"></a>

Envoy支持多种健康检查类型，包括：

1. **HTTP/HTTPS 健康检查**：发送HTTP/HTTPS请求并检查响应状态码。
2. **TCP 健康检查**：通过TCP连接建立是否成功来判断健康状态。
3. **gRPC 健康检查**：发送gRPC请求并检查响应状态。

### 3.2 健康检查配置示例 <a href="#id-32-jian-kang-jian-cha-pei-zhi-shi-li-21" id="id-32-jian-kang-jian-cha-pei-zhi-shi-li-21"></a>

以下是如何在Envoy中配置主动健康检查的示例，包括HTTP、TCP和gRPC健康检查。



{% tabs %}
{% tab title="HTTP 健康检查" %}
```yaml
static_resources:
  clusters:
  - name: http_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: http_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: http_service.example.com
                port_value: 80
    health_checks:
    - timeout: 1s
      interval: 10s
      unhealthy_threshold: 3
      healthy_threshold: 2
      http_health_check:
        path: /health
        expected_statuses:
          - start: 200
            end: 200
```

在这个示例中，Envoy会每10秒向 `http_service.example.com`发送一个HTTP请求，请求路径为 `/health`。

如果连续2次返回200状态码，Envoy会将该实例标记为健康；

如果连续3次没有返回200状态码，则标记为不健康。
{% endtab %}

{% tab title="TCP 健康检查" %}
```yaml
static_resources:
  clusters:
  - name: tcp_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: tcp_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: tcp_service.example.com
                port_value: 9000
    health_checks:
    - timeout: 1s
      interval: 10s
      unhealthy_threshold: 3
      healthy_threshold: 2
      tcp_health_check: {}
```

在这个示例中，Envoy会每10秒尝试与 `tcp_service.example.com`的9000端口建立TCP连接。

如果连续2次连接成功，则标记为健康；

如果连续3次连接失败，则标记为不健康。
{% endtab %}

{% tab title="gRPC 健康检查" %}
```yaml
static_resources:
  clusters:
  - name: grpc_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: grpc_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: grpc_service.example.com
                port_value: 50051
    health_checks:
    - timeout: 1s
      interval: 10s
      unhealthy_threshold: 3
      healthy_threshold: 2
      grpc_health_check:
        service_name: "my_service"
```

在这个示例中，Envoy会每10秒向 `grpc_service.example.com`的50051端口发送一个gRPC健康检查请求，服务名为 `my_service`。

如果连续2次检查成功，则标记为健康；

如果连续3次检查失败，则标记为不健康。
{% endtab %}
{% endtabs %}

**关键配置项解释**

* **timeout**: 每次健康检查的超时时间。
* **interval**: 健康检查的时间间隔。
* **unhealthy\_threshold**: 连续检查失败次数超过该值时，实例被标记为不健康。
* **healthy\_threshold**: 连续检查成功次数超过该值时，实例被标记为健康。
* **http\_health\_check**: HTTP健康检查的具体配置，包括检查路径和期望的状态码范围。
* **tcp\_health\_check**: TCP健康检查的配置，通常为空。
* **grpc\_health\_check**: gRPC健康检查的具体配置，包括服务名。

#### 3.2.4 监控和调试 <a href="#id-324-jian-kong-he-tiao-shi-34" id="id-324-jian-kong-he-tiao-shi-34"></a>

Envoy提供了丰富的监控和调试工具，可以通过/admin接口查看健康检查的状态和结果。例如，访问 `http://localhost:9901/stats`可以查看健康检查的统计信息。

通过主动健康检查，Envoy可以动态监控上游服务实例的健康状态，并根据检查结果调整流量路由。这种机制有助于提高服务的可靠性，确保只有健康的实例接收请求，避免因实例故障导致的服务不可用。

## 四、 主动健康检测案例 <a href="#si-zhu-dong-jian-kang-jian-ce-an-li-37" id="si-zhu-dong-jian-kang-jian-ce-an-li-37"></a>

### 4.1 基于http协议主动健康检测 <a href="#id-41-ji-yu-http-xie-yi-zhu-dong-jian-kang-jian-ce-38" id="id-41-ji-yu-http-xie-yi-zhu-dong-jian-kang-jian-ce-38"></a>

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719840287059/646aa033ea24491897ad481fc8dae5e4.png)

```bash
# mkdir envoy_cluster_health_checks
# cd envoy_cluster_health_checks
```

```yaml
# vim docker-compose.yaml
 version: '3.3'
 services:
   envoy:
     image: envoyproxy/envoy:v1.30.1
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
     volumes:
     - ./front-envoy.yaml:/etc/envoy/envoy.yaml
     networks:
       envoymesh:
         ipv4_address: 172.29.1.2
         aliases:
         - front-proxy
     depends_on:
     - webserver01-sidecar
     - webserver02-sidecar
   webserver01-sidecar:
     image: envoyproxy/envoy:v1.30.1
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
     volumes:
     - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
     hostname: blue
     networks:
       envoymesh:
         ipv4_address: 172.29.1.3
         aliases:
         - myservice
   webserver01:
     image: www.kubemsb.com/envoy/demoapp:v1.0
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
       - PORT=8080
       - HOST=127.0.0.1
     network_mode: "service:webserver01-sidecar"
     depends_on:
     - webserver01-sidecar
   webserver02-sidecar:
     image: envoyproxy/envoy:v1.30.1
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
     volumes:
     - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
     hostname: yellow
     networks:
       envoymesh:
         ipv4_address: 172.29.1.4
         aliases:
         - myservice
   webserver02:
     image: www.kubemsb.com/envoy/demoapp:v1.0
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
       - PORT=8080
       - HOST=127.0.0.1
     network_mode: "service:webserver02-sidecar"
     depends_on:
     - webserver02-sidecar
 networks:
   envoymesh:
     driver: bridge
     ipam:
       config:
         - subnet: 172.29.1.0/24
```

```
上述内容解释：

### 版本

version: '3.3'

这行指定了Docker Compose文件使用的版本，这里是3.3版，这对应于支持的Docker Compose功能。

### 服务
这个部分定义了所有的容器服务。

#### 1. envoy

envoy:
  image: envoyproxy/envoy:v1.30.1
  environment:
    - ENVOY_UID=0
    - ENVOY_GID=0
  volumes:
    - ./front-envoy.yaml:/etc/envoy/envoy.yaml
  networks:
    envoymesh:
      ipv4_address: 172.29.1.2
      aliases:
      - front-proxy
  depends_on:
    - webserver01-sidecar
    - webserver02-sidecar

- 使用`envoyproxy/envoy:v1.30.1`镜像。
- 环境变量`ENVOY_UID`和`ENVOY_GID`设置为0，通常代表root用户。
- 挂载本地的`front-envoy.yaml`配置文件到容器的`/etc/envoy/envoy.yaml`。
- 分配固定的IPv4地址`172.29.1.2`并在`envoymesh`网络中别名为`front-proxy`。
- 此服务依赖于`webserver01-sidecar`和`webserver02-sidecar`服务。

#### 2. webserver01-sidecar 和 webserver02-sidecar
这两个服务配置几乎相同，只是分配的IP和主机名不同。例如`webserver01-sidecar`的配置如下：

webserver01-sidecar:
  image: envoyproxy/envoy:v1.30.1
  environment:
    - ENVOY_UID=0
    - ENVOY_GID=0
  volumes:
    - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
  hostname: blue
  networks:
    envoymesh:
      ipv4_address: 172.29.1.3
      aliases:
      - myservice

- 使用相同的Envoy镜像。
- 同样挂载了Envoy的配置文件。
- 在`envoymesh`网络中配置了特定的IPv4地址和别名`myservice`。
- `hostname`设置为`blue`，为该容器提供了一个网络内部可以解析的名字。

#### 3. webserver01 和 webserver02
这两个web服务器容器配置也类似，它们使用相同的自定义镜像，并通过`network_mode`连接到各自的sidecar服务。例如`webserver01`的配置如下：

webserver01:
  image: www.kubemsb.com/envoy/demoapp:v1.0
  environment:
    - ENVOY_UID=0
    - ENVOY_GID=0
    - PORT=8080
    - HOST=127.0.0.1
  network_mode: "service:webserver01-sidecar"
  depends_on:
    - webserver01-sidecar

- 使用定制的`demoapp`镜像。
- 设置环境变量，包括应用端口和主机。
- 通过`network_mode`将网络配置设置为与`webserver01-sidecar`相同，这意味着它会共享网络栈。

### 网络

networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.29.1.0/24

定义了一个名为`envoymesh`的网络，使用桥接模式，并配置了一个特定的子网。这允许在同一网络下的容器可以直接通过IP地址互相访问。
```

```
# vim front-envoy.yaml
# cat front-envoy.yaml
admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
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
            - name: webservice
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: web_cluster_01 }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
    
  clusters:
  - name: web_cluster_01
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: web_cluster_01
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: myservice, port_value: 80 }
    health_checks:
    - timeout: 5s
      interval: 10s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /livez
        expected_statuses:
          start: 200
          end: 399
```

```
上述内容解释：
这个YAML内容是Envoy代理的配置文件的一部分，它定义了代理服务器的行为、路由规则以及如何处理进出流量。下面是对这些配置的详细解释：

### Admin 部分

admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

- **profile_path**: 指定Envoy性能分析数据的保存路径。
- **access_log_path**: 指定Envoy管理接口的访问日志保存路径。
- **address**: 配置管理接口的监听地址和端口。这里使用`0.0.0.0`表示监听所有网络接口，`port_value: 9901`是管理接口的端口。

### Static Resources 部分

static_resources:

这个部分定义了不会在运行时更改的资源，比如监听器（listeners）和集群（clusters）。

#### Listeners

  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
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
            - name: webservice
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: web_cluster_01 }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router

- **listener_0**: 一个监听器配置，监听所有接口的80端口，用于HTTP流量。
- **http_connection_manager**: 这是一个Envoy的网络过滤器，用于管理HTTP连接和路由。
  - **stat_prefix**: 流量统计前缀。
  - **codec_type**: HTTP编解码器类型，`AUTO`表示自动选择。
  - **route_config**: 路由配置，包括虚拟主机和路由规则。在此配置中，所有域（`domains: ["*"]`）的根路径（`prefix: "/"`）都会被路由到名为`web_cluster_01`的集群。
  - **http_filters**: HTTP过滤器链，这里使用了基本的路由器过滤器来处理路由决策。

#### Clusters

  clusters:
  - name: web_cluster_01
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: web_cluster_01
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: myservice, port_value: 80 }
    health_checks:
    - timeout: 5s
      interval: 10s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /livez
        expected_statuses:
          start: 200
          end: 399

- **web_cluster_01**: 一个集群配置，指定如何连接到服务。
- **connect_timeout**: 连接超时设置。
- **type**: 解析策略，`STRICT_DNS`表示基于DNS严格解析。
- **lb_policy**: 负载均衡策略，这里使用`ROUND_ROBIN`表示轮询。
- **load_assignment**: 指定集群的负载分配和端点。这里端点通过DNS名`myservice`在80端口上进行连接。
- **health_checks**: 健康检查配置，定期检查服务的健康状态，这里使用HTTP健康检查。
```

```
# vim envoy-sidecar-proxy.yaml
# cat envoy-sidecar-proxy.yaml
admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
       address: 0.0.0.0
       port_value: 9901

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
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
                route: { cluster: local_cluster }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router

  clusters:
  - name: local_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: local_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: 127.0.0.1, port_value: 8080 }
```

```
上述内容解释：
这个YAML文件是Envoy代理的配置文件，定义了Envoy如何管理和路由网络流量。该文件包含两个主要部分：`admin`和`static_resources`。以下是这些部分的详细解释：

### Admin 配置

admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
       address: 0.0.0.0
       port_value: 9901

这部分配置了Envoy的管理接口：
- **profile_path**: 指定性能分析文件的存储路径，用于记录性能相关的数据。
- **access_log_path**: 指定管理接口访问日志的存储路径，记录对管理接口的所有访问。
- **address**: 定义管理接口的监听地址。`0.0.0.0`表示监听所有网络接口，而`port_value: 9901`是监听的端口，使得管理接口可以从任何地址访问。

### Static Resources 配置

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
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
                route: { cluster: local_cluster }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router

这部分配置了Envoy的监听器和相关的过滤器：
- **listeners**: 配置一个名为`listener_0`的监听器，监听所有接口的80端口。
- **http_connection_manager**: 是一个网络过滤器，负责管理HTTP连接和路由。
  - **stat_prefix**: 指定统计数据的前缀。
  - **codec_type**: 设置HTTP编解码器，`AUTO`自动选择编解码器。
  - **route_config**: 定义路由配置，其中包括虚拟主机和路由规则。此处路由配置对所有域(`"*"`)的根路径(`"/"`)的请求路由到名为`local_cluster`的集群。
  - **http_filters**: 定义HTTP过滤器链，这里包括一个路由器过滤器，负责执行路由决策。

### Clusters 配置

  clusters:
  - name: local_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: local_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: 127.0.0.1, port_value: 8080 }

这部分定义了一个名为`local_cluster`的集群：
- **connect_timeout**: 连接超时时间设置为0.25秒。
- **type**: 集群类型为`STATIC`，表示集群的服务节点是静态配置的。
- **lb_policy**: 负载均衡策略为`ROUND_ROBIN`，即轮询方式。
- **load_assignment**: 指定集群服务节点的分配，这里配置的节点是本地的`127.0.0.1`地址，端口`8080`。
```

```
# vim README.md

# cat README.md

# Http Health Check Demo

### 环境说明
五个Service:
- envoy：Front Proxy,地址为172.29.1.2
- webserver01：第一个后端服务
- webserver01-sidecar：第一个后端服务的Sidecar Proxy,地址为172.29.1.3
- webserver02：第二个后端服务
- webserver02-sidecar：第二个后端服务的Sidecar Proxy,地址为172.29.1.4

### 运行和测试
1. 创建

docker-compose up


2. 测试

# 持续请求服务上的特定路径/livez
while true; do curl 172.29.1.2; sleep 1; done

# 等服务调度就绪后，另启一个终端，修改其中任何一个服务的/livez响应为非"OK"值，例如，修改第一个后端端点;
curl -X POST -d 'livez=FAIL' http://172.29.1.3/livez

# 通过请求的响应结果即可观测服务调度及响应的记录

# 请求中，可以看出第一个端点因主动健康状态检测失败，因而会被自动移出集群，直到其再次转为健康为止；
# 我们可使用类似如下命令修改为正常响应结果；
curl -X POST -d 'livez=OK' http://172.29.1.3/livez


3. 停止后清理

docker-compose down
```

```
# docker-compose up
 [+] Running 6/6
  ✔ Network envoy_cluster_health_checks_envoymesh                Created                     0.1s
  ✔ Container envoy_cluster_health_checks-webserver01-sidecar-1  Created                     0.0s
  ✔ Container envoy_cluster_health_checks-webserver02-sidecar-1  Created                     0.0s
  ✔ Container envoy_cluster_health_checks-webserver02-1          Created                     0.0s
  ✔ Container envoy_cluster_health_checks-webserver01-1          Created                     0.0s
  ✔ Container envoy_cluster_health_checks-envoy-1                Created                     0.0s
 Attaching to envoy-1, webserver01-1, webserver01-sidecar-1, webserver02-1, webserver02-sidecar-1
```

```
在没有访问的情况下，可以看看webserver访问日志，如下：
 webserver01-1          | 127.0.0.1 - - [28/May/2024 08:05:07] "GET /livez HTTP/1.1" 200 -
 webserver02-1          | 127.0.0.1 - - [28/May/2024 08:05:07] "GET /livez HTTP/1.1" 200 -
```

```
重新打开一个终端进行访问
 
 # curl http://172.29.1.2
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 
 # curl http://172.29.1.2
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: blue, ServerIP: 172.29.1.3!
 
 当访问时，日志如下：
 webserver02-1          | 127.0.0.1 - - [28/May/2024 08:08:52] "GET / HTTP/1.1" 200 -
 webserver01-1          | 127.0.0.1 - - [28/May/2024 08:08:53] "GET / HTTP/1.1" 200 -
```

```
查看listeners
 # curl http://172.29.1.2:9901/listeners
 listener_0::0.0.0.0:80
```

```
查看clusters
 # curl http://172.29.1.2:9901/clusters
 web_cluster_01::observability_name::web_cluster_01
 web_cluster_01::default_priority::max_connections::1024
 web_cluster_01::default_priority::max_pending_requests::1024
 web_cluster_01::default_priority::max_requests::1024
 web_cluster_01::default_priority::max_retries::3
 web_cluster_01::high_priority::max_connections::1024
 web_cluster_01::high_priority::max_pending_requests::1024
 web_cluster_01::high_priority::max_requests::1024
 web_cluster_01::high_priority::max_retries::3
 web_cluster_01::added_via_api::false
 web_cluster_01::172.29.1.3:80::cx_active::1
 web_cluster_01::172.29.1.3:80::cx_connect_fail::0
 web_cluster_01::172.29.1.3:80::cx_total::1
 web_cluster_01::172.29.1.3:80::rq_active::0
 web_cluster_01::172.29.1.3:80::rq_error::0
 web_cluster_01::172.29.1.3:80::rq_success::1
 web_cluster_01::172.29.1.3:80::rq_timeout::0
 web_cluster_01::172.29.1.3:80::rq_total::1
 web_cluster_01::172.29.1.3:80::hostname::myservice
 web_cluster_01::172.29.1.3:80::health_flags::healthy
 web_cluster_01::172.29.1.3:80::weight::1
 web_cluster_01::172.29.1.3:80::region::
 web_cluster_01::172.29.1.3:80::zone::
 web_cluster_01::172.29.1.3:80::sub_zone::
 web_cluster_01::172.29.1.3:80::canary::false
 web_cluster_01::172.29.1.3:80::priority::0
 web_cluster_01::172.29.1.3:80::success_rate::-1
 web_cluster_01::172.29.1.3:80::local_origin_success_rate::-1
 web_cluster_01::172.29.1.4:80::cx_active::1
 web_cluster_01::172.29.1.4:80::cx_connect_fail::0
 web_cluster_01::172.29.1.4:80::cx_total::1
 web_cluster_01::172.29.1.4:80::rq_active::0
 web_cluster_01::172.29.1.4:80::rq_error::0
 web_cluster_01::172.29.1.4:80::rq_success::1
 web_cluster_01::172.29.1.4:80::rq_timeout::0
 web_cluster_01::172.29.1.4:80::rq_total::1
 web_cluster_01::172.29.1.4:80::hostname::myservice
 web_cluster_01::172.29.1.4:80::health_flags::healthy
 web_cluster_01::172.29.1.4:80::weight::1
 web_cluster_01::172.29.1.4:80::region::
 web_cluster_01::172.29.1.4:80::zone::
 web_cluster_01::172.29.1.4:80::sub_zone::
 web_cluster_01::172.29.1.4:80::canary::false
 web_cluster_01::172.29.1.4:80::priority::0
 web_cluster_01::172.29.1.4:80::success_rate::-1
 web_cluster_01::172.29.1.4:80::local_origin_success_rate::-1
```

```
访问livez，确认状态为OK
 # curl http://172.29.1.2/livez
 OK
```

```
使用while循环多次访问查看状态，所有的上游主机都在
 # while true; do curl 172.29.1.2; sleep 1; done
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: blue, ServerIP: 172.29.1.3!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: blue, ServerIP: 172.29.1.3!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
```

```
为指定主机设置livez=FAIL后再访问
 # curl -X POST -d 'livez=FAIL' http://172.29.1.3/livez
```

```
通过日志可以查看到通过POST方法提交：
 webserver01-1          | 127.0.0.1 - - [28/May/2024 08:54:02] "POST /livez HTTP/1.1" 200 -
```

```
使用while循环多次访问，可以看到上述指定主机已不存在：
 # while true; do curl 172.29.1.2; sleep 1; done
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
```

```
查看检测日志时，可以看到有506状态码出现，表示服务器没有正确配置。
 webserver02-1          | 127.0.0.1 - - [28/May/2024 08:49:08] "GET /livez HTTP/1.1" 200 -
 webserver01-1          | 127.0.0.1 - - [28/May/2024 08:49:08] "GET /livez HTTP/1.1" 506 -
 webserver02-1          | 127.0.0.1 - - [28/May/2024 08:49:18] "GET /livez HTTP/1.1" 200 -
 webserver01-1          | 127.0.0.1 - - [28/May/2024 08:49:18] "GET /livez HTTP/1.1" 506 -
 webserver02-1          | 127.0.0.1 - - [28/May/2024 08:49:28] "GET /livez HTTP/1.1" 200 -
 webserver01-1          | 127.0.0.1 - - [28/May/2024 08:49:28] "GET /livez HTTP/1.1" 506 -
```

```
# curl -X POST -d 'livez=OK' http://172.29.1.3/livez
```

```
# while true; do curl 172.29.1.2; sleep 1; done
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: blue, ServerIP: 172.29.1.3!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: yellow, ServerIP: 172.29.1.4!
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: blue, ServerIP: 172.29.1.3!
```

**查看上述web服务代码实现**

```
# docker-compose exec webserver01 /bin/sh
 [root@blue /]# cd /usr/local/bin/
```

```
[root@blue /usr/local/bin]# cat demo.py
 #!/usr/bin/python3
 #
 from flask import Flask, request, abort, Response, jsonify as flask_jsonify, make_response
 import argparse
 import sys, os, getopt, socket, json, time
 
 app = Flask(__name__)
 
 @app.route('/')
 def index():
     return ('demoapp v1.0 !! ClientIP: {}, ServerName: {}, '
           'ServerIP: {}!\n'.format(request.remote_addr, socket.gethostname(),
                                   socket.gethostbyname(socket.gethostname())))
 
 @app.route('/hostname')
 def hostname():
     return ('ServerName: {}\n'.format(socket.gethostname()))
 
 health_status = {'livez': 'OK', 'readyz': 'OK'}
 probe_count = {'livez': 0, 'readyz': 0}
 
 @app.route('/livez', methods=['GET','POST'])
 def livez():
     if request.method == 'POST':
         status = request.form['livez']
         health_status['livez'] = status
         return ''
 
     else:
         if probe_count['livez'] == 0:
             time.sleep(5)
         probe_count['livez'] += 1
         if health_status['livez'] == 'OK':
             return make_response((health_status['livez']), 200)
         else:
             return make_response((health_status['livez']), 506)
 
 @app.route('/readyz', methods=['GET','POST'])
 def readyz():
     if request.method == 'POST':
         status = request.form['readyz']
         health_status['readyz'] = status
         return ''
 
     else:
         if probe_count['readyz'] == 0:
             time.sleep(15)
         probe_count['readyz'] += 1
         if health_status['readyz'] == 'OK':
             return make_response((health_status['readyz']), 200)
         else:
             return make_response((health_status['readyz']), 507)
 
 @app.route('/configs')
 def configs():
     return ('DEPLOYENV: {}\nRELEASE: {}\n'.format(os.environ.get('DEPLOYENV'), os.environ.get('RELEASE')))
 
 @app.route("/user-agent")
 def view_user_agent():
     # user_agent=request.headers.get('User-Agent')
     return('User-Agent: {}\n'.format(request.headers.get('user-agent')))
 
 def main(argv):
     port = 80
     host = '0.0.0.0'
     debug = False
 
     if os.environ.get('PORT') is not None:
         port = os.environ.get('PORT')
 
     if os.environ.get('HOST') is not None:
         host = os.environ.get('HOST')
 
     try:
         opts, args = getopt.getopt(argv,"vh:p:",["verbose","host=","port="])
     except getopt.GetoptError:
         print('server.py -p <portnumber>')
         sys.exit(2)
     for opt, arg in opts:
         if opt in ("-p", "--port"):
             port = arg
         elif opt in ("-h", "--host"):
             host = arg
         elif opt in ("-v", "--verbose"):
             debug = True
 
     app.run(host=str(host), port=int(port), debug=bool(debug))
 
 
 if __name__ == "__main__":
     main(sys.argv[1:])
```

```
上述代码解释：
这段Python代码是一个使用Flask框架创建的简单Web服务器应用程序，通常用于提供API端点和处理HTTP请求。让我们逐部分解析这个代码：

### 导入模块
代码首先导入了Flask和其他几个Python标准库，用于网络、系统和命令行参数处理。

### Flask 应用实例

app = Flask(__name__)

创建了一个Flask应用实例，这是任何Flask应用的起点。

### 路由和视图函数
Flask 使用装饰器 `@app.route()` 来绑定URL到Python函数。这里定义了几个路由：

1. **根路由("/")**:
   - 返回客户端IP地址、服务器名称和服务器IP。
   
   @app.route('/')
   def index():
       return ('demoapp v1.0 !! ClientIP: {}, ServerName: {}, ServerIP: {}!\n'.format(request.remote_addr, socket.gethostname(), socket.gethostbyname(socket.gethostname())))
 

2. **主机名("/hostname")**:
   - 返回服务器的主机名。
  
   @app.route('/hostname')
   def hostname():
       return ('ServerName: {}\n'.format(socket.gethostname()))


3. **存活探针("/livez")**:
   - 用于存活性检查，可以通过POST方法更新状态。
   - 第一次GET请求时会延迟5秒，模拟启动延迟。

   @app.route('/livez', methods=['GET','POST'])
   def livez():
       # POST方法更新状态，GET方法返回当前状态


4. **就绪探针("/readyz")**:
   - 用于就绪性检查，同样支持状态更新。
   - 第一次GET请求时会延迟15秒，模拟就绪延迟。

   @app.route('/readyz', methods=['GET','POST'])
   def readyz():
       # 类似livez处理逻辑


5. **配置信息("/configs")**:
   - 返回环境变量中的`DEPLOYENV`和`RELEASE`值。

   @app.route('/configs')
   def configs():
       return ('DEPLOYENV: {}\nRELEASE: {}\n'.format(os.environ.get('DEPLOYENV'), os.environ.get('RELEASE')))


6. **用户代理信息("/user-agent")**:
   - 返回请求中的`User-Agent`头信息。

   @app.route("/user-agent")
   def view_user_agent():
       return('User-Agent: {}\n'.format(request.headers.get('user-agent')))


### 主函数
定义了一个`main()`函数，处理命令行参数并启动Flask服务器。
- 默认在`0.0.0.0`的80端口监听。
- 支持命令行参数修改端口(`-p`)、主机(`-h`)和开启调试模式(`-v`或`--verbose`)。
- 通过环境变量`PORT`和`HOST`可进一步配置端口和主机。

### 程序入口点

if __name__ == "__main__":
    main(sys.argv[1:])

当直接运行这个脚本时，将调用`main()`函数，传入除了脚本名称外的所有命令行参数。
```

### 4.2 基于tcp协议主动健康检测 <a href="#id-42-ji-yu-tcp-xie-yi-zhu-dong-jian-kang-jian-ce-65" id="id-42-ji-yu-tcp-xie-yi-zhu-dong-jian-kang-jian-ce-65"></a>

> 由于web应用前端有envoy代理，所以本案例验证时选择直接关闭envoy代理。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719840287059/7742d371329647e1bf6f5230ee58955d.png)

```
# mkdir envoy_cluster_health_checks_tcp
 # cd envoy_cluster_health_checks_tcp
```

```
# vim docker-compose.yaml
 
 # cat docker-compose.yaml
 version: '3.3'
 
 services:
   envoy:
     image: envoyproxy/envoy:v1.30.1
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
     volumes:
     - ./front-envoy-with-tcp-check.yaml:/etc/envoy/envoy.yaml
     networks:
       envoymesh:
         ipv4_address: 172.30.1.2
         aliases:
         - front-proxy
     depends_on:
     - webserver01-sidecar
     - webserver02-sidecar
 
   webserver01-sidecar:
     image: envoyproxy/envoy:v1.30.1
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
     volumes:
     - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
     hostname: blue
     networks:
       envoymesh:
         ipv4_address: 172.30.1.3
         aliases:
         - myservice
 
   webserver01:
     image: www.kubemsb.com/envoy/demoapp:v1.0
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
       - PORT=8080
       - HOST=127.0.0.1
     network_mode: "service:webserver01-sidecar"
     depends_on:
     - webserver01-sidecar
 
   webserver02-sidecar:
     image: envoyproxy/envoy:v1.30.1
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
     volumes:
     - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
     hostname: yellow
     networks:
       envoymesh:
         ipv4_address: 172.30.1.4
         aliases:
         - myservice
 
   webserver02:
     image: www.kubemsb.com/envoy/demoapp:v1.0
     environment:
       - ENVOY_UID=0
       - ENVOY_GID=0
       - PORT=8080
       - HOST=127.0.0.1
     network_mode: "service:webserver02-sidecar"
     depends_on:
     - webserver02-sidecar
 
 networks:
   envoymesh:
     driver: bridge
     ipam:
       config:
         - subnet: 172.30.1.0/24
```

```
# vim front-envoy-with-tcp-check.yaml
 
 # cat front-envoy-with-tcp-check.yaml
 admin:
   profile_path: /tmp/envoy.prof
   access_log_path: /tmp/admin_access.log
   address:
     socket_address: { address: 0.0.0.0, port_value: 9901 }
 
 static_resources:
   listeners:
   - name: listener_0
     address:
       socket_address: { address: 0.0.0.0, port_value: 80 }
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
             - name: webservice
               domains: ["*"]
               routes:
               - match: { prefix: "/" }
                 route: { cluster: web_cluster_01 }
           http_filters:
           - name: envoy.filters.http.router
             typed_config:
               "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
   clusters:
   - name: web_cluster_01
     connect_timeout: 0.25s
     type: STRICT_DNS
     lb_policy: ROUND_ROBIN
     load_assignment:
       cluster_name: web_cluster_01
       endpoints:
       - lb_endpoints:
         - endpoint:
             address:
               socket_address: { address: myservice, port_value: 80 }
     health_checks:
     - timeout: 5s
       interval: 10s
       unhealthy_threshold: 2
       healthy_threshold: 2
       tcp_health_check: {}
```

```
# vim envoy-sidecar-proxy.yaml
 
 # cat envoy-sidecar-proxy.yaml
 admin:
   profile_path: /tmp/envoy.prof
   access_log_path: /tmp/admin_access.log
   address:
     socket_address:
        address: 0.0.0.0
        port_value: 9901
 
 static_resources:
   listeners:
   - name: listener_0
     address:
       socket_address: { address: 0.0.0.0, port_value: 80 }
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
                 route: { cluster: local_cluster }
           http_filters:
           - name: envoy.filters.http.router
             typed_config:
               "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
 
   clusters:
   - name: local_cluster
     connect_timeout: 0.25s
     type: STATIC
     lb_policy: ROUND_ROBIN
     load_assignment:
       cluster_name: local_cluster
       endpoints:
       - lb_endpoints:
         - endpoint:
             address:
               socket_address: { address: 127.0.0.1, port_value: 8080 }
```

```
在终端1中运行
 # docker-compose up
```

```
在终端2中查看状态
 # curl http://172.30.1.2:9901/stats | grep health_check
   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                  Dload  Upload   Total   Spent    Left  Speed
 100 20281    0 20281    0     0  17.0M      0 --:--:-- --:--:-- --:--:-- 19.3M
 cluster.web_cluster_01.health_check.attempt: 8
 cluster.web_cluster_01.health_check.degraded: 0
 cluster.web_cluster_01.health_check.failure: 0
 cluster.web_cluster_01.health_check.healthy: 2
 cluster.web_cluster_01.health_check.network_failure: 0
 cluster.web_cluster_01.health_check.passive_failure: 0
 cluster.web_cluster_01.health_check.success: 8
 cluster.web_cluster_01.health_check.verify_cluster: 0
 http.ingress_http.tracing.health_check: 0
```

```
在终端2中执行
 # docker ps
 CONTAINER ID   IMAGE                                COMMAND                   CREATED         STATUS         PORTS       NAMES
 a1f2d190db5d   envoyproxy/envoy:v1.30.1             "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   10000/tcp   envoy_cluster_health_checks_tcp-envoy-1
 eba5e2d21e26   www.kubemsb.com/envoy/demoapp:v1.0   "/bin/sh -c 'python3…"   3 minutes ago   Up 3 minutes               envoy_cluster_health_checks_tcp-webserver01-1
 a14fac3a0265   www.kubemsb.com/envoy/demoapp:v1.0   "/bin/sh -c 'python3…"   3 minutes ago   Up 3 minutes               envoy_cluster_health_checks_tcp-webserver02-1
 0cd68453fa48   envoyproxy/envoy:v1.30.1             "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   10000/tcp   envoy_cluster_health_checks_tcp-webserver02-sidecar-1
 cad933da773d   envoyproxy/envoy:v1.30.1             "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   10000/tcp   envoy_cluster_health_checks_tcp-webserver01-sidecar-1
```

```
在终端2中执行
 # docker stop envoy_cluster_health_checks_tcp-webserver01-sidecar-1
 envoy_cluster_health_checks_tcp-webserver01-sidecar-1
```

```
在终端2中执行
 # curl http://172.30.1.2:9901/stats | grep health_check
   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                  Dload  Upload   Total   Spent    Left  Speed
 100 20302    0 20302    0     0  20.8M      0 --:--:-- --:--:-- --:--:-- 19.3M
 cluster.web_cluster_01.health_check.attempt: 12
 cluster.web_cluster_01.health_check.degraded: 0
 cluster.web_cluster_01.health_check.failure: 1
 cluster.web_cluster_01.health_check.healthy: 2
 cluster.web_cluster_01.health_check.network_failure: 1
 cluster.web_cluster_01.health_check.passive_failure: 0
 cluster.web_cluster_01.health_check.success: 11
 cluster.web_cluster_01.health_check.verify_cluster: 0
 http.ingress_http.tracing.health_check:
```
