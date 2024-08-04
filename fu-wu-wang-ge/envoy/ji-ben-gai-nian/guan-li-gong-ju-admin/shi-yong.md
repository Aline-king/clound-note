# 使用

```powershell
[root@dockerhost-envoy ~]# mkdir envoy_admin
[root@dockerhost-envoy ~]# cd envoy_admin/
```

## 部署文件

{% tabs %}
{% tab title="docker-compose" %}
该配置文件旨在部署一个包含Envoy代理和两个后端Web服务器的微服务架构，通过自定义网络`envoymesh`实现服务间的通信，Envoy作为前端代理处理进入的流量并将其导向后端Web服务器。

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

<figure><img src="../../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>


{% endtab %}

{% tab title="envoy" %}
这段配置是Envoy代理服务器的配置片段，主要描述了管理和监听器的配置以及静态资源中的监听器和集群设定。下面是每部分的详细解释：

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

<pre class="language-yaml"><code class="lang-yaml"><strong># Admin Interface 配置
</strong>admin:
  profile_path: /tmp/envoy.prof           # Envoy性能剖析数据的输出路径。
  access_log_path: /tmp/admin_access.log  # Envoy管理接口的访问日志文件路径。
  address:
    socket_address:
       address: 0.0.0.0                   # 监听所有可用网络接口。
       port_value: 9901                   # Envoy管理接口监听的端口号。
</code></pre>
{% endtab %}
{% endtabs %}

