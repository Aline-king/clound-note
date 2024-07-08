# 使用

```powershell
[root@dockerhost-envoy ~]# mkdir envoy_admin
[root@dockerhost-envoy ~]# cd envoy_admin/
```

## 部署文件



<details>

<summary></summary>



</details>

{% tabs %}
{% tab title="docker-compose" %}


```
# vim docker-compose.yaml
# cat docker-compose.yaml
```

<details>

<summary>文件内容</summary>

```powershell
version: '3.3'
services:
  envoy:
    image: envoyproxy/envoy:v1.30.1
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    environment:
      - ENVOY_UID=0
      - ENVOY_GID=0
    networks:
      envoymesh:
        ipv4_address: 172.25.1.2
        aliases:
        - front-proxy
    depends_on:
    - webserver01
    - webserver02
  webserver01:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - PORT=8080
    hostname: webserver01
    networks:
      envoymesh:
        ipv4_address: 172.25.1.3
        aliases:
        - webserver01
  webserver02:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - PORT=8080
    hostname: webserver02
    networks:
      envoymesh:
        ipv4_address: 172.25.1.4
        aliases:
        - webserver02
networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.1.0/24
```



</details>


{% endtab %}

{% tab title="envoy" %}


<details>

<summary>文件内容</summary>

```powershell
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
            - name: web_service_1
              domains: ["*.kubemsb.com", "kubemsb.com"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: local_cluster }
            - name: web_service_2
              domains: ["*.kubex.com","kubex.com"]
              routes:
              - match: { prefix: "/" }
                redirect:
                  host_redirect: "www.kubemsb.com"
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
              socket_address: { address: 172.25.1.3, port_value: 8080 }
        - endpoint:
            address:
              socket_address: { address: 172.25.1.4, port_value: 8080 }
```



</details>
{% endtab %}
{% endtabs %}

```powershell
上述内容解释：

### 版本声明
- `version: '3.3'`
  指定使用Docker Compose的版本3.3进行服务编排。

### 服务定义
#### Envoy服务
- `envoy`
  定义了一个名为`envoy`的服务。
- `image: envoyproxy/envoy:v1.30.1`
  使用Envoy官方镜像，版本为1.30.1。
- `volumes:`
  将主机上的`./envoy.yaml`配置文件映射到容器内的`/etc/envoy/envoy.yaml`，用以配置Envoy的行为和规则。
- `environment:`
  设置环境变量`ENVOY_UID=0`和`ENVOY_GID=0`，确保Envoy在容器内以root权限运行。
- `networks:`
  将此服务加入到名为`envoymesh`的网络中，分配静态IPv4地址`172.25.1.2`，并为其设置别名`front-proxy`。
- `depends_on:`
  表明`envoy`服务启动前需等待`webserver01`和`webserver02`服务就绪。

#### Web服务器服务（两个实例）
- `webserver01` 和 `webserver02`
  分别定义了两个Web服务器服务，基于特定仓库的镜像`www.kubemsb.com/envoy/demoapp:v1.0`。
- `environment:`
  为每个Web服务器配置环境变量`PORT=8080`，指示应用程序监听8080端口。
- `hostname:`
  分别为两台Web服务器设定了主机名。
- `networks:`
  将这两个服务也加入到`envoymesh`网络，并分别为它们分配了固定的IP地址：`172.25.1.3`给`webserver01`，`172.25.1.4`给`webserver02`，同时设置别名与主机名一致。

### 网络定义
- `networks:`
  创建一个自定义网络`envoymesh`，类型为`bridge`（桥接网络），允许容器间相互通信，同时也可从宿主机访问。
- `ipam:`
  为该网络配置IP地址管理，定义了一个CIDR子网`172.25.1.0/24`，用于分配给网络内的容器使用。

该配置文件旨在部署一个包含Envoy代理和两个后端Web服务器的微服务架构，通过自定义网络`envoymesh`实现服务间的通信，Envoy作为前端代理处理进入的流量并将其导向后端Web服务器。
```

````powershell
上述内容解释：
这段配置是Envoy代理服务器的配置片段，主要描述了管理和监听器的配置以及静态资源中的监听器和集群设定。下面是每部分的详细解释：

### Admin Interface 配置
```yaml
admin:
  profile_path: /tmp/envoy.prof           # Envoy性能剖析数据的输出路径。
  access_log_path: /tmp/admin_access.log  # Envoy管理接口的访问日志文件路径。
  address:
    socket_address:
       address: 0.0.0.0                   # 监听所有可用网络接口。
       port_value: 9901                   # Envoy管理接口监听的端口号。
````
