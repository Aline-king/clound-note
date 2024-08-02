# 服务发现

## 服务发现机制 <a href="#si-fu-wu-fa-xian-ji-zhi-56" id="si-fu-wu-fa-xian-ji-zhi-56"></a>

在Envoy中，服务发现机制是指Envoy如何找到和维护上游服务实例的信息。服务发现机制允许Envoy自动发现和更新上游服务的地址信息，以便进行负载均衡和路由。

Envoy支持多种服务发现机制，包括<mark style="color:orange;">**静态配置**</mark>、<mark style="color:yellow;">**DNS服务发现**</mark>、EDS（Endpoint Discovery Service）等。以下是几种主要的服务发现机制及其工作原理：

## 1.1 静态服务发现 <a href="#id-41-jing-tai-fu-wu-fa-xian-58" id="id-41-jing-tai-fu-wu-fa-xian-58"></a>

静态服务发现是通过在Envoy配置文件中直接指定上游服务的地址信息。这种方式适用于上游服务的地址相对固定的情况。

```yaml
static_resources:
  clusters:
  - name: static_service_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: static_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 10.0.0.1
                port_value: 80
        - endpoint:
            address:
              socket_address:
                address: 10.0.0.2
                port_value: 80
```

## 1.2 DNS服务发现 <a href="#id-42dns-fu-wu-fa-xian-61" id="id-42dns-fu-wu-fa-xian-61"></a>

DNS服务发现通过DNS解析来获取上游服务的地址信息，适用于动态变化的服务环境。Envoy可以定期刷新DNS记录以更新上游服务的地址。

```yaml
static_resources:
  clusters:
  - name: dns_service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: dns_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service.example.com
                port_value: 80
    dns_refresh_rate: 5s
    dns_lookup_family: V4_ONLY
```

在Envoy中，`STRICT_DNS`和 `LOGICAL_DNS`是两种不同的DNS服务发现类型，它们的主要区别在于如何解析和使用DNS记录来发现和管理上游服务的地址。

### 1.2.1 STRICT\_DNS（严格DNS） <a href="#id-421strictdns-yan-ge-dns65" id="id-421strictdns-yan-ge-dns65"></a>

`STRICT_DNS`模式用于通过DNS解析来动态发现上游服务的地址。

这种模式下，Envoy会定期解析DNS记录，并将解析到的所有IP地址视为上游服务的成员。`STRICT_DNS`适用于需要高可用性和负载均衡的场景，因为它能够动态更新上游节点列表。

**特点：**

* **动态解析**：Envoy定期刷新DNS记录，以确保上游服务地址的最新状态。
* **多地址支持**：如果DNS解析返回多个IP地址，Envoy会将这些地址全部添加到上游服务节点列表中。
* **负载均衡**：在负载均衡过程中，所有解析到的IP地址都会参与负载均衡。

**示例配置：**

```yaml
static_resources:
  clusters:
  - name: strict_dns_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: strict_dns_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service.example.com
                port_value: 80
    dns_refresh_rate: 5s
    dns_lookup_family: V4_ONLY
```

在这个示例中，`service.example.com`会每5秒解析一次，以获取最新的IP地址列表。

### 1.2.2 LOGICAL\_DNS（逻辑DNS） <a href="#id-422logicaldns-luo-ji-dns72" id="id-422logicaldns-luo-ji-dns72"></a>

`LOGICAL_DNS`模式与 `STRICT_DNS`模式类似，但有一个关键区别：`LOGICAL_DNS`模式只使用第一次DNS解析的结果来建立一个持久的连接。后续的DNS解析不会影响已经建立的连接。这意味着，新的DNS解析结果不会动态更新现有的上游服务地址列表。

**特点：**

* **初始解析**：Envoy在启动时或配置更新时解析DNS记录，以获取初始的上游服务地址。
* **持久连接**：使用初始解析的地址建立连接，后续的DNS解析不会影响现有连接。
* **适用场景**：适用于上游服务地址相对稳定，不需要频繁动态更新的场景。

**示例配置：**

```yaml
static_resources:
  clusters:
  - name: logical_dns_cluster
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: logical_dns_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service.example.com
                port_value: 80
    dns_lookup_family: V4_ONLY
```

在这个示例中，`service.example.com`会在Envoy启动或配置更新时解析一次，并使用解析到的地址建立持久连接。

### **主要区别总结**

<table><thead><tr><th width="147"></th><th>STRICT_DNS</th><th>LOGICAL_DNS</th></tr></thead><tbody><tr><td><strong>DNS解析频率</strong></td><td>定期解析DNS记录，动态更新上游节点列表。</td><td>只在启动或配置更新时解析DNS记录，后续解析不会影响现有连接。</td></tr><tr><td><strong>上游节点管理</strong></td><td>所有解析到的IP地址都会参与负载均衡。</td><td>使用初始解析的IP地址建立持久连接，后续解析结果不影响现有连接。</td></tr><tr><td><strong>适用场景</strong></td><td>适用于上游服务地址频繁变化，需要动态更新的场景。</td><td>适用于上游服务地址相对稳定，不需要频繁更新的场景。</td></tr><tr><td><strong>选择</strong></td><td>如果上游服务的地址频繁变化，建议使用 <code>STRICT_DNS</code></td><td>如果地址较为稳定，<code>LOGICAL_DNS</code>可能是更好的选择。</td></tr></tbody></table>

## 1.3 EDS（Endpoint Discovery Service） <a href="#id-4-2-3-eds-endpoint-discovery-service-_82" id="id-4-2-3-eds-endpoint-discovery-service-_82"></a>

EDS是xDS API的一部分，通过控制平面动态管理和下发上游服务的地址信息。EDS适用于需要灵活和动态服务发现的大规模微服务架构。

**控制平面配置（假设使用xDS Server）**

```yaml
dynamic_resources:
  cds_config:
    api_config_source:
      api_type: GRPC
      grpc_services:
        envoy_grpc:
          cluster_name: xds_cluster
static_resources:
  clusters:
  - name: xds_cluster
    type: STATIC
    connect_timeout: 0.25s
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: xds_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: xds-server.example.com
                port_value: 443
```

**EDS配置示例**

```yaml
dynamic_resources:
  lds_config:
    ads: {}
  cds_config:
    ads: {}
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: xds_cluster
```

在EDS配置中，Envoy通过xDS API与控制平面通信，动态获取集群和端点信息。`lds_config`和 `cds_config`使用ADS（Aggregated Discovery Service）配置，通过 `ads_config`与xDS Server通信。

#### 1.3.1 自定义服务发现 <a href="#id-424-zi-ding-yi-fu-wu-fa-xian-89" id="id-424-zi-ding-yi-fu-wu-fa-xian-89"></a>

Envoy还可以通过文件系统、HTTP服务、GRPC服务等方式进行自定义服务发现。以下是通过文件系统进行服务发现的示例：

```yaml
static_resources:
  clusters:
  - name: file_service_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: file_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 8080

dynamic_resources:
  cds_config:
    api_config_source:
      api_type: REST
      cluster_names:
      - file_service_cluster
      refresh_delay: 30s
      refresh_rate: 5s
      file_config_source:
        path: /etc/envoy/cds.yaml
```

在这个示例中，Envoy通过读取文件系统中的配置文件来更新服务发现信息。

#### 1.3.2 服务注册和发现系统 <a href="#id-425-fu-wu-zhu-ce-he-fa-xian-xi-tong-93" id="id-425-fu-wu-zhu-ce-he-fa-xian-xi-tong-93"></a>

Envoy可以与服务注册和发现系统（如Consul、Eureka、Etcd、Kubernetes等）集成，通过这些系统动态发现上游服务。

**与Kubernetes集成示例**

```yaml
static_resources:
  clusters:
  - name: kubernetes_service_cluster
    connect_timeout: 0.25s
    type: EDS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: kubernetes_service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: kubernetes.default.svc.cluster.local
                port_value: 443

dynamic_resources:
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: xds_cluster
  cds_config:
    ads: {}
  lds_config:
    ads: {}
```

在与Kubernetes集成的示例中，Envoy通过Kubernetes的服务发现机制动态获取服务信息。

Envoy提供了多种服务发现机制，以适应不同的环境和需求。从简单的静态配置到复杂的动态发现，Envoy能够灵活地集成各种服务发现系统，确保上游服务的地址信息始终保持最新，从而实现高效的负载均衡和路由。
