# 线程模型

理解Envoy的单进程/多线程架构模型，可以更好地了解其高效性能和可扩展性。

## 1.1 架构模型概述 <a href="#id-321-jia-gou-mo-xing-gai-shu-104" id="id-321-jia-gou-mo-xing-gai-shu-104"></a>

Envoy使用单进程/多线程的架构模型，这意味着Envoy作为一个单一的进程运行，内部通过多个线程来处理不同的任务。

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>主线程（Main Thread）</summary>

主线程是Envoy的核心管理线程，负责多个关键管理任务：

1. **启动和关闭**：负责Envoy程序的启动和关闭，包括初始化配置和资源。
2. **xDS API调用处理**：处理与xDS API的交互，xDS（Extensible Service Discovery and Configuration）是Envoy用来动态获取配置信息的机制。包括集群管理、服务发现、负载均衡等。
3. **DNS和健康状态检测**：管理DNS解析和集群健康检查，确保服务的可用性。
4. **运行时配置**：管理运行时配置更新，允许动态修改配置而无需重启Envoy。
5. **统计数据刷新**：定期刷新和维护统计数据，用于监控和分析。
6. **线程管理**：维护其他工作线程，包括信号处理和热重启等操作。

主线程中的所有事件都是以异步非阻塞模式完成的，这确保了高效的管理操作，不会阻塞其他工作线程的执行。

主线程负责Envoy的管理和控制任务，它的职责包括：

* **xDS**：处理与xDS API的交互，管理动态配置，包括服务发现和负载均衡。
* **Runtime**：管理运行时配置，允许动态修改配置而无需重启Envoy。
* **Stat flush**：负责统计数据的刷新和维护，提供监控和分析信息。
* **Admin**：维护管理接口，提供操作和控制功能。
* **Process management**：处理信号和热重启等进程管理任务。

主线程以<mark style="color:blue;">**异步非阻塞**</mark>的方式处理这些任务，确保不会阻塞其他线程的执行。

</details>

<details>

<summary>工作线程（Worker Threads）</summary>

工作线程是Envoy处理实际代理功能的核心：

1. **线程数量**：默认情况下，Envoy根据主机的CPU核心数创建等量的工作线程，管理员也可以通过 `--concurrency`选项指定具体数量。
2. **非阻塞事件循环**：每个工作线程运行一个非阻塞的事件循环，负责处理监听器分配的套接字、接收新请求，并初始化和管理过滤器栈。
3. **请求处理**：处理请求的整个生命周期，包括解析请求、应用过滤器和转发请求等。

工作线程负责实际的请求处理，每个工作线程包含以下部分：

* **Listeners**：监听器，负责接受新连接。Envoy可以配置多个监听器，每个监听器可以监听不同的端口和协议。
* **Connections**：连接管理，处理每个连接的生命周期，包括请求的接收、解析、处理和响应。

每个工作线程运行一个非阻塞事件循环，确保高效地处理大量并发连接。默认情况下，Envoy会根据主机的CPU核心数创建等量的工作线程，但也可以通过 `--concurrency`选项指定具体数量。

</details>

<details>

<summary>文件刷写线程（File Flush Thread）</summary>

文件刷写线程负责Envoy的日志和其他文件写入操作：

1. **独立线程**：每个文件写入操作都有一个专用的独立阻塞型刷写线程。
2. **内存缓冲区**：工作线程将数据写入内存缓冲区，然后由文件刷写线程同步写入文件。
3. **同步刷写**：确保数据最终一致性和文件的持久化存储。

文件刷写线程负责Envoy的日志和其他文件写入操作。每个文件写入都有一个独立的阻塞型刷写线程：

* **内存缓冲区**：工作线程将数据写入内存缓冲区。
* **文件同步**：文件刷写线程将内存缓冲区中的数据同步写入文件，确保数据的一致性和持久性。

</details>

**总结**

Envoy的单进程/多线程架构模型实现了高效的资源管理和任务处理：

* **主线程**处理管理任务和动态配置，确保代理的可用性和灵活性。
* **工作线程**并发处理实际的请求，提供高性能的代理服务。
* **文件刷写线程**确保日志和数据的持久化，独立于主线程和工作线程，避免阻塞核心代理功能。

这种架构使Envoy能够在高负载和高并发的环境中高效运行，同时提供动态配置和灵活管理的能力。

## 1.2 Envoy连接处理流程 <a href="#si-envoy-lian-jie-chu-li-liu-cheng-129" id="si-envoy-lian-jie-chu-li-liu-cheng-129"></a>

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

1. **Worker Thread**
   * **工作线程**负责处理具体的请求。Envoy根据主机的CPU核心数创建等量的工作线程（可通过 `--concurrency`选项指定数量）。每个工作线程运行一个非阻塞事件循环来处理请求。
2. **Listener Filters**
   * **监听过滤器**是请求处理的第一个环节。它们在连接刚建立时应用，用于执行一些初步处理，如协议检测和SNI（服务器名称指示）处理。
3. **Connection**
   * **连接管理**组件接受新连接并管理连接的整个生命周期，包括连接的建立、维护和关闭。
4. **TCP Filter Manager**
   * **TCP过滤器管理器**负责调度TCP层的读写过滤器。它在连接建立后，管理TCP过滤器的执行顺序和逻辑。
5. **TCP Read Filters**
   * **TCP读过滤器**处理TCP层的数据读取。它们可以执行如协议解码、数据验证等操作，将TCP数据转换为更高级别的协议数据。
6. **HTTP Codec**
   * **HTTP编解码器**负责将TCP层的数据解码为HTTP请求。它将字节流解析为HTTP请求头、请求体等结构化数据，供后续处理。
7. **HTTP Connection Manager**
   * **HTTP连接管理器**负责管理HTTP连接的生命周期，包括HTTP请求的接收、处理和响应发送。它是HTTP请求处理的核心管理组件。
8. **HTTP Read Filters**
   * **HTTP读过滤器**处理HTTP请求数据，可以执行如认证、授权、请求修改等操作。它们在HTTP连接管理器中被调用，逐步处理HTTP请求。
9. **Service Router**
   * **服务路由器**基于配置的路由规则，将HTTP请求路由到适当的后端服务。它根据请求的路径、头信息等，将请求转发到正确的集群或服务实例。
10. **Upstream Connection Pool**
    * **上游连接池**管理与后端服务的连接，支持连接复用和负载均衡。它维护与后端服务的持久连接，优化连接的建立和管理。
11. **Backend Services**
    * **后端服务**是实际处理请求的服务器或服务实例。请求在这里得到处理，并生成响应。
12. **HTTP Write Filters**
    * **HTTP写过滤器**处理HTTP响应数据，可以对响应进行修改、添加头信息等操作。这些过滤器在响应发送给客户端之前被调用。
13. **HTTP Connection Manager**
    * **HTTP连接管理器**负责将HTTP响应数据编码为TCP数据。它将结构化的HTTP响应转为字节流，并准备发送回客户端。
14. **TCP Write Filters**
    * **TCP写过滤器**处理TCP层的数据写入。它们将编码好的TCP数据发送回客户端，完成整个请求处理流程。

**辅助模块**

* **Stats**
  * **统计模块**实时收集和报告Envoy的运行时统计数据，提供详细的性能和使用情况报告。
* **Admin**
  * **管理接口**提供Envoy的配置、控制和监控功能。管理员可以通过这个接口查看和修改Envoy的配置，监控运行状态。
* **Cluster/Listener/Route Manager**
  * **集群、监听器和路由管理器**负责管理集群的健康检查、服务发现和路由配置。它通过与xDS API交互动态更新配置。
* **xDS API**
  * **xDS API**用于动态配置和服务发现，包括CDS（Cluster Discovery Service）、EDS（Endpoint Discovery Service）、LDS（Listener Discovery Service）和RDS（Route Discovery Service）等。这些API允许Envoy在运行时动态获取和更新配置。

**配置示例文件**

```powershell
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

> http\_connection\_manager 专门用于激活7层代理功能的4层过滤器。
