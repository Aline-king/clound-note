# 基于gRPC管理服务器订阅

基于gRPC的管理服务器订阅是Envoy的xDS API中一种常用的动态配置方法。

在这种配置模式中，Envoy通过gRPC连接到一个外部的控制平面（也被称为管理服务器），该服务器负责实时地推送配置更新给Envoy实例。

这种方法使得Envoy能够在不重启的情况下，动态地接收和应用配置变更。

<figure><img src="../../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**使用优势**

* **高度动态**：允许实时更新配置，无需重新启动Envoy。
* **集中管理**：所有的配置更新由一个集中的控制平面管理，方便维护和更新。
* **灵活性和可扩展性**：适合大规模和复杂的服务网络，可以灵活处理各种动态变化的需求。

**如何工作**

1. **建立连接**：Envoy启动时，会建立一个持久的gRPC连接到配置好的管理服务器。
2. **订阅配置**：Envoy通过gRPC流向管理服务器订阅一种或多种类型的xDS API（例如CDS、EDS、LDS、RDS等）。
3. **接收配置**：管理服务器响应订阅请求，并根据Envoy的需求动态推送配置数据。
4. **应用更新**：Envoy接收到新的配置信息后，会自动解析并应用这些更新，而无需重启服务。

**管理服务器的角色**

管理服务器的作用是中央控制点，负责生成和分发配置。它可以是任何实现了Envoy xDS协议的服务，如Istio的Pilot、开源的Go Control Plane，或者是自定义的实现。管理服务器通常会进行以下操作：

* 监控服务注册表或其他来源以了解后端服务的变化。
* 生成相应的xDS响应，如服务发现（EDS）、路由（RDS）、监听器（LDS）等配置。
* 响应Envoy的xDS请求，根据Envoy的当前状态和策略推送更新。

这种基于gRPC的动态配置方法是在大规模和动态环境中实现服务网格管理的一种高效方式。

{% tabs %}
{% tab title="jq 安装" %}


```bash
# wget -O /etc/yum.repos.d/epel.repo  https://mirrors.aliyun.com/repo/epel-7.repo
```

```bash
# yum -y install jq
```
{% endtab %}

{% tab title="准备" %}
```bash
[root@dockerhost-envoy ~]# mkdir envoy_xds_grpc_lds_cds
[root@dockerhost-envoy ~]# cd envoy_xds_grpc_lds_cds/
[root@dockerhost-envoy envoy_xds_grpc_lds_cds]# mkdir resources
[root@dockerhost-envoy envoy_xds_grpc_lds_cds]# ls
resources
```

```
# ls
docker-compose.yaml  envoy-sidecar-proxy.yaml  front-envoy.yaml  README.md  resources
```

<details>

<summary>docker-compose.yaml</summary>

```
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
        ipv4_address: 172.28.1.2
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
        ipv4_address: 172.28.1.3
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
        ipv4_address: 172.28.1.4
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
        ipv4_address: 172.28.1.5
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
        - subnet: 172.28.1.0/24
```



</details>

<details>

<summary>front-envoy</summary>

这份配置通过定义静态和动态资源的方式，使 Envoy 能够灵活地处理入站和出站流量，同时提供了丰富的配置选项来优化性能和可靠性。

```yaml
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
  lds_config:
    resource_api_version: V3
    api_config_source:
      api_type: GRPC
      transport_api_version: V3
      grpc_services:
      - envoy_grpc:
          cluster_name: xds_cluster
  cds_config:
    resource_api_version: V3
    api_config_source:
      api_type: GRPC
      transport_api_version: V3
      grpc_services:
      - envoy_grpc:
          cluster_name: xds_cluster
static_resources:
  clusters:
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
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
    http2_protocol_options: {}
    circuit_breakers:
      thresholds:
      - priority: DEFAULT
        max_connections: 1000
        max_pending_requests: 1000
        max_requests: 1000
        max_retries: 3
```

![](<../../../../../.gitbook/assets/image (8) (1) (1).png>)

</details>

<details>

<summary>envoy-sidecar-proxy</summary>

这些配置为 Envoy 提供了详细的指南，确保其能够有效地管理网络流量，同时提供高效的路由、过滤和负载均衡功能。

```yaml
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

![](<../../../../../.gitbook/assets/image (11) (1).png>)

</details>
{% endtab %}

{% tab title="配置文件" %}
```
# cd resources/
# ls
 config.yaml  config.yaml-v1  config.yaml-v2
```

<details>

<summary>config.yaml</summary>

```
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
     - address: 172.28.1.3
       port: 80
```



</details>

<details>

<summary>config.yaml-v2</summary>

```
name: myconfig
 spec:
   listeners:
   - name: listener_http
     address: 0.0.0.0
     port: 30080
     routes:
     - name: local_route
       prefix: /
       clusters:
       - webcluster
   clusters:
   - name: webcluster
     endpoints:
     - address: 172.28.1.3
       port: 80
     - address: 172.28.1.4
       port: 80
```

</details>

<details>

<summary>config.yaml-v1</summary>

```
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
     - address: 172.28.1.3
       port: 80
```

</details>
{% endtab %}

{% tab title="运行并测试" %}
六个Service:&#x20;

* <mark style="color:blue;">envoy：Front Proxy，地址为172.28.1.2</mark>&#x20;
* <mark style="color:purple;">webserver01：第一个后端服务</mark>&#x20;
* <mark style="color:purple;">webserver01-sidecar：第一个后端服务的Sidecar Proxy，地址为172.28.1.3</mark>&#x20;
* <mark style="color:green;">webserver02：第二个后端服务</mark>&#x20;
* <mark style="color:green;">webserver02-sidecar：第二个后端服务的Sidecar Proxy，地址为172.28.1.4</mark>
* &#x20;xdsserver: xDS management server，地址为172.28.1.5

## 创建

```powershell
[root@dockerhost-envoy envoy_xds_grpc_lds_cds]# docker-compose up
[+] Running 7/0
 ✔ Network envoy_xds_grpc_lds_cds_envoymesh                Created                         0.0s
 ✔ Container envoy_xds_grpc_lds_cds-xdsserver-1            Created                         0.0s
 ✔ Container envoy_xds_grpc_lds_cds-webserver01-1          Created                         0.0s
 ✔ Container envoy_xds_grpc_lds_cds-webserver02-1          Created                         0.0s
 ✔ Container envoy_xds_grpc_lds_cds-webserver02-sidecar-1  Created                         0.0s
 ✔ Container envoy_xds_grpc_lds_cds-webserver01-sidecar-1  Created                         0.0s
 ✔ Container envoy_xds_grpc_lds_cds-envoy-1                Created                         0.0s
Attaching to envoy-1, webserver01-1, webserver01-sidecar-1, webserver02-1, webserver02-sidecar-1, xdsserver-1
```

## 测试

查看Cluster及Endpoints信息

```powershell
# curl http://172.28.1.2
demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver01, ServerIP: 172.28.1.3!
```

```powershell
# curl 172.28.1.2:9901/clusters
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
webcluster::172.28.1.3:80::cx_active::4
webcluster::172.28.1.3:80::cx_connect_fail::0
webcluster::172.28.1.3:80::cx_total::5
webcluster::172.28.1.3:80::rq_active::0
webcluster::172.28.1.3:80::rq_error::0
webcluster::172.28.1.3:80::rq_success::5
webcluster::172.28.1.3:80::rq_timeout::0
webcluster::172.28.1.3:80::rq_total::5
webcluster::172.28.1.3:80::hostname::
webcluster::172.28.1.3:80::health_flags::healthy
webcluster::172.28.1.3:80::weight::1
webcluster::172.28.1.3:80::region::
webcluster::172.28.1.3:80::zone::
webcluster::172.28.1.3:80::sub_zone::
webcluster::172.28.1.3:80::canary::false
webcluster::172.28.1.3:80::priority::0
webcluster::172.28.1.3:80::success_rate::-1
webcluster::172.28.1.3:80::local_origin_success_rate::-1
xds_cluster::observability_name::xds_cluster
xds_cluster::default_priority::max_connections::1000
xds_cluster::default_priority::max_pending_requests::1000
xds_cluster::default_priority::max_requests::1000
xds_cluster::default_priority::max_retries::3
xds_cluster::high_priority::max_connections::1024
xds_cluster::high_priority::max_pending_requests::1024
xds_cluster::high_priority::max_requests::1024
xds_cluster::high_priority::max_retries::3
xds_cluster::added_via_api::false
xds_cluster::172.28.1.5:18000::cx_active::1
xds_cluster::172.28.1.5:18000::cx_connect_fail::0
xds_cluster::172.28.1.5:18000::cx_total::1
xds_cluster::172.28.1.5:18000::rq_active::2
xds_cluster::172.28.1.5:18000::rq_error::0
xds_cluster::172.28.1.5:18000::rq_success::0
xds_cluster::172.28.1.5:18000::rq_timeout::0
xds_cluster::172.28.1.5:18000::rq_total::2
xds_cluster::172.28.1.5:18000::hostname::xdsserver
xds_cluster::172.28.1.5:18000::health_flags::healthy
xds_cluster::172.28.1.5:18000::weight::1
xds_cluster::172.28.1.5:18000::region::
xds_cluster::172.28.1.5:18000::zone::
xds_cluster::172.28.1.5:18000::sub_zone::
xds_cluster::172.28.1.5:18000::canary::false
xds_cluster::172.28.1.5:18000::priority::0
xds_cluster::172.28.1.5:18000::success_rate::-1
xds_cluster::172.28.1.5:18000::local_origin_success_rate::-1
```

查看Listener列表；

```powershell
# curl 172.28.1.2:9901/listeners
listener_http::0.0.0.0:80
```

```powershell
# curl -s 172.28.1.2:9901/config_dump | jq '.configs[1].dynamic_active_clusters'
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
                      "address": "172.28.1.3",
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
    "last_updated": "2024-05-25T09:06:29.111Z"
  }
]
```

```powershell
# curl -s 172.28.1.2:9901/config_dump?resource=dynamic_listeners | jq '.configs[0].active_state.listener.address'
{
  "socket_address": {
    "address": "0.0.0.0",
    "port_value": 80
  }
}
```

```powershell
# curl -s 172.28.1.2:9901/config_dump?resource=dynamic_listeners | jq '.configs[0].active_state.listener'
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

```powershell
# cd envoy_xds_grpc_lds_cds/

# docker-compose ps
NAME                                           IMAGE                                      COMMAND                   SERVICE               CREATED       STATUS       PORTS
envoy_xds_grpc_lds_cds-envoy-1                 envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   envoy                 3 hours ago   Up 3 hours   10000/tcp
envoy_xds_grpc_lds_cds-webserver01-1           www.kubemsb.com/envoy/demoapp:v1.0         "/bin/sh -c 'python3…"   webserver01           3 hours ago   Up 3 hours
envoy_xds_grpc_lds_cds-webserver01-sidecar-1   envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   webserver01-sidecar   3 hours ago   Up 3 hours
envoy_xds_grpc_lds_cds-webserver02-1           www.kubemsb.com/envoy/demoapp:v1.0         "/bin/sh -c 'python3…"   webserver02           3 hours ago   Up 3 hours
envoy_xds_grpc_lds_cds-webserver02-sidecar-1   envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   webserver02-sidecar   3 hours ago   Up 3 hours
envoy_xds_grpc_lds_cds-xdsserver-1             www.kubemsb.com/envoy/xds-server:v0.12.0   "./xds-server"            xdsserver             3 hours ago   Up 3 hours   18000/tcp
```

### 接入xdsserver容器的交互式接口

修改config.yaml文件中的内容，将另一个endpoint添加进文件中，或进行其它修改；

```powershell
# docker-compose exec xdsserver /bin/sh
# cd /etc/envoy-xds-server/config/
/etc/envoy-xds-server/config # ls
config.yaml     config.yaml-v1  config.yaml-v2
/etc/envoy-xds-server/config # cat config.yaml-v2 > config.yaml
```

```powershell
查看容器输出日志：

xdsserver-1            | 2024/05/25 12:54:27 Config file modified: /etc/envoy-xds-server/config/config.yaml
xdsserver-1            | 2024/05/25 12:54:27 Parsed config: {Name: Spec:{Listeners:[] Clusters:[]}}
xdsserver-1            | 2024/05/25 12:54:27 Loaded snapshot: &{Resources:[{Version:1 Items:map[]} {Version: Items:map[]} {Version:1 Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]}] VersionMap:map[]}
xdsserver-1            | 2024/05/25 12:54:27 Snapshot set successfully for node ID: envoy_front_proxy
xdsserver-1            | 2024/05/25 12:54:27 Config file modified: /etc/envoy-xds-server/config/config.yaml
xdsserver-1            | 2024/05/25 12:54:27 Parsed config: {Name:myconfig Spec:{Listeners:[{Name:listener_http Address:0.0.0.0 Port:30080 Routes:[{Name:local_route Prefix:/ Clusters:[webcluster]}]}] Clusters:[{Name:webcluster Endpoints:[{Address:172.28.1.3 Port:80} {Address:172.28.1.4 Port:80}]}]}}
xdsserver-1            | 2024/05/25 12:54:27 Loaded snapshot: &{Resources:[{Version:1 Items:map[webcluster:{Resource:name:"webcluster"  type:STATIC  connect_timeout:{nanos:250000000}  load_assignment:{cluster_name:"webcluster"  endpoints:{lb_endpoints:{endpoint:{address:{socket_address:{address:"172.28.1.3"  port_value:80}}}}  lb_endpoints:{endpoint:{address:{socket_address:{address:"172.28.1.4"  port_value:80}}}}}} TTL:<nil>}]} {Version: Items:map[]} {Version:1 Items:map[listener_http:{Resource:name:"listener_http"  address:{socket_address:{address:"0.0.0.0"  port_value:30080}}  filter_chains:{filters:{name:"envoy.filters.network.http_connection_manager"  typed_config:{[type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager]:{stat_prefix:"ingress_http"  route_config:{name:"local_route"  virtual_hosts:{name:"local_route"  domains:"*"  routes:{match:{prefix:"/"}  route:{cluster:"webcluster"}}}}  http_filters:{name:"envoy.filters.http.router"  typed_config:{type_url:"type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"}}}}}} TTL:<nil>}]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]}] VersionMap:map[]}
xdsserver-1            | 2024/05/25 12:54:27 Snapshot set successfully for node ID: envoy_front_proxy
```

```powershell
# curl http://172.28.1.2:9901/clusters
xds_cluster::observability_name::xds_cluster
xds_cluster::default_priority::max_connections::1000
xds_cluster::default_priority::max_pending_requests::1000
xds_cluster::default_priority::max_requests::1000
xds_cluster::default_priority::max_retries::3
xds_cluster::high_priority::max_connections::1024
xds_cluster::high_priority::max_pending_requests::1024
xds_cluster::high_priority::max_requests::1024
xds_cluster::high_priority::max_retries::3
xds_cluster::added_via_api::false
xds_cluster::172.28.1.5:18000::cx_active::1
xds_cluster::172.28.1.5:18000::cx_connect_fail::0
xds_cluster::172.28.1.5:18000::cx_total::1
xds_cluster::172.28.1.5:18000::rq_active::2
xds_cluster::172.28.1.5:18000::rq_error::0
xds_cluster::172.28.1.5:18000::rq_success::0
xds_cluster::172.28.1.5:18000::rq_timeout::0
xds_cluster::172.28.1.5:18000::rq_total::2
xds_cluster::172.28.1.5:18000::hostname::xdsserver
xds_cluster::172.28.1.5:18000::health_flags::healthy
xds_cluster::172.28.1.5:18000::weight::1
xds_cluster::172.28.1.5:18000::region::
xds_cluster::172.28.1.5:18000::zone::
xds_cluster::172.28.1.5:18000::sub_zone::
xds_cluster::172.28.1.5:18000::canary::false
xds_cluster::172.28.1.5:18000::priority::0
xds_cluster::172.28.1.5:18000::success_rate::-1
xds_cluster::172.28.1.5:18000::local_origin_success_rate::-1
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
webcluster::172.28.1.3:80::cx_active::1
webcluster::172.28.1.3:80::cx_connect_fail::0
webcluster::172.28.1.3:80::cx_total::1
webcluster::172.28.1.3:80::rq_active::0
webcluster::172.28.1.3:80::rq_error::0
webcluster::172.28.1.3:80::rq_success::1
webcluster::172.28.1.3:80::rq_timeout::0
webcluster::172.28.1.3:80::rq_total::1
webcluster::172.28.1.3:80::hostname::
webcluster::172.28.1.3:80::health_flags::healthy
webcluster::172.28.1.3:80::weight::1
webcluster::172.28.1.3:80::region::
webcluster::172.28.1.3:80::zone::
webcluster::172.28.1.3:80::sub_zone::
webcluster::172.28.1.3:80::canary::false
webcluster::172.28.1.3:80::priority::0
webcluster::172.28.1.3:80::success_rate::-1
webcluster::172.28.1.3:80::local_origin_success_rate::-1
webcluster::172.28.1.4:80::cx_active::5
webcluster::172.28.1.4:80::cx_connect_fail::0
webcluster::172.28.1.4:80::cx_total::5
webcluster::172.28.1.4:80::rq_active::0
webcluster::172.28.1.4:80::rq_error::0
webcluster::172.28.1.4:80::rq_success::5
webcluster::172.28.1.4:80::rq_timeout::0
webcluster::172.28.1.4:80::rq_total::5
webcluster::172.28.1.4:80::hostname::
webcluster::172.28.1.4:80::health_flags::healthy
webcluster::172.28.1.4:80::weight::1
webcluster::172.28.1.4:80::region::
webcluster::172.28.1.4:80::zone::
webcluster::172.28.1.4:80::sub_zone::
webcluster::172.28.1.4:80::canary::false
webcluster::172.28.1.4:80::priority::0
webcluster::172.28.1.4:80::success_rate::-1
webcluster::172.28.1.4:80::local_origin_success_rate::-1
```

```powershell
# curl http://172.28.1.2:9901/listeners
listener_http::0.0.0.0:30080
```

```powershell
# curl http://172.28.1.2:30080
demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver02, ServerIP: 172.28.1.4!

# curl http://172.28.1.2:30080
demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver01, ServerIP: 172.28.1.3!
```

```powershell
/etc/envoy-xds-server/config # cat config.yaml-v1 > config.yaml
```

```powershell
查看容器输出日志：
xdsserver-1            | 2024/05/25 13:49:18 Config file modified: /etc/envoy-xds-server/config/config.yaml
xdsserver-1            | 2024/05/25 13:49:18 Parsed config: {Name: Spec:{Listeners:[] Clusters:[]}}
xdsserver-1            | 2024/05/25 13:49:18 Loaded snapshot: &{Resources:[{Version:15 Items:map[]} {Version: Items:map[]} {Version:15 Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]}] VersionMap:map[]}
xdsserver-1            | 2024/05/25 13:49:18 Snapshot set successfully for node ID: envoy_front_proxy with version: 16
xdsserver-1            | 2024/05/25 13:49:18 Config file modified: /etc/envoy-xds-server/config/config.yaml
xdsserver-1            | 2024/05/25 13:49:18 Parsed config: {Name:myconfig Spec:{Listeners:[{Name:listener_http Address:0.0.0.0 Port:80 Routes:[{Name:local_route Prefix:/ Clusters:[webcluster]}]}] Clusters:[{Name:webcluster Endpoints:[{Address:172.28.1.3 Port:80}]}]}}
xdsserver-1            | 2024/05/25 13:49:18 Loaded snapshot: &{Resources:[{Version:17 Items:map[webcluster:{Resource:name:"webcluster"  type:STATIC  connect_timeout:{nanos:250000000}  load_assignment:{cluster_name:"webcluster"  endpoints:{lb_endpoints:{endpoint:{address:{socket_address:{address:"172.28.1.3"  port_value:80}}}}}} TTL:<nil>}]} {Version: Items:map[]} {Version:17 Items:map[listener_http:{Resource:name:"listener_http"  address:{socket_address:{address:"0.0.0.0"  port_value:80}}  filter_chains:{filters:{name:"envoy.filters.network.http_connection_manager"  typed_config:{[type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager]:{stat_prefix:"ingress_http"  route_config:{name:"local_route"  virtual_hosts:{name:"local_route"  domains:"*"  routes:{match:{prefix:"/"}  route:{cluster:"webcluster"}}}}  http_filters:{name:"envoy.filters.http.router"  typed_config:{type_url:"type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"}}}}}} TTL:<nil>}]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]} {Version: Items:map[]}] VersionMap:map[]}
xdsserver-1            | 2024/05/25 13:49:18 Snapshot set successfully for node ID: envoy_front_proxy with version: 18
envoy-1                | [2024-05-25 13:49:18.436][1][info][upstream] [source/common/listener_manager/lds_api.cc:64] lds: remove listener 'listener_http'
envoy-1                | [2024-05-25 13:49:18.437][1][info][upstream] [source/common/upstream/cds_api_helper.cc:32] cds: add 0 cluster(s), remove 2 cluster(s)
envoy-1                | [2024-05-25 13:49:18.437][1][info][upstream] [source/common/upstream/cds_api_helper.cc:71] cds: added/updated 0 cluster(s), skipped 0 unmodified cluster(s)
envoy-1                | [2024-05-25 13:49:18.438][1][info][upstream] [source/common/upstream/cds_api_helper.cc:32] cds: add 1 cluster(s), remove 1 cluster(s)
envoy-1                | [2024-05-25 13:49:18.439][1][info][upstream] [source/common/upstream/cds_api_helper.cc:71] cds: added/updated 1 cluster(s), skipped 0 unmodified cluster(s)
envoy-1                | [2024-05-25 13:49:18.441][1][info][upstream] [source/common/listener_manager/lds_api.cc:102] lds: add/update listener 'listener_http'
```

### 再次查看Cluster中的Endpoint信息

```powershell
# curl http://172.28.1.2:9901/clusters
xds_cluster::observability_name::xds_cluster
xds_cluster::default_priority::max_connections::1000
xds_cluster::default_priority::max_pending_requests::1000
xds_cluster::default_priority::max_requests::1000
xds_cluster::default_priority::max_retries::3
xds_cluster::high_priority::max_connections::1024
xds_cluster::high_priority::max_pending_requests::1024
xds_cluster::high_priority::max_requests::1024
xds_cluster::high_priority::max_retries::3
xds_cluster::added_via_api::false
xds_cluster::172.28.1.5:18000::cx_active::1
xds_cluster::172.28.1.5:18000::cx_connect_fail::0
xds_cluster::172.28.1.5:18000::cx_total::1
xds_cluster::172.28.1.5:18000::rq_active::2
xds_cluster::172.28.1.5:18000::rq_error::0
xds_cluster::172.28.1.5:18000::rq_success::0
xds_cluster::172.28.1.5:18000::rq_timeout::0
xds_cluster::172.28.1.5:18000::rq_total::2
xds_cluster::172.28.1.5:18000::hostname::xdsserver
xds_cluster::172.28.1.5:18000::health_flags::healthy
xds_cluster::172.28.1.5:18000::weight::1
xds_cluster::172.28.1.5:18000::region::
xds_cluster::172.28.1.5:18000::zone::
xds_cluster::172.28.1.5:18000::sub_zone::
xds_cluster::172.28.1.5:18000::canary::false
xds_cluster::172.28.1.5:18000::priority::0
xds_cluster::172.28.1.5:18000::success_rate::-1
xds_cluster::172.28.1.5:18000::local_origin_success_rate::-1
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
webcluster::172.28.1.3:80::cx_active::0
webcluster::172.28.1.3:80::cx_connect_fail::0
webcluster::172.28.1.3:80::cx_total::0
webcluster::172.28.1.3:80::rq_active::0
webcluster::172.28.1.3:80::rq_error::0
webcluster::172.28.1.3:80::rq_success::0
webcluster::172.28.1.3:80::rq_timeout::0
webcluster::172.28.1.3:80::rq_total::0
webcluster::172.28.1.3:80::hostname::
webcluster::172.28.1.3:80::health_flags::healthy
webcluster::172.28.1.3:80::weight::1
webcluster::172.28.1.3:80::region::
webcluster::172.28.1.3:80::zone::
webcluster::172.28.1.3:80::sub_zone::
webcluster::172.28.1.3:80::canary::false
webcluster::172.28.1.3:80::priority::0
webcluster::172.28.1.3:80::success_rate::-1
webcluster::172.28.1.3:80::local_origin_success_rate::-1
```

```powershell
# curl http://172.28.1.2:9901/listeners
listener_http::0.0.0.0:80
```

```powershell
# curl http://172.28.1.2
demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: webserver01, ServerIP: 172.28.1.3!
```
{% endtab %}
{% endtabs %}
