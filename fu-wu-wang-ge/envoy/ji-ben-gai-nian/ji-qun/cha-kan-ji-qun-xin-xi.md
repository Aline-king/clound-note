# 查看集群信息

这段输出是Envoy服务端点`http://172.25.1.2:9901/clusters`返回的集群状态信息，主要反映了名为`local_cluster`的集群配置和运行状况。以下是对关键行的详细解读：

```
local_cluster::observability_name::local_cluster: 表示集群的可观测性名称也是`local_cluster`。
local_cluster::default_priority::max_connections::1024: 在默认优先级下，允许的最大并发连接数为1024。
local_cluster::default_priority::max_pending_requests::1024: 默认优先级下，等待处理的最大请求数为1024。
local_cluster::default_priority::max_requests::1024: 默认优先级下，单个连接上允许的最大请求数为1024。
local_cluster::default_priority::max_retries::3: 默认优先级下的最大重试次数为3次。
local_cluster::high_priority::... 类似于上述但针对高优先级设置。
local_cluster::added_via_api::false: 表明这个集群不是通过API动态添加的。
local_cluster::172.25.1.3:8080::cx_active::0: 针对端点`172.25.1.3:8080`，当前活跃的连接数为0。
local_cluster::172.25.1.3:8080::cx_connect_fail::0: 连接失败次数为0。
local_cluster::172.25.1.3:8080::cx_total::0: 总尝试连接次数为0。
local_cluster::172.25.1.3:8080::rq_active::0: 当前活跃的请求数为0。
local_cluster::172.25.1.3:8080::rq_error::0: 请求错误次数为0。
local_cluster::172.25.1.3:8080::rq_success::0: 成功的请求数为0。
local_cluster::172.25.1.3:8080::rq_timeout::0: 超时的请求数为0。
local_cluster::172.25.1.3:8080::rq_total::0: 总请求次数为0。
local_cluster::172.25.1.3:8080::hostname:: 未提供特定主机名。
local_cluster::172.25.1.3:8080::health_flags::healthy: 该端点被标记为健康。
local_cluster::172.25.1.3:8080::weight::1: 该端点的权重为1。
```

接下来的信息重复展示了另一个端点`172.25.1.4:8080`的状态，但请注意，对于此端点有实际的请求和成功数据（如`rq_success::1`和`rq_total::1`），表明至少有一个请求成功发送到该端点。

这份输出展示了`local_cluster`的配置限制、健康状况、以及两个后端服务器（IP地址分别为172.25.1.3和172.25.1.4，端口8080）的当前连接和请求处理统计信息。两个后端目前均健康且没有明显的性能问题，但具体负载和活动度较低，尤其是对第一个后端而言。



## 查看集群各项指标

这些内容是Envoy代理服务器的统计指标输出，每一行代表了不同维度的性能指标或者状态信息，用于监控和故障排查。下面是对每一行内容的详细解释：

这些指标提供了Envoy如何管理其内部状态、处理上游和下游连接、执行请求以及响应过载保护等方面的重要洞见。在性能调优、故障诊断或监控系统健康状况时，这些信息非常关键。

这些内容是Envoy代理服务器提供的详细统计信息，涵盖了从HTTP请求处理、监听器状态、工作线程行为、服务器配置到性能指标等多个维度的数据。

{% hint style="info" %}


<details>

<summary>Cluster Status Metrics (集群状态指标)</summary>

* `cluster.local_cluster.assignment_stale`, `assignment_timeout_received`, `assignment_use_cached`: 这些指标与集群内目标（如后端实例）分配有关，比如分配是否过时、是否收到分配超时通知、是否使用了缓存的分配信息。
* `bind_errors`, `circuit_breakers.*`: 绑定错误计数和各种断路器状态（如默认和高优先级下的连接、请求打开计数），断路器用于防止过载。
* `total_match_count`, `external.upstream_rq_*`: 匹配规则的总数，以及外部上游请求的成功（如HTTP状态码200、2xx）、完成的请求等。
* `http1.*`: 与HTTP/1.x相关的统计，如因下划线头部被丢弃的请求、元数据不支持错误、请求因下划线头部被拒绝等。
* `retry_or_shadow_abandoned`, `update_*`: 重试或影子操作放弃的计数，以及集群更新的各种状态（尝试、空更新、成功、失败等）。
* `upstream_cx_*`: 与上游连接相关的统计，包括活动连接数、关闭通知、连接尝试失败、连接超时、销毁原因等。
* `upstream_rq_*`: 上游请求相关的统计，例如请求的活跃数、取消、完成、重试、超时等。
* `version`, `warming_state`: 集群的版本信息和预热状态。
* `cluster_manager.*`: 集群管理器的统计，如活动集群数、添加、修改、移除的集群数等。

</details>

<details>

<summary>Envoy Overall Metrics (Envoy总体指标)</summary>

* `envoy.overload_actions.reset_high_memory_stream.count`: 过载保护动作导致的高内存流重置计数。
* `filesystem.*`: 文件系统操作相关的指标，例如定时刷新次数、写入失败、缓冲区状态等。
* `http.admin/downstream_cx_*`: 与Admin接口（Envoy管理界面）相关的下游连接统计，包括活跃连接、销毁原因、协议分布等。
* `http.ingress_http/downstream_cx_*`: 入口HTTP连接的下游统计，类似上面，但针对面向服务的实际入口流量。

</details>

<details>

<summary>HTTP请求处理相关</summary>

* `http.ingress_http.rs_too_large: 0`: 表示由于响应体太大而被Envoy拒绝的HTTP请求数量为0。
* `http.ingress_http.tracing.*`: 关于HTTP跟踪的统计，如客户端追踪启用、健康检查、不可追踪请求、随机采样等的计数均为0，说明追踪配置和行为的当前状态。
* `http1.*`: 与HTTP/1.x协议相关的统计，如因包含下划线的头部被丢弃的请求、不支持的元数据错误、请求被拒绝等。

</details>

<details>

<summary>监听器（Listener）</summary>

* `listener.0.0.0.0_80.*`: 指向所有IPv4地址上监听80端口的监听器状态，包括活跃连接数（downstream\_cx\_active）、销毁的连接数（downstream\_cx\_destroy）、连接溢出（downstream\_cx\_overflow）等。
* `listener.admin.*`: 特指管理接口（admin）监听器的相应统计信息，如活跃连接数、请求状态码统计（downstream\_rq\_XX）等。

</details>

<details>

<summary>工作线程和服务器状态</summary>

```
`worker_N.downstream_cx_active`: 每个工作线程（N表示线程编号）处理的下游连接活跃数。
`server.concurrency`: 服务器配置的并发数（工作线程数）。
`server.uptime`: 服务器运行时间（秒）。
`server.memory_allocated`: 分配的内存大小。
`server.version`: Envoy的版本号。
```

</details>

<details>

<summary>运行时和配置</summary>

```
`runtime.*`: 运行时配置的统计，如覆盖激活的运行时覆盖、过期特性警告、加载成功或错误等。
`server.*`: 服务器的配置设置和运行状况，包括FIPS模式、静态未知字段、统计查找次数、总连接数等。
```

</details>

<details>

<summary>性能指标统计分布（Percentiles）</summary>

```
P0(nan,0) 到 P100(nan,1.1) 表示诸如请求时间、连接长度等指标的百分位分布，这里“nan”表明没有足够的样本进行计算，而括号内的数字代表了各个百分位的值。
```

</details>

<details>

<summary>Overload Manager</summary>

```
overload.refresh_interval_delay: 如果有记录，则会显示重载管理器刷新间隔延迟，此处显示无记录值
```

</details>

<details>

<summary>初始化时间</summary>

```
server.initialization_time_ms: 服务器初始化时间的百分位分布，同样以“nan”开头表明缺少足够数据。
```

</details>
{% endhint %}

