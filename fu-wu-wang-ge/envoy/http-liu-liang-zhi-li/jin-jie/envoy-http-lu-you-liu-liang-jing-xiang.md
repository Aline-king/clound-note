# Envoy HTTP路由流量镜像

### 流量镜像介绍 <a href="#id-71-liu-liang-jing-xiang-jie-shao-103" id="id-71-liu-liang-jing-xiang-jie-shao-103"></a>

**流量镜像**，也称为复制或影子流量，功能通常用于在生产环境进行测试，通过将流量拷贝到集群或者新版本群，实现新版本接近真环境的测试，旨在有效地降低上线风险。

**流量镜像可用于以下场景**

1. **验证新版本**：
   * 实时对比镜像流量与生产环境的输出结果，完成对新版本的验证。
2. **测试**：
   * 用生产实例的真实流量进行模拟测试，以确保新版本在生产环境中的表现。
3. **隔离测试数据库**：
   * 对与处理相关的业务进行镜像流量操作，使用空存储加载，以实现测试数据的隔离和独立性。

### 7.2 配置HTTP流量镜像的方法 <a href="#id-72-pei-zhi-http-liu-liang-jing-xiang-de-fang-fa-107" id="id-72-pei-zhi-http-liu-liang-jing-xiang-de-fang-fa-107"></a>

通过将流量转发至一个主集群的同时，也将流量转发至另一个影子集群，从而实现流量镜像。这个配置不需要等待影子集群返回响应，并支持收集影子集群的常规统计信息，常用于测试。

**配置 HTTP 流量镜像的方法**

* **原理**：将流量转发至主集群的同时，也转发到影子集群，无需等待影子集群返回响应。
* **应用场景**：常用于在生产环境中进行测试，验证新版本的功能和性能。

**配置示例**

```yaml
route:
  cluster|weighted_clusters:
  ...
  request_mirror_policies: []
  - cluster: "..."  # 指定集群的名称。
    runtime_fraction:  # 用于控制镜像流量的比例
      default_value:  # 运行时键值不可用时，则使用此默认值；
        numerator: 0  # 指定分子，默认为0；
        denominator: HUNDRED  # 指定分母，小于分子时，最终百分比为1；分母可固定使用HUNDRED（默认）、TEN_THOUSAND和MILLION；
      runtime_key: routing.request_mirror.KEY  # 指定要使用的运行时键，其值需要用户自定义；
    trace_sampled: {…}  # 是否对trace span进行采样，默认为true
```

**流量比例配置**

* **runtime\_key**：用于明确定义向影子集群转发的流量的百分比，取值范围为 0-10000，每个数字表示 0.01% 的请求比例。若定义了此键但未指定其值，默认为 0。

**默认行为**

* 默认情况下，路由器会镜像所有请求。可以通过上述参数配置转发至影子集群的流量比例。
