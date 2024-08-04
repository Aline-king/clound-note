# 架构设计

Envoy 采⽤单进程多线程架构。

⼀个独⽴的 primary 线程负责控制各种零散的协调任务，⽽⼀些 worker 线程则负责执⾏监听、过滤和转发任务。

⼀旦侦听器接受连接，该连接就会将其⽣命周期绑定到⼀个单独的 worker 线程。这使得 Envoy 的⼤部分⼯作基本上是单线程来处理的，只有少量更复杂的代码处理⼯作线程之间的协调。

通常情况下 Envoy 实现了 100% ⾮阻塞 。对于⼤多数⼯作负载，我们建议将 worker 线程的数量配置为机器上的硬件线程数量。

Envoy 整体架构如下图所示：

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

这幅图展示了Envoy Proxy的架构和工作原理。Envoy Proxy是一个开源的边缘服务代理，用于在微服务体系结构中提供网络流量控制。

1. **Listener**：Envoy Proxy监听特定端口上的传入连接。这些连接可以是HTTP或TCP协议。
2. **Filter Chains**：接收到连接后，它们通过一系列过滤器链进行处理。这些过滤器可以执行各种操作，如身份验证、速率限制、日志记录等。
3. **Network Filters**：网络过滤器负责处理底层的网络通信。例如，Dubbo proxy允许Envoy与Dubbo服务交互，MySQL proxy则允许它作为MySQL服务器的代理。
4. **HTTP Connection Manager**：HTTP连接管理器处理HTTP请求，并将它们路由到正确的上游服务。
5. **Upstream Services**：Envoy将请求转发给上游服务（也称为集群）。每个集群都有一组关联的endpoint，这些endpoint代表实际的服务实例。
6. **Load Balancing**：Envoy使用负载均衡策略来决定将请求发送到哪个endpoint。这确保了请求均匀地分布在整个集群上。
7. **xDS API**：Envoy使用xDS（扩展发现服务）API从集中式配置源获取其配置信息。这包括监听器、路由规则、集群定义等。
8. **Downstream Clients**：最后，Envoy将响应返回给下游客户端，完成整个请求-响应周期。

Envoy Proxy作为一个透明的中间层，为应用程序提供了强大的网络功能，同时保持了对应用程序代码的最小干扰。

Envoy 进程中运行着一系列 Inbound/Outbound 监听器（Listener），Inbound 代理入站流量，Outbound 代理出站流量。Listener 的核心就是过滤器链（FilterChain），**链中每个过滤器都能够控制流量的处理流程**。

Envoy 接收到请求后，会先走 `FilterChain`，通过各种 L3/L4/L7 Filter 对请求进行处理，然后再路由到指定的集群，并通过负载均衡获取一个目标地址，最后再转发出去。

其中每一个环节可以静态配置，也可以动态服务发现，也就是所谓的 `xDS`，这里的 `x` 是一个代词，是 `lds`、`rds`、`cds`、`eds`、`sds` 的总称，即服务发现，后 2 个字母 `ds` 就是 `discovery service`。

> xDS可以说是envoy里面的一个重点，也是envoy的一个特色。
