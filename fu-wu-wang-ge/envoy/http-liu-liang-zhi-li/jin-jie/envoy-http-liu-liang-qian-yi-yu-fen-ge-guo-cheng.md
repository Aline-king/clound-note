# Envoy HTTP流量迁移与分割过程

Envoy 流量迁移和分割的机制中的这些策略在新版本上线和进行不同测试时非常重要，以确保产品的稳定性和用户的接受度。

1. **新版本上线**：
   * **描述**：为了确保产品的稳定性和用户的接受程度，在新老版本同时在线的情况下，将流量按需分派至不同的版本。这可以通过以下方式实现：
     * **蓝绿发布**：同时维护两个生产环境（蓝和绿），一个处理当前流量，另一个用于新版本，最终在新版本准备就绪后切换流量。
     * **A/B测试**：将流量分成两部分，分别发送到不同版本，以测试不同版本的效果。
     * **金丝雀发布**：将少量流量引导到新版本，观察一段时间后再决定是否扩大范围。
2. **HTTP路由器的流量分割和迁移**：
   * **描述**：HTTP路由器能够将流量按比例分配到不同的上游集群中的虚拟主机，从而实现以下两种常见用例：
     * **版本升级**：在路由时将流量逐渐从一个集群迁移至另一个集群，实现灰度发布。这可以通过在路由中定义流量相关的百分比来实现。
     * **A/B测试或多变量测试**：同时测试多个相同服务的不同版本，将流量在这些版本之间分配。通过在路由中使用基于权重的集群路由来完成。
3. **基于内容的流量管理**：
   * **描述**：结合指定的标头，可以完成基于内容的流量管理。这意味着可以根据请求内容（如标头信息）将流量分配到不同的服务版本。