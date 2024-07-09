# ADS

在 Envoy Proxy 的上下文中，ADS 指的是**聚合发现服务**（Aggregated Discovery Service）。ADS 是一种优化方式，允许 Envoy 使用单个 gRPC 连接来接收所有类型的 xDS 配置更新，而不是为每种资源类型（如 CDS、EDS、LDS、RDS 等）建立单独的连接。

这种方法的主要优点包括：

1. **减少网络开销**：通过使用单个连接来传输所有类型的配置更新，减少了网络负载和连接维护的复杂性。
2. **配置一致性**：聚合发现服务可以确保配置的原子更新和一致性。这意味着所有相关的配置更新可以在同一时间内同步传送给 Envoy，减少了不同配置之间可能出现的不一致性问题。
3. **简化管理**：对于管理服务器来说，使用单一的 gRPC 连接向 Envoy 推送更新可以简化配置管理的逻辑，尤其是在大规模部署中。

ADS 的工作流程大致如下：

* **订阅**：Envoy 启动时，通过单一的 gRPC 连接向管理服务器请求订阅所有需要的 xDS 资源。
* **推送更新**：当任何配置资源（如路由规则、服务端点）发生变化时，管理服务器会将更新的资源集合通过这个单一的连接推送给 Envoy。
* **处理更新**：Envoy 接收到更新后，将相应地更新其内部状态以反映新的配置，这包括可能的路由更改、服务发现信息更新等。

通过 ADS，Envoy 和其管理服务器之间的通信更加高效，同时也支持了更加动态和弹性的服务网络管理。这在复杂的微服务环境中尤为重要，可以有效地支持服务的快速扩展和频繁变更。



{% tabs %}
{% tab title="准备文件" %}
```powershell
[root@dockerhost-envoy ~]# mkdir envoy_xds_grpc_ads
[root@dockerhost-envoy ~]# cd envoy_xds_grpc_ads/
```

<details>

<summary>docker-compose.yaml</summary>

```yaml
# cat docker-compose.yaml
version: '3.3'
services:
  envoy:
    image: envoyproxy/envoy:v1.30.1
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
    volumes:
    - ./front-envoy.yaml:/etc/envoy/envoy.yaml
    networks:
      envoymesh:
        ipv4_address: 172.29.1.2
        aliases:
        - front-proxy
    depends_on:
    - webserver01
    - webserver02
    - xdsserver
  webserver01:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
      - PORT=8080
      - HOST=127.0.0.1
    hostname: webserver01
    networks:
      envoymesh:
        ipv4_address: 172.29.1.3
  webserver01-sidecar:
    image: envoyproxy/envoy:v1.30.1
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
    volumes:
    - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
    network_mode: "service:webserver01"
    depends_on:
    - webserver01
  webserver02:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
      - PORT=8080
      - HOST=127.0.0.1
    hostname: webserver02
    networks:
      envoymesh:
        ipv4_address: 172.29.1.4
  webserver02-sidecar:
    image: envoyproxy/envoy:v1.30.1
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
    volumes:
    - ./envoy-sidecar-proxy.yaml:/etc/envoy/envoy.yaml
    network_mode: "service:webserver02"
    depends_on:
    - webserver02
  xdsserver:
    image: www.kubemsb.com/envoy/xds-server:v0.12.0
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
      - SERVER_PORT=18000
      - NODE_ID=envoy_front_proxy
      - RESOURCES_FILE=/etc/envoy-xds-server/config/config.yaml
    volumes:
    - ./resources:/etc/envoy-xds-server/config/
    networks:
      envoymesh:
        ipv4_address: 172.29.1.5
        aliases:
        - xdsserver
        - xds-service
    expose:
    - "18000"
networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.29.1.0/24
```



</details>

<details>

<summary>front-envoy.yaml</summary>

```powershell
# cat front-envoy.yaml
node:
  id: envoy_front_proxy
  cluster: webcluster
admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
       address: 0.0.0.0
       port_value: 9901
dynamic_resources:
  ads_config:
    api_type: GRPC
    transport_api_version: V3
    grpc_services:
    - envoy_grpc:
        cluster_name: xds_cluster
    set_node_on_first_message_only: true
  cds_config:
    resource_api_version: V3
    ads: {}
  lds_config:
    resource_api_version: V3
    ads: {}
static_resources:
  clusters:
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    # The extension_protocol_options field is used to provide extension-specific protocol options for upstream connections.
    typed_extension_protocol_options:
      envoy.extensions.upstreams.http.v3.HttpProtocolOptions:
        "@type": type.googleapis.com/envoy.extensions.upstreams.http.v3.HttpProtocolOptions
        explicit_http_config:
          http2_protocol_options: {}
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: xds_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: xdsserver
                port_value: 18000
```



</details>

<details>

<summary>envoy-sidecar-proxy.yaml</summary>

```powershell
# cat envoy-sidecar-proxy.yaml
admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
       address: 0.0.0.0
       port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
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
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: local_cluster }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
  clusters:
  - name: local_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: local_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: 127.0.0.1, port_value: 8080 }
```



</details>
{% endtab %}

{% tab title="resources" %}
<pre class="language-powershell"><code class="lang-powershell"><strong># mkdir resources
</strong># cd resources/
# ls
config.yaml  config.yaml-v1  config.yaml-v2
</code></pre>

<details>

<summary>config.yaml</summary>

```yaml
# cat config.yaml
name: myconfig
spec:
  listeners:
  - name: listener_http
    address: 0.0.0.0
    port: 80
    routes:
    - name: local_route
      prefix: /
      clusters:
      - webcluster
  clusters:
  - name: webcluster
    endpoints:
    - address: 172.29.1.3
      port: 80
```



</details>

<details>

<summary>config.yaml-v2</summary>

```yaml
# cat config.yaml-v2
name: myconfig
spec:
  listeners:
  - name: listener_http
    address: 0.0.0.0
    port: 80
    routes:
    - name: local_route
      prefix: /
      clusters:
      - webcluster
  clusters:
  - name: webcluster
    endpoints:
    - address: 172.29.1.3
      port: 80
    - address: 172.29.1.4
      port: 80
```



</details>

<details>

<summary>config.yaml-v1</summary>

```yaml
# cat config.yaml-v1
name: myconfig
spec:
  listeners:
  - name: listener_http
    address: 0.0.0.0
    port: 80
    routes:
    - name: local_route
      prefix: /
      clusters:
      - webcluster
  clusters:
  - name: webcluster
    endpoints:
    - address: 172.29.1.3
      port: 80
```



</details>
{% endtab %}

{% tab title="运行并测试" %}
六个Service:

* envoy：Front Proxy,地址为172.29.1.2
* webserver01：第一个后端服务
* webserver01-sidecar：第一个后端服务的Sidecar Proxy,地址为172.29.1.3
* webserver02：第二个后端服务
* webserver02-sidecar：第二个后端服务的Sidecar Proxy,地址为172.29.1.4
* xdsserver: xDS management server，地址为172.29.1.5

{% tabs %}
{% tab title="启动" %}
```
# docker-compose up
 [+] Running 7/0
  ✔ Network envoy_xds_grpc_ads_envoymesh                Created                              0.0s
  ✔ Container envoy_xds_grpc_ads-webserver02-1          Created                              0.0s
  ✔ Container envoy_xds_grpc_ads-xdsserver-1            Created                              0.0s
  ✔ Container envoy_xds_grpc_ads-webserver01-1          Created                              0.0s
  ✔ Container envoy_xds_grpc_ads-webserver01-sidecar-1  Created                              0.0s
  ✔ Container envoy_xds_grpc_ads-webserver02-sidecar-1  Created                              0.0s
  ✔ Container envoy_xds_grpc_ads-envoy-1                Created                              0.0s
 Attaching to envoy-1, webserver01-1, webserver01-sidecar-1, webserver02-1, webserver02-sidecar-1, xdsserver-1
```


{% endtab %}

{% tab title="undefined" %}
```
# curl http://172.29.1.2
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver01, ServerIP: 172.29.1.3!
```

## 查看cluster

任选其一

<details>

<summary>查看Cluster及Endpoints信息</summary>



```
# curl http://172.29.1.2:9901/clusters
 # curl http://172.29.1.2:9901/clusters
 xds_cluster::observability_name::xds_cluster
 xds_cluster::default_priority::max_connections::1024
 xds_cluster::default_priority::max_pending_requests::1024
 xds_cluster::default_priority::max_requests::1024
 xds_cluster::default_priority::max_retries::3
 xds_cluster::high_priority::max_connections::1024
 xds_cluster::high_priority::max_pending_requests::1024
 xds_cluster::high_priority::max_requests::1024
 xds_cluster::high_priority::max_retries::3
 xds_cluster::added_via_api::false
 xds_cluster::172.29.1.5:18000::cx_active::1
 xds_cluster::172.29.1.5:18000::cx_connect_fail::0
 xds_cluster::172.29.1.5:18000::cx_total::1
 xds_cluster::172.29.1.5:18000::rq_active::1
 xds_cluster::172.29.1.5:18000::rq_error::0
 xds_cluster::172.29.1.5:18000::rq_success::0
 xds_cluster::172.29.1.5:18000::rq_timeout::0
 xds_cluster::172.29.1.5:18000::rq_total::1
 xds_cluster::172.29.1.5:18000::hostname::xdsserver
 xds_cluster::172.29.1.5:18000::health_flags::healthy
 xds_cluster::172.29.1.5:18000::weight::1
 xds_cluster::172.29.1.5:18000::region::
 xds_cluster::172.29.1.5:18000::zone::
 xds_cluster::172.29.1.5:18000::sub_zone::
 xds_cluster::172.29.1.5:18000::canary::false
 xds_cluster::172.29.1.5:18000::priority::0
 xds_cluster::172.29.1.5:18000::success_rate::-1
 xds_cluster::172.29.1.5:18000::local_origin_success_rate::-1
 webcluster::observability_name::webcluster
 webcluster::default_priority::max_connections::1024
 webcluster::default_priority::max_pending_requests::1024
 webcluster::default_priority::max_requests::1024
 webcluster::default_priority::max_retries::3
 webcluster::high_priority::max_connections::1024
 webcluster::high_priority::max_pending_requests::1024
 webcluster::high_priority::max_requests::1024
 webcluster::high_priority::max_retries::3
 webcluster::added_via_api::true
 webcluster::172.29.1.3:80::cx_active::1
 webcluster::172.29.1.3:80::cx_connect_fail::0
 webcluster::172.29.1.3:80::cx_total::1
 webcluster::172.29.1.3:80::rq_active::0
 webcluster::172.29.1.3:80::rq_error::0
 webcluster::172.29.1.3:80::rq_success::1
 webcluster::172.29.1.3:80::rq_timeout::0
 webcluster::172.29.1.3:80::rq_total::1
 webcluster::172.29.1.3:80::hostname::
 webcluster::172.29.1.3:80::health_flags::healthy
 webcluster::172.29.1.3:80::weight::1
 webcluster::172.29.1.3:80::region::
 webcluster::172.29.1.3:80::zone::
 webcluster::172.29.1.3:80::sub_zone::
 webcluster::172.29.1.3:80::canary::false
 webcluster::172.29.1.3:80::priority::0
 webcluster::172.29.1.3:80::success_rate::-1
 webcluster::172.29.1.3:80::local_origin_success_rate::-1
```

</details>

<details>

<summary>查看动态Clusters的相关信息</summary>

```json
# curl -s 172.29.1.2:9901/config_dump | jq '.configs[1].dynamic_active_clusters'
 [
   {
     "version_info": "1",
     "cluster": {
       "@type": "type.googleapis.com/envoy.config.cluster.v3.Cluster",
       "name": "webcluster",
       "type": "STATIC",
       "connect_timeout": "0.250s",
       "load_assignment": {
         "cluster_name": "webcluster",
         "endpoints": [
           {
             "lb_endpoints": [
               {
                 "endpoint": {
                   "address": {
                     "socket_address": {
                       "address": "172.29.1.3",
                       "port_value": 80
                     }
                   }
                 }
               }
             ]
           }
         ]
       }
     },
     "last_updated": "2024-05-25T14:55:33.262Z"
   }
 ]
```



</details>

## 查看listeners

任选其一

```
# curl http://172.29.1.2:9901/listeners
 listener_http::0.0.0.0:80
```

{% code title="观察容器日志输出" %}
```bash
 xdsserver-1            | 2024/05/25 14:55:32 Loading config from /etc/envoy-xds-server/config/config.yaml
 xdsserver-1            | 2024/05/25 14:55:32 Parsed config: {Name:myconfig Spec:{Listeners:[{Name:listener_http Address:0.0.0.0 Port:80 Routes:[{Name:local_route Prefix:/ Clusters:[webcluster]}]}] Clusters:[{Name:webcluster Endpoints:[{Address:172.29.1.3 Port:80}]}]}}
 xdsserver-1            | 2024/05/25 14:55:32 Loaded snapshot: &{Resources:[{Version:1 Items:map[webcluster:{Resource:name:"webcluster"  type:STATIC  connect_timeout:{nanos:250000000}  load_assignment:{cluster_name:"webcluster"  endpoints:{lb_endpoints:{endpoint:{address:{socket_address:{address:"172.29.1.3"  port_value:80}}}}}} TTL:<nil>}]} {Version: Items:map[]} {Version:1 Items:map[listener_http:{Resource:name:"listener_http"  address:{socket_address:{address:"0.0.0.0"  port_value:80}}  filter_chains:{filters:{name:"envoy.filters.network.http_connection_manager"  typed_config:{[type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager]:{stat_prefix:"ingress_http"  route_config:{name:"local_route"  virtual_hosts:{name:"local_route"  domains:"*"  routes:{match:{prefix:"/"}  route:{cluster:"webcluster"}}}}  http_filters:{name:"envoy.filters.http.router"  typed_config:{type_url:"type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"}}}}}} TTL:<nil>}]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]}] VersionMap:map[]}
 xdsserver-1            | 2024/05/25 14:55:32 Snapshot set successfully for node ID: envoy_front_proxy with version: 2
```
{% endcode %}

<details>

<summary>查看动态的Listener信息</summary>

```json
# curl -s 172.29.1.2:9901/config_dump?resource=dynamic_listeners | jq '.configs[0].active_state.listener.address'
 {
   "socket_address": {
     "address": "0.0.0.0",
     "port_value": 80
   }
 }
```

```
# curl -s 172.29.1.2:9901/config_dump?resource=dynamic_listeners | jq '.configs[0].active_state.listener'
 {
   "@type": "type.googleapis.com/envoy.config.listener.v3.Listener",
   "name": "listener_http",
   "address": {
     "socket_address": {
       "address": "0.0.0.0",
       "port_value": 80
     }
   },
   "filter_chains": [
     {
       "filters": [
         {
           "name": "envoy.filters.network.http_connection_manager",
           "typed_config": {
             "@type": "type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager",
             "stat_prefix": "ingress_http",
             "route_config": {
               "name": "local_route",
               "virtual_hosts": [
                 {
                   "name": "local_route",
                   "domains": [
                     "*"
                   ],
                   "routes": [
                     {
                       "match": {
                         "prefix": "/"
                       },
                       "route": {
                         "cluster": "webcluster"
                       }
                     }
                   ]
                 }
               ]
             },
             "http_filters": [
               {
                 "name": "envoy.filters.http.router",
                 "typed_config": {
                   "@type": "type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"
                 }
               }
             ]
           }
         }
       ]
     }
   ]
 }
```



</details>

```
[root@dockerhost-envoy ~]# cd envoy_xds_grpc_ads/
 [root@dockerhost-envoy envoy_xds_grpc_ads]# docker-compose ps
 NAME                                       IMAGE                                      COMMAND                   SERVICE               CREATED         STATUS         PORTS
 envoy_xds_grpc_ads-envoy-1                 envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   envoy                 6 minutes ago   Up 6 minutes   10000/tcp
 envoy_xds_grpc_ads-webserver01-1           www.kubemsb.com/envoy/demoapp:v1.0         "/bin/sh -c 'python3…"   webserver01           6 minutes ago   Up 6 minutes
 envoy_xds_grpc_ads-webserver01-sidecar-1   envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   webserver01-sidecar   6 minutes ago   Up 6 minutes
 envoy_xds_grpc_ads-webserver02-1           www.kubemsb.com/envoy/demoapp:v1.0         "/bin/sh -c 'python3…"   webserver02           6 minutes ago   Up 6 minutes
 envoy_xds_grpc_ads-webserver02-sidecar-1   envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   webserver02-sidecar   6 minutes ago   Up 6 minutes
 envoy_xds_grpc_ads-xdsserver-1             www.kubemsb.com/envoy/xds-server:v0.12.0   "./xds-server"            xdsserver             6 minutes ago   Up 6 minutes   18000/tcp
```

### 接入xdsserver容器的交互式接口

```
# docker-compose exec xdsserver /bin/sh
 # cd /etc/envoy-xds-server/config/
 /etc/envoy-xds-server/config # ls
 config.yaml     config.yaml-v1  config.yaml-v2
 /etc/envoy-xds-server/config # cat config.yaml-v2 > config.yaml
```

```json
# curl -s 172.29.1.2:9901/config_dump | jq '.configs[1].dynamic_active_clusters'
 [
   {
     "version_info": "5",
     "cluster": {
       "@type": "type.googleapis.com/envoy.config.cluster.v3.Cluster",
       "name": "webcluster",
       "type": "STATIC",
       "connect_timeout": "0.250s",
       "load_assignment": {
         "cluster_name": "webcluster",
         "endpoints": [
           {
             "lb_endpoints": [
               {
                 "endpoint": {
                   "address": {
                     "socket_address": {
                       "address": "172.29.1.3",
                       "port_value": 80
                     }
                   }
                 }
               },
               {
                 "endpoint": {
                   "address": {
                     "socket_address": {
                       "address": "172.29.1.4",
                       "port_value": 80
                     }
                   }
                 }
               }
             ]
           }
         ]
       }
     },
     "last_updated": "2024-05-25T15:03:05.299Z"
   }
 ]
```

```
# curl http://172.29.1.2
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver02, ServerIP: 172.29.1.4!
 
 # curl http://172.29.1.2
 demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver01, ServerIP: 172.29.1.3!
```


{% endtab %}
{% endtabs %}


{% endtab %}
{% endtabs %}

