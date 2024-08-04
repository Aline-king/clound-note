# 组件与架构解读

## <mark style="color:blue;">架构</mark>

istio架构设计分为数据平面和控制平面两部分。

1. &#x20;<mark style="color:purple;">**数据平面**</mark>：由一系列智能代理(sidecar)组成，该代理最大的特点在于可以通过API配置**实时生效**，实现动态化的流量代理。这些代理通过Mixer来控制所有微服务间的网络通信，Mixer是一个通用的策略和遥测中心。sidecar-官方默认是Envoy,但不强耦合
2. <mark style="color:purple;">**控制平面**</mark>：是数据平面中智能代理的统一配置与维护平台，不仅包括请求路由，还包括限流降级，熔断，日志收集，性能监控等各种请求。

总结

简单来说，Istio的数据平面主要负责流量转发、策略实施与遥测数据上报；控制层面主要负责接收用户配置生成路由规则、分发路由规则到代理、分发策略与遥测数据收集。

> 资料来源：https://istio.io/latest/zh/docs/ops/deployment/architecture/

## <mark style="color:blue;">**组件**</mark>

{% tabs %}
{% tab title="Pilot" %}
为数据平面的Envoy代理提供服务发现功能，并提供智能路由功能（例如：A/B测试、金丝雀发布等）和弹性功能（例如：超时、重试、熔断器等）。它将控制流量行为的高级路由规则转换为特定于Envoy 的配置，并在运行时将它们传播到 Envoy代理。 Pilot抽象了平台相关的服务发现机制，并转换成Envoy数据平面支持的标准格式。这种松耦合设计使得Istio能运行在多平台环境(Kubernetes、Consul、Cloud Foundry、Apache Mesos等)，并保持一致的流量管理接口。
{% endtab %}

{% tab title="mixer" %}
是一个与平台无关的组件。Mixer负责在服务网络中实施访问控制和策略，并负责从Envoy代理和其他服务上收集遥测数据。代理提取请求级别的属性并发送到Mixer用于评估，评估请求是否能放行。 Mixer有一个灵活的插件模型。这个模型使得Istio可以与多种主机环境和后端基础设施对接。因此，Istio从这些细节中抽象了Envoy代理和Istio管理的服务。 从逻辑上可以看出 Mixer 可以提供两种服务：后端逻辑抽象 + 中心化的控制，为了实现这两个功能，每个请求到达sidecar的时候，都需要向Mixer 发送一次请求，前置检查。请求结束之后再向Mixer做一次汇报。 从本质上来说，Mixer本身获取的指标，本身就是来源于sidecar内部的envoy，然后再交给其他的可视化平台，这就走了两步，还不如一步到位交给可视化平台，所以它的出身就不好。
{% endtab %}
{% endtabs %}
