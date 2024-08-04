---
description: 服务网格
---

# service mesh

Buoyant 公司的 CEO Willian Morgan 2017年初在他的这篇文章 WHAT’S A Service Mesh? AND WHY DO I NEED ONE? 中解释了什么是 Service Mesh，关于服务网格的定义内容如下：&#x20;

A Service Mesh is a dedicated infrastructure layer for handling service-to-service communication. It’s responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the Service Mesh is typically implemented\[应用] as an array of lightweight network proxies that are deployed alongside application code, without the application needing to be aware\[意识到].&#x20;

参考资料：https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/

通过原作者对服务网格的介绍，我们知道这么几个核心关键词： 服务间通信的基础设施层 云原生场景下，通过复杂网络拓扑实现可靠的信息传递 它是一组轻量级网络代理的典型应用 与应用一起部署，而无需服务感知

### **网格解读**

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

服务网格主要专注于处理服务间通信的稳定性，在云原生应用场景中负责非常重要的数据通信基础设施。它的实现方式是经过多代演练后逐渐成熟起来的。

目前，它主要以服务治理的sidecar模式存在在互联网场景中。服务网格让业务应用的管理和底层网络数据的管理实现解耦，他们的关联关系变成了 "数据管理(控制管理\[应用层业务])"

**数据层面**&#x20;

* 网络流量数据的控制，触及系统中的每个数据包或请求&#x20;
* 负责服务发现、健康检测、路由分发、负载均衡、身份验证、可观测性等。&#x20;

**控制平面**&#x20;

* 为数据层面提供策略和配置，不接触任何数据包或请求，以api接口的方式被程序员调用使用。&#x20;
* &#x20;负责发现策略、调度策略、断路策略、流量转移机制等

#### 服务网格的作用

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

当我们在K8s集群中创建出来的应用，那么这些应用所面对的流量主要有两类：

**集群内部的服务间流量** 和 **集群内外的用户和服务间流量**，我们可以借助于所谓的网络策略来对这些流量进行控制。但是，我们无法保证这些集群内部服务间的通信安全；一旦遇到服务内部的相关调用关系的时候，我们无法跟踪；当我们需要实现更加细粒度或者更加高级的定向流量的时候，k8s的网络策略受网络解决方案的限制较多。 而一旦有了服务网格，这些k8s集群内部的应用流量，我们就可以进行更加精细化的处理控制了。



CNCF对云原生领域的服务网格产品进行了相关的梳理，但是根据我的了解，虽然服务网格实现的产品解决方案很多，但是，他们在功能上面有着十分的类同，这些功能主要体现在：2022年4月，Google宣布把Istio捐赠给CNCF。

流量管理：这是服务网格的根本所在 主要包括四层|七层网络的转发、动态路由功能、流量质量(超时|重试等)、流量策略(访问控制|流量速率等)、等 安全管理，这是服务网格的高级功能 主要包括加密通信(TLS|mTLS)、认证授权(认证、审计、授权等)、等 观测管理，这是服务网格最为人所知的高阶功能 主要包括监控指标、服务跟踪、流量控制、多集群等。这部分功能是任何分布式平台都需要重点关注的内容。
