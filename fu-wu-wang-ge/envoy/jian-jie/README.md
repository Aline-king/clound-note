# 简介

Envoy 是用 C++ 开发的高性能代理，用于协调服务网格中所有服务的入站和出站流量。&#x20;

Envoy 代理是唯一与数据平面流量交互的 Istio 组件。Envoy 代理被部署为服务的 Sidecar，在逻辑上为服务增加了 Envoy 的许多内置特性，例如：动态服务发现、负载均衡、TLS 终端、HTTP/2 与 gRPC 代理、熔断器、健康检查、基于百分比流量分割的分阶段发布、故障注入、丰富的指标等。 这种 Sidecar 部署允许 Istio 可以执行策略决策，并提取丰富的遥测数据，接着将这些数据发送到监视系统以提供有关整个网格行为的信息。Sidecar 代理模型还允许您向现有的部署添加 Istio 功能，而不需要重新设计架构或重写代码。

## 工作位置

* 前端代理
* SideCar代理

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
