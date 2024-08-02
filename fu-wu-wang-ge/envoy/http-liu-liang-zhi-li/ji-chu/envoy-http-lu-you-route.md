# Envoy HTTP路由（Route）

路由是指根据请求的路径、方法、头信息等将请求转发到特定的后端服务。它定义了当请求匹配某个虚拟主机后，如何处理这个请求。

### 2.1 路由配置 <a href="#id-21-lu-you-pei-zhi-36" id="id-21-lu-you-pei-zhi-36"></a>

在虚拟主机下，你可以配置一个或多个路由，每个路由包括匹配条件（match）和处理方式（route）。



{% tabs %}
{% tab title="简单配置示例" %}
```yaml
routes:
- match:
    prefix: "/"
  route:
    cluster: service_cluster
- match:
    prefix: "/api"
  route:
    cluster: api_service_cluster
```

在上述配置中，有两个路由规则：

1. 匹配所有路径（`/`），请求将被转发到 `service_cluster`。
2. 匹配路径以 `/api` 开头的请求，将被转发到 `api_service_cluster`。
{% endtab %}

{% tab title="详细路由配置示例" %}
下面是一个更详细的路由配置示例，包含更多的匹配条件和处理选项：

```powershell
routes:
- match:
    prefix: "/"
    headers:
    - name: "x-custom-header"
      exact_match: "value"
  route:
    cluster: service_cluster
    timeout: 0.5s
    retry_policy:
      retry_on: "5xx"
      num_retries: 3
      per_try_timeout: 0.2s
```

在这个示例中，路由规则匹配路径为 `/` 且请求头 `x-custom-header` 为 `value` 的请求。匹配后，将请求转发到 `service_cluster`，并配置了超时时间和重试策略。
{% endtab %}
{% endtabs %}

**路由配置详细解释**

* `match`：定义请求匹配条件。可以是路径前缀（prefix），完全路径（path），正则表达式（regex）等。
* `route`：定义匹配到请求后的处理方式。主要指定转发的后端集群（cluster），还可以定义超时时间、重试策略、负载均衡等。

### 2.2 路由配置步骤 <a href="#id-22-lu-you-pei-zhi-bu-zhou-48" id="id-22-lu-you-pei-zhi-bu-zhou-48"></a>

1. **定义虚拟主机**：在虚拟主机中定义多个域名。
2. **添加路由规则**：在虚拟主机中添加一个或多个路由规则。
3. **设置匹配条件**：在路由规则中设置路径、请求头等匹配条件。
4. **配置转发选项**：在路由规则中配置转发的后端集群、超时、重试等选项。

通过以上步骤，可以灵活地根据请求的不同属性配置Envoy的路由策略，从而实现流量的精确控制和管理。

### 2.3 路由匹配(match) <a href="#id-23-lu-you-pi-pei-match51" id="id-23-lu-you-pi-pei-match51"></a>

在Envoy中，路由匹配（match）用于定义如何筛选进入的流量并匹配特定的路由规则。路由匹配可以根据请求路径、请求头、查询参数等进行配置。主要包括基础匹配和高级匹配两类。

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



{% tabs %}
{% tab title="匹配" %}
基础匹配

* **prefix**: 按路径前缀匹配。
* **path**: 按完全路径匹配。
* **safe\_regex**: 按正则表达式匹配。

高级匹配

* **headers**: 按请求头匹配。
* **query\_parameters**: 按查询参数匹配。

**配置示例**

* 第一条路由规则匹配路径前缀为 `/api` 且请求头 `x-custom-header` 为 `custom_value` 且查询参数 `user` 为 `admin` 的请求。
* 第二条路由规则使用正则表达式匹配路径以 `/user/` 开头的请求。

```yaml
routes:
- match:
    prefix: "/api"
    headers:
    - name: "x-custom-header"
      exact_match: "custom_value"
    query_parameters:
    - name: "user"
      string_match:
        exact: "admin"
  route:
    cluster: api_service_cluster
- match:
    safe_regex:
      google_re2: {}
      regex: "^/user/.*"
  route:
    cluster: user_service_cluster
```
{% endtab %}

{% tab title="路由" %}


#### <mark style="color:blue;">重定向 (redirect)</mark> <a href="#id-241-zhong-ding-xiang-redirect64" id="id-241-zhong-ding-xiang-redirect64"></a>

重定向规则用于将请求重定向到另一个URL。可以配置301或302重定向。

**配置示例**

```yaml
routes:
- match:
    prefix: "/old-path"
  redirect:
    path_redirect: "/new-path"
    response_code: 301
```

#### 直接响应 (direct\_response) <a href="#id-242-zhi-jie-xiang-ying-directresponse68" id="id-242-zhi-jie-xiang-ying-directresponse68"></a>

直接响应规则用于由Envoy直接响应请求，而不转发给其他上游服务处理。可以配置静态响应内容和状态码。

**配置示例**

```yaml
routes:
- match:
    prefix: "/healthcheck"
  direct_response:
    status: 200
    body:
      inline_string: "Healthy"
```

在这个示例中，请求匹配路径前缀为 `/healthcheck` 的请求将由Envoy直接响应，返回状态码200和响应体"Healthy"。
{% endtab %}
{% endtabs %}

通过这些匹配条件，可以将进入的流量基于报文的某些特征筛选出来，进行匹配或流量筛选，从而将其交换或路由到某个特定的路径上去。

### 2.5 Envoy匹配路由工作过程 <a href="#id-25envoy-pi-pei-lu-you-gong-zuo-guo-cheng-73" id="id-25envoy-pi-pei-lu-you-gong-zuo-guo-cheng-73"></a>

Envoy作为一个高度灵活的代理，通过详细的配置实现对入站HTTP请求的精确路由。这一过程主要涉及三个关键步骤，如下所述：

{% tabs %}
{% tab title="匹配虚拟主机" %}
* **检测HTTP请求的 `Host`头部或 `:authority`**：当HTTP请求到达Envoy时，Envoy首先检查请求中的 `Host`头部或HTTP/2的 `:authority`头部。这些头部包含了发起请求的域名信息。
* **与虚拟主机域名进行匹配**：Envoy将这个域名与路由配置中定义的所有虚拟主机的域名进行匹配。每个虚拟主机可以配置一个或多个域名。Envoy选择第一个域名匹配的虚拟主机来处理该请求。
{% endtab %}

{% tab title="匹配路由条目" %}
* **按顺序检查路由条目**：在找到合适的虚拟主机后，Envoy将按照定义的顺序遍历这个虚拟主机中的每个路由条目。每个路由条目都有其自己的匹配条件，这些条件可以基于请求的路径、HTTP方法、请求头等信息。
* **第一个匹配的路由条目**：Envoy使用“短路”逻辑来处理匹配，即一旦找到第一个符合条件的路由条目，就停止进一步的检查，并按照该路由条目定义的指令处理请求。这可能包括将请求路由到一个后端集群、执行重定向或直接返回一个响应。
{% endtab %}

{% tab title="虚拟集群的匹配（如果配置）" %}
* **检查虚拟集群**：如果在虚拟主机配置中定义了虚拟集群，Envoy还会进行额外的匹配检查。虚拟集群主要用于流量的细粒度统计，不影响路由决策。
* **虚拟集群匹配**：Envoy会按顺序检查每个定义的虚拟集群的匹配条件，直到找到第一个匹配的虚拟集群。匹配成功的虚拟集群会被用来收集该请求的统计数据。
{% endtab %}
{% endtabs %}

这个路由匹配过程确保Envoy可以处理各种复杂的路由场景，并且通过优先匹配的逻辑提供高效的请求处理。

当然，这里有一个更详细的YAML配置示例，说明了一个Envoy监听器配置。该配置包含了路由处理和虚拟主机的详细信息。

```yaml
listeners:
  - name: main_listener
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 80
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
                    domains: ["local.example.com"] # 虚拟主机的域名，用于匹配请求中的Host头部
                    routes:
                      - name: home_page
                        match:
                          prefix: "/" # 匹配所有以"/"开始的请求路径
                        route:
                          cluster: home_service # 将匹配的请求路由到名为home_service的集群
                      - name: api
                        match:
                          prefix: "/api" # 匹配所有以"/api"开始的请求路径
                        route:
                          cluster: api_cluster # 将匹配的请求路由到名为api_cluster的集群
                    virtual_clusters:
                      - name: api_stats
                        pattern: "^/api/.*$" # 使用正则表达式匹配所有API请求
                        method: GET # 只统计GET方法的请求
```

**解释**

* **listeners**: 这是顶层元素，包含所有的监听器配置。
  * **name**: 监听器的名称，这里是"main\_listener"。
  * **address**:
    * **socket\_address**: 指定监听器的网络地址。
      * **address**: 监听的IP地址，这里是"0.0.0.0"表示监听所有接口。
      * **port\_value**: 监听的端口号，这里是80，HTTP默认端口。
* **filter\_chains**:
  * **filters**:
    * **name**: 使用的过滤器名称，这里是 `envoy.filters.network.http_connection_manager`，它管理HTTP连接。
    * **typed\_config**: 过滤器的详细配置。
      * **@type**: 配置类型，指定为HTTP连接管理器的v3版本。
      * **stat\_prefix**: 用于统计的前缀，这里是"ingress\_http"。
      * **codec\_type**: 解码类型，自动选择适当的HTTP编解码器。
* **route\_config**: 路由配置的详细信息。
  * **name**: 路由配置的名称，这里是"local\_route"。
  * **virtual\_hosts**: 包含一组虚拟主机配置。
    * **name**: 虚拟主机的名称，这里是"local\_service"。
    * **domains**: 这个虚拟主机处理的域名列表，这里只有"local.example.com"。
    * **routes**: 定义如何处理匹配到此虚拟主机的请求。
      * **name**: 路由的名称，例如"home\_page"和"api"。
      * **match**: 定义匹配条件，如前缀"/"和"/api"。
      * **route**: 定义路由目标，指定集群如"home\_service"和"api\_cluster"。
* **virtual\_clusters**: 定义用于统计信息的虚拟集群。
  * **name**: 虚拟集群的名称，例如"api\_stats"。
  * **pattern**: 匹配模式，这里使用正则表达式匹配所有"/api/.\*$"路径。
  * **method**: 指定要统计的HTTP方法，这里是"GET"。

这个配置示例展示了如何在Envoy中设置监听器来处理不同类型的HTTP请求，并将它们路由到不同的后端服务，同时也如何收集特定请求的统计信息。
