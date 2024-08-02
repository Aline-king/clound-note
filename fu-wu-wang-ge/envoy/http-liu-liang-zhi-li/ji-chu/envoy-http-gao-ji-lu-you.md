# Envoy HTTP高级路由



## 高级路由 <a href="#liu-envoyhttp-gao-ji-lu-you-182" id="liu-envoyhttp-gao-ji-lu-you-182"></a>

Envoy的HTTP高级路由功能提供了强大的工具和配置选项，以便灵活地管理和处理HTTP请求。以下是Envoy的HTTP高级路由的主要内容和功能：



{% tabs %}
{% tab title="路由匹配" %}
### （Route Matching） <a href="#id-61-lu-you-pi-pei-routematching184" id="id-61-lu-you-pi-pei-routematching184"></a>

路由匹配是Envoy路由的核心部分，通过配置不同的匹配条件，将请求路由到合适的上游服务。匹配条件包括：

* **前缀匹配（prefix match）**：匹配请求路径的前缀。
* **完全路径匹配（path match）**：匹配完整的请求路径。
* **正则表达式匹配（regex match）**：使用正则表达式匹配请求路径。
* **请求头匹配（header match）**：基于请求头的值进行匹配。
* **查询参数匹配（query parameter match）**：基于查询参数的值进行匹配。
{% endtab %}

{% tab title="路由目标" %}
定义匹配请求后的目标服务和行为，包括：

* **集群（cluster）**：将请求转发到特定的上游集群。
* **集群权重（weighted clusters）**：将请求按权重分配到多个集群，实现流量分配和负载均衡。
* **直连响应（direct response）**：直接由Envoy响应请求，而不是转发到上游服务。
* **重定向（redirect）**：将请求重定向到另一个URL。
{% endtab %}

{% tab title="路由策略" %}
通过路由策略（Route Policies）对请求进行进一步的处理和控制：

* **请求/响应修改**：修改请求和响应的头信息、路径等。
* **重试策略（retry policy）**：定义请求失败后的重试行为，包括重试次数、条件等。
* **超时策略（timeout policy）**：定义请求的超时时间，包括总超时、空闲超时等。
* **流量镜像（traffic mirroring）**：将请求复制并发送到其他服务进行调试和测试。
{% endtab %}

{% tab title="路由级别的特性和操作" %}
* **健康检查（health checks）**：确保请求仅被路由到健康的上游服务实例。
* **负载均衡策略（load balancing policy）**：定义请求在上游集群中的负载均衡行为，例如轮询、最少连接、随机等。
* **请求限速（rate limiting）**：对请求进行限速控制，防止过载。
* **服务质量（QoS）**：配置不同的优先级和延迟管理策略。
{% endtab %}

{% tab title="高级匹配条件" %}
* **复合匹配（composite matching）**：组合多种匹配条件进行复杂匹配，例如同时基于路径、请求头和查询参数进行匹配。
* **谓词匹配（predicate matching）**：使用自定义条件进行匹配，例如基于请求的某些特征进行高级匹配。
{% endtab %}

{% tab title="动态路由" %}
* **xDS API**：通过xDS API动态更新路由配置，包括CDS（Cluster Discovery Service）、EDS（Endpoint Discovery Service）、LDS（Listener Discovery Service）和RDS（Route Discovery Service）。
* **实时配置更新**：支持路由配置的实时更新，确保系统的高灵活性和可用性。
{% endtab %}
{% endtabs %}
