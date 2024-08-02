# 子集负载均衡器Subset

## 一、子集负载均衡Subset介绍 <a href="#yi-zi-ji-fu-zai-jun-heng-subset-jie-shao-1" id="yi-zi-ji-fu-zai-jun-heng-subset-jie-shao-1"></a>

在Envoy中，子集负载均衡（Subset Load Balancing）是一种高级负载均衡策略，通过将上游集群分成多个子集，根据请求的属性（如元数据标签）将流量路由到特定子集中的节点。

子集负载均衡有助于实现更细粒度的流量控制，满足多租户、分区、多版本等复杂场景的需求。

常见的应用场景包括：

1. **多租户架构**：将不同租户的流量路由到各自的服务实例。
2. **蓝绿部署**：在蓝绿部署场景中，将流量路由到新旧版本的服务实例，方便版本切换和测试。
3. **A/B测试**：支持A/B测试，将流量路由到不同版本的服务实例进行实验。
4. **地域分布**：根据地理位置或其他属性，将流量路由到最近的服务实例，提高性能和用户体验。

## 二、子集负载均衡Subset 配置示例 <a href="#er-zi-ji-fu-zai-jun-heng-subset-pei-zhi-shi-li-5" id="er-zi-ji-fu-zai-jun-heng-subset-pei-zhi-shi-li-5"></a>

```yaml
static_resources:
  clusters:
  - name: subset_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: subset_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service-v1.example.com
                port_value: 80
          metadata:
            filter_metadata:
              envoy.lb:
                version: v1
        - endpoint:
            address:
              socket_address:
                address: service-v2.example.com
                port_value: 80
          metadata:
            filter_metadata:
              envoy.lb:
                version: v2
    lb_subset_config:
      fallback_policy: NO_FALLBACK
      subset_selectors:
      - keys:
        - version
```

**配置项解释**

* **lb\_policy**：指定基本的负载均衡策略，这里使用 `ROUND_ROBIN`。
* **lb\_endpoints**：定义上游服务的节点。
* **metadata**：在每个节点的元数据中添加负载均衡相关的信息。
* **filter\_metadata**：定义与子集负载均衡相关的元数据标签（例如 `version`）。
* **lb\_subset\_config**：配置子集负载均衡的具体参数。
  * **fallback\_policy**：指定当没有匹配的子集时的回退策略，这里设置为 `NO_FALLBACK`，表示没有匹配子集时不进行回退。
  * **subset\_selectors**：定义子集选择器，根据元数据标签创建子集。

**验证子集负载均衡**

通过配置子集负载均衡，可以将请求路由到不同版本的服务实例。以下是如何验证子集负载均衡的效果：

1. **启动Envoy**：使用上述配置文件启动Envoy。
2. **发送带有元数据的请求**：为了测试子集负载均衡，需要发送带有特定元数据的请求。

例如，可以通过HTTP头或查询参数传递元数据：

```sh
curl -H "x-envoy-upstream-rq-version: v1" http://localhost:10000/your_service_endpoint
curl -H "x-envoy-upstream-rq-version: v2" http://localhost:10000/your_service_endpoint
```

根据请求中的元数据标签，Envoy会将流量路由到对应版本的服务实例。

**监控和调试**

Envoy提供了丰富的监控指标，可以通过/admin接口查看子集负载均衡的状态和统计信息。例如：

* **`cluster.subset_service.lb_subsets_active`**：当前活跃的子集数。
* **`cluster.subset_service.lb_subsets_created`**：创建的子集数。
* **`cluster.subset_service.lb_subsets_removed`**：移除的子集数。

这些统计数据可以通过访问Envoy的admin接口获取，例如 `http://localhost:9901/stats`。

## 四、子集负载均衡器Subset作用 <a href="#si-zi-ji-fu-zai-jun-heng-qi-subset-zuo-yong-19" id="si-zi-ji-fu-zai-jun-heng-qi-subset-zuo-yong-19"></a>

子集负载均衡是一种网络技术，它允许你在一个大的服务集群中对流量进行更精细的控制和分配。这种技术通过将集群中的服务器划分为多个子集，然后根据特定的规则将流量定向到这些子集，来实现流量的有效管理。这里是如何操作的：

1. **上游主机的元数据添加与子集划分**：
   * 在集群的每个服务器（上游主机）上添加元数据，这些元数据是一些键值标签，用来描述服务器的特征或属性。
   * 通过这些元数据，使用子集选择器可以将具有相似特征的服务器划分为一个子集。比如，你可以根据服务器的地理位置、服务器类型或者提供的服务类型来划分子集。
2. **基于元数据的路由配置**：
   * 在路由配置中设定规则，指定负载均衡器在选择服务器时，必须选择具有匹配特定元数据的上游主机。这意味着只有那些符合特定特征的服务器才会被选择来处理特定的流量。
   * 这样可以确保流量根据预定的策略被正确分发到合适的子集。
3. **子集间的负载均衡策略**：
   * 子集内的负载均衡则按照集群定义的策略进行，这些策略可能包括轮询、最少连接数等。
4. **回退策略**：
   * 如果路由配置没有指定元数据或者没有找到匹配的子集，负载均衡器将采用回退策略处理请求。回退策略包括：
     * **NO\_FALLBACK**：如果没有匹配的子集，请求会失败，就像集群中没有任何可用主机一样。
     * **ANY\_ENDPOINT**：不考虑主机的元数据，将请求在所有可用主机之间进行调度。
     * **DEFAULT SUBSET**：将请求路由到一个预定义的默认子集。

这种细粒度的流量分配方法可以提高服务的效率和可靠性，允许更灵活地处理不同类型的流量需求。

## 五、配置子集负载均衡器方法 <a href="#wu-pei-zhi-zi-ji-fu-zai-jun-heng-qi-fang-fa-23" id="wu-pei-zhi-zi-ji-fu-zai-jun-heng-qi-fang-fa-23"></a>

想要理解“子集必须预定义方可由子集负载均衡器在调度时使用”的内容，我们可以借助一个简单的例子来说明：

想象你是一家大型连锁咖啡店的运营经理，这家连锁店在全国有数百家分店。为了更高效地管理顾客订单和提供服务，你决定根据每家店的位置（比如城市）和类型（比如是否提供24小时服务）来分组管理。

1. **定义元数据（键值数据）**：
   * 就像你需要对每家咖啡店进行分类一样，Envoy在管理服务器（主机）时，也需要给每个主机定义一些键值对的元数据。例如，元数据可以是服务器的版本号（如1.0）、所在的阶段（如生产环境）等。
   * 这些元数据需要定义在特定的地方，即在Envoy的“envoy.lb”过滤器下。这样Envoy才能正确地识别和使用这些元数据。
2. **支持主机元数据的方式**：
   * 主机元数据只在通过EDS（Endpoint Discovery Service）发现的端点或者通过“load assignment”字段定义的端点中支持。
   * 这意味着，如果你想让Envoy能够使用这些定义好的元数据，你必须通过这两种方式之一来配置你的主机信息。

通过这种方式，Envoy可以更精确地管理流量，确保特定类型的请求被发送到具有相应特征的服务器。这类似于你按地理位置和服务类型分组管理咖啡店，以确保顾客在任何地方都能接受到一致的服务质量。

**路由配置和元数据匹配**

此外，使用子集负载均衡还需要在路由配置中设置 `metadata_match`，以确保只有匹配特定元数据的子集可以被路由匹配。

```powershell
routes:
  - name: ...
    match: {...}
    route:
      cluster: ...
      metadata_match:             # 定义路由时使用的元数据匹配条件
        filter_metadata:
          envoy.lb:
            version: "1.0"
            stage: "prod"
      weighted_clusters:          # 使用加权集群定义路由目标
        clusters:
          - name: ...
            weight: ...
            metadata_match: {...} # 同样可以在加权集群中定义元数据匹配
```

这个配置确保流量只被发送到符合特定 `metadata_match`条件的子集。例如，只有那些元数据中包含 `version: "1.0"`和 `stage: "prod"`的主机才会处理匹配这个路由的流量。

通过这样的配置，Envoy 能够确保流量精确地分配到具有特定特征的服务实例，从而提高效率并优化资源利用。这种方式使得基于多维度标签的流量管理成为可能，对于运行在动态和多变环境中的大规模应用尤为重要。

**子集配置示例**

**子集选择器定义**

```powershell
clusters:
- name: webclusters
  lb_policy: ROUND_ROBIN
  lb_subset_config:
    fallback_policy: DEFAULT_SUBSET
    default_subset:
      stage: prod
      version: '1.0'
      type: std
    subset_selectors:
    - keys: [stage,type]
    - keys: [stage,version]
    - keys: [version]
    - keys: [xlarge,version]
```

* **名称**：集群名为 "webclusters"。
* **负载均衡策略**：使用 "ROUND\_ROBIN"，意味着请求将按照轮询方式均匀分配给集群中的节点。
* **子集配置**：
  * **回退策略**：`DEFAULT_SUBSET`。当没有找到匹配的子集时，流量将路由到定义的默认子集。
  * **默认子集**：定义了一个默认的子集，其元数据为 stage 为 "prod"、version 为 "1.0" 和 type 为 "std"。

**子集选择器**

* **选择器**：
  * `keys: [stage, type]`：基于 stage 和 type 的组合创建子集。
  * `keys: [stage, version]`：基于 stage 和 version 的组合创建子集。
  * `keys: [version]`：仅基于 version 创建子集。
  * `keys: [xlarge, version]`：基于 xlarge（特殊标志或属性）和 version 的组合创建子集。

**集群中节点及其拥有的元数据**

| endpoints | stage | version | type   | xlarge |
| --------- | ----- | ------- | ------ | ------ |
| e1        | prod  | 1.0     | std    | true   |
| e2        | prod  | 1.0     | std    |        |
| e3        | prod  | 1.1     | std    |        |
| e4        | prod  | 1.1     | std    |        |
| e5        | prod  | 1.0     | bigmem |        |
| e6        | prod  | 1.1     | bigmem |        |
| e7        | dev   | 1.2-pre | std    |        |

表格列出了集群中的各个节点（Endpoint e1 到 e7）以及它们的元数据属性，包括 stage, version, type, 以及 xlarge。这些元数据用于根据上面子集定义的子集选择器对节点进行分类。

基于提供的配置和节点元数据，可以看到如何根据不同的子集选择器对节点进行分类：

* **`keys: [stage, type]`**
  * 节点 e1, e2, e3, e4（因为它们的 stage 和 type 组合相同）。
* **`keys: [stage, version]`**
  * 节点 e1 和 e2，e5（stage: prod, version: 1.0）。
  * 节点 e3 和 e4，e6（stage: prod, version: 1.1）。
* **`keys: [version]`**
  * 节点 e1, e2, e5（version: 1.0）。
  * 节点 e3, e4, e6（version: 1.1）。
* **`keys: [xlarge, version]`**
  * 节点 e1（因为它标记为 xlarge 并且 version 是 1.0）。

这种子集分类方法使得 Envoy 能够根据配置的策略和具体的元数据高效地路由流量，确保特定类型的流量被正确地送达到具备相应属性的节点。

**路由配置示例**

**功能和用途**

这些路由配置示例展示了如何使用 Envoy 的灵活路由功能来根据请求的特定属性（如请求头或服务实例的元数据）将流量精确地分配给满足特定条件的服务实例。通过这种方式，Envoy 可以根据实际的服务部署情况（如版本、环境阶段等）和请求特性进行动态路由决策。

* **元数据匹配**：确保只有符合特定元数据条件的服务实例会处理这些流量。这对于进行蓝绿部署、A/B 测试或特定功能的逐步推出非常有用。
* **加权路由**：允许不同比例的流量根据权重被分配到不同的服务版本，从而可以灵活地管理流量分配和进行渐进式部署。
* \*\*回退策略：\*\*如果没有子集与给定的元数据匹配，Envoy 将应用配置的回退策略，如默认路由到某个基础版本或其他配置的子集，从而确保服务的可用性和稳定性。这是通过在配置中预设的默认行为或额外的路由规则来实现的。

```powershell
routes:
  - match:
      prefix: "/"
      headers:
        - name: x-custom-version
          exact_match: "pre-release"
    route:
      cluster: webcluster1
      metadata_match:
        filter_metadata:
          envoy.lb:
            version: "1.2-pre"
            stage: "dev"

  - match:
      prefix: "/"
      headers:
        - name: x-hardware-test
          exact_match: "memory"
    route:
      cluster: webcluster1
      metadata_match:
        filter_metadata:
          envoy.lb:
            type: "bigmem"
            stage: "prod"
```

1. **基于特定请求头的路由配置**：
   * 对于所有以 `/` 为前缀的请求，并且请求头 `x-custom-version` 精确匹配 `pre-release`，路由将指向 `webcluster1` 集群。
   * 在路由决策时，将匹配元数据，确保版本为 `1.2-pre` 和阶段为 `dev`。
2. **另一基于请求头的路由配置**：
   * 对于所有以 `/` 为前缀的请求，并且请求头 `x-hardware-test` 精确匹配 `memory`，路由将指向 `webcluster1` 集群。
   * 路由决策时，将匹配元数据以确保类型为 `bigmen` 和阶段为 `prod`。

这个配置展示了 Envoy 的强大能力，能够根据不同的请求头和服务实例的特定元数据进行精确的路由控制。这样的配置对于进行特定功能测试或针对特定用户群体部署功能非常有用。

```powershell
- match:
    prefix: "/"
  route:
    weighted_clusters:
      clusters:
        - name: webcluster1
          weight: 90
          metadata_match:
            filter_metadata:
              envoy.lb:
                version: "1.0"
        - name: webcluster1
          weight: 10
          metadata_match:
            filter_metadata:
              envoy.lb:
                version: "1.1"
                stage: "prod"
```

1. **匹配规则**：
   * 所有以 `/` 开头的请求都将被这个规则捕获。
2. **加权集群**：
   * `webcluster1` 集群配置了两个权重不同的版本。
   * 90% 的流量将路由到元数据中指定 `version: "1.0"` 的实例。
   * 另外 10% 的流量将路由到同时满足 `version: "1.1"` 和 `stage: "prod"` 条件的实例。

这个配置示例演示了 Envoy 如何使用元数据和权重来精确控制流量分配，使得可以根据不同的服务版本进行有效的流量管理。
