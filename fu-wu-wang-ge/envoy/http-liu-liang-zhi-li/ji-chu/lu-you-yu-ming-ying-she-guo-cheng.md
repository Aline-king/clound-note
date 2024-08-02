# 路由域名映射过程

## Envoy HTTP路由域名映射过程 <a href="#qi-envoyhttp-lu-you-yu-ming-ying-she-guo-cheng-234" id="qi-envoyhttp-lu-you-yu-ming-ying-she-guo-cheng-234"></a>

在Envoy配置中，虚拟主机的域名匹配是处理入站HTTP请求的关键环节。这一机制允许Envoy根据请求的 `Host`头部将不同的请求路由到不同的后端服务。下面是对域名搜索顺序的详细解释：

### 域名搜索顺序 <a href="#id-71-yu-ming-sou-suo-shun-xu-236" id="id-71-yu-ming-sou-suo-shun-xu-236"></a>

当HTTP请求到达Envoy代理时，Envoy会根据请求中的 `Host`头部或HTTP/2的 `:authority`头部中的值来确定该请求应该由哪个虚拟主机处理。Envoy按以下顺序搜索与 `Host`头部匹配的虚拟主机的域名：

1. **精确匹配** (`Exact domain names`):
   * 这是最直接的匹配方式，Envoy会检查配置中是否有与 `Host`头部完全相同的域名。
   * 例如，如果 `Host`头为 `www.kubex.io`，Envoy会查找配置中是否有一个虚拟主机定义了 `www.kubex.io`为其域名。
2. **左前缀匹配** (`Prefix domain wildcards`):
   * 如果没有找到精确匹配，Envoy会查找是否有域名前缀匹配。这种匹配使用星号 `*`作为前缀通配符。
   * 例如，如果 `Host`头为 `blog.kubex.io`，Envoy会匹配诸如 `*.kubex.io`的域名。
3. **右后缀匹配** (`Suffix domain wildcards`):
   * 如果前两种方式都没有找到匹配项，Envoy会尝试后缀通配符匹配，这也使用星号 `*`，但放在域名的末尾。
   * 例如，如果 `Host`头为 `kubex.com`，Envoy会匹配如 `kubex.*`的配置。
4. **通配符“\*”** (`Special wildcard *`):
   * 如果以上所有匹配方式都未成功，Envoy将使用特殊的通配符 `*`，这意味着匹配任何域名。
   * 这通常用于默认的虚拟主机配置，以确保所有未明确匹配到特定配置的请求都能被处理。

### 7.2 终止搜索 <a href="#id-72-zhong-zhi-sou-suo-239" id="id-72-zhong-zhi-sou-suo-239"></a>

在Envoy的虚拟主机匹配逻辑中，一旦找到匹配的域名，搜索过程即终止。这意味着，匹配过程是按优先级进行的，一旦匹配成功，不会继续考虑其他可能的匹配。

**域名示例**

假设配置了以下虚拟主机的域名：

* `www.kubex.io` (精确匹配)
* `*.kubex.io` (左前缀匹配)
* `kubex.*` (右后缀匹配)
* `*` (通配符)

对于请求 `Host`为 `api.kubex.io`，Envoy会按照上述顺序首先尝试精确匹配，然后是左前缀匹配，这将成功匹配到 `*.kubex.io`。

这种灵活而详尽的匹配机制使得Envoy能够有效地管理大规模和复杂的微服务架构中的网络流量。



{% tabs %}
{% tab title="路由配置示例一" %}
```powershell
virtual_hosts:
  - name: vh_001
    domains: ["kubex.io", "*.kubex.io", "kubex.*"]
    routes:
    - match:
        path: "/service/blue"
      route:
        cluster: blue
    - match:
        safe_regex:
          google_re2: {}
          regex: "^/service/.*blue$"
        redirect:
          path_redirect: "/service/blue"
    - match:
        prefix: "/service/yellow"
        direct_response:
          status: 200
          body:
            inline_string: "This page will be provided soon later.\n"
    - match:
        prefix: "/"
      route:
        cluster: red
  - name: vh_002
    domains: ["*"]
    routes:
    - match:
        prefix: "/"
      route:
        cluster: gray
```

以上路由配置示例在Envoy的上下文中演示了虚拟主机的配置及多种路由匹配机制。这里的配置包括两个虚拟主机（`vh_001`和 `vh_002`），每个主机都有针对特定类型请求的路由规则。下面是对这些路由规则的详细解释：



**虚拟主机 `vh_001`**

* **域名**：`kubex.io`, `*.kubex.io`, `kubex.*` 这个虚拟主机处理从 `kubex.io`及其所有子域和不同顶级域的请求。
* **路由规则**：
  1. **精确路径匹配**：
     * **路径**：`/service/blue`
     * **动作**：将请求路由到名为 `blue`的集群。
  2. **正则表达式匹配**：
     * **正则表达式**：`^/service/.*blue$`
     * **动作**：将请求重定向到 `/service/blue`。这可以用于规范化URL格式或处理旧URL的迁移。
  3. **前缀匹配**：
     * **前缀**：`/service/yellow`
     * **动作**：返回直接响应，状态码为200，响应体为"This page will be provided soon later.\n"。这种类型的路由常用于站点维护或未完成页面的提示。
  4. **根路径前缀匹配**：
     * **前缀**：`/`
     * **动作**：将根路径及其子路径的请求路由到名为 `red`的集群。

**虚拟主机 `vh_002`**

* **域名**：`*` 这个虚拟主机处理所有未由 `vh_001`捕获的域名的请求。
* **路由规则**：
  1. **前缀匹配**：
     * **前缀**：`/`
     * **动作**：将所有请求路由到名为 `gray`的集群。

这个配置演示了Envoy在处理HTTP请求时的灵活性：

* **匹配类型多样**：支持基于精确路径、正则表达式、前缀的匹配，允许细致地控制请求路由。
* **多种路由动作**：可以执行请求的路由、重定向、直接响应等不同动作。
* **域名通配符**：通过使用通配符，虚拟主机可以灵活处理来自多个域名的请求。

这种配置使得Envoy非常适合作为微服务架构中的边缘或内部代理，能够根据复杂的规则集处理入站和出站流量。
{% endtab %}

{% tab title="路由配置示例二" %}
这个YAML配置定义了一个Envoy代理中的虚拟主机配置。虚拟主机包含了几个路由规则，这些规则基于请求的特定属性（如头部信息和查询参数）将流量路由到不同的后端服务。

```yaml
virtual_hosts:
  - name: vh_001
    domains: ["*"] # 该虚拟主机处理的域名列表。这里的 "*"表示这个虚拟主机将处理所有域名的请求。
    routes:
      - match:
          prefix: "/"
          headers:
            - name: X-Canary
              exact_match: "true"
        route:
          cluster: demoappv12
      - match:
          prefix: "/"
          query_parameters:
            - name: "username"
              string_match:
                prefix: "vip_"
        route:
          cluster: demoappv11
      - match:
          prefix: "/"
        route:
          cluster: demoappv10
```

* **路由规则 (Routes)**:
  * 每个 `match`条件定义了一组规则，根据请求的属性决定使用哪个路由。所有的路由都从根路径 `"/"`开始匹配，但基于其他属性（如头部或查询参数）来进一步细化匹配规则。
  * **第一条路由规则**:
    * **头部匹配**: 查找头部 `X-Canary`且值必须精确为 `"true"`。
    * **路由目标**: 如果匹配成功，请求将被路由到名为 `demoappv12`的集群。
  * **第二条路由规则**:
    * **查询参数匹配**: 查找查询参数 `username`，其值必须以 `"vip_"`为前缀。
    * **路由目标**: 如果匹配成功，请求将被路由到名为 `demoappv11`的集群。
  * **第三条路由规则**:
    * **基本匹配**: 仅基于路径 `"/"`，没有额外条件。
    * **路由目标**: 如果前两条规则都未匹配，请求默认路由到名为 `demoappv10`的集群。

这个配置文件示例显示了Envoy如何利用HTTP头部和查询参数进行条件匹配，以决定将请求路由到何种后端服务。这种灵活的路由机制使得Envoy非常适合动态的、基于条件的路由场景，如进行A/B测试、灰度发布或处理多租户应用。
{% endtab %}
{% endtabs %}

**两个YAML示例文件展示了Envoy路由配置中不同的焦点和用途：**

1. **第一个示例**：
   * **展示了多种match的基本匹配机制**：这包括精确路径匹配、正则表达式匹配、前缀匹配以及直接响应的配置。这些匹配机制使得路由配置能够处理各种类型的HTTP请求，根据路径或路径模式来决定如何路由请求或是提供直接响应。
   * **不同的路由方式**：示例中展示了将请求路由到指定集群、执行HTTP重定向、以及返回直接响应等操作。这种多样化的路由行为展示了Envoy的强大灵活性在处理不同路由需求上的能力。
2. **第二个示例**：
   * **侧重于基于HTTP头部和查询参数的匹配**：此示例中的路由规则特别关注于使用请求中的特定HTTP头部（如 `X-Canary`）和查询参数（如用户名前缀 `vip_`）来决定路由目标。这表明了配置的目的是根据请求的具体内容来动态地决定路由，这在进行A/B测试、用户分群等场景中非常有用。
   * **动态路由决策**：通过检查请求的特定属性来路由到不同的后端服务，强调了根据实际请求动态选择服务的能力，而不是仅仅基于请求路径。

总的来说，第一个示例强调了Envoy如何处理多种路由情况，而第二个示例则显示了基于更细致条件（如HTTP头部和查询参数）的路由决策，两者都体现了Envoy在现代云基础设施中的应用灵活性。
