# 事例

```powershell
[root@dockerhost-envoy ~]# mkdir envoy_runtime
[root@dockerhost-envoy ~]# cd envoy_runtime/
```



<details>

<summary>docker-compose</summary>

```powershell
# vim docker-compose.yaml
# cat docker-compose.yaml
version: '3.3'
services:
  envoy:
    image: envoyproxy/envoy:v1.30.1
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    networks:
      envoymesh:
        ipv4_address: 172.26.1.2
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
        ipv4_address: 172.26.1.3
        aliases:
        - webserver01
  webserver02:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - PORT=8080
    hostname: webserver02
    networks:
      envoymesh:
        ipv4_address: 172.26.1.4
        aliases:
        - webserver02
networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.26.1.0/24
```



</details>

<details>

<summary>envoy</summary>

```powershell
# vim envoy.yaml
# cat envoy.yaml
admin:
  profile_path: /tmp/envoy.prof
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
       address: 0.0.0.0
       port_value: 9901

layered_runtime:
  layers:
  - name: static_layer_0
    static_layer:
      health_check:
        min_interval: 5
  - name: admin_layer_0
    admin_layer: {}

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
              socket_address: { address: 172.26.1.3, port_value: 8080 }
        - endpoint:
            address:
              socket_address: { address: 172.26.1.4, port_value: 8080 }
```

这段配置是Envoy代理的配置片段，主要涉及管理接口（Admin）、运行时层配置（Layered Runtime）以及静态资源定义（Static Resources），包括监听器（Listeners）和集群（Clusters）。下面是各部分的详细解释：

</details>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<details>

<summary>运行并测试</summary>



</details>

这条查询结果显示了Envoy的健康检查最小间隔时间被设定为5秒，且这一配置是在`static_layer_0`中设定的，而`admin_layer_0`没有对此配置进行覆盖或修改。这样的查询对于理解和调试Envoy的动态配置非常有用。

```
# curl 172.26.1.2:9901/runtime
{
 "entries": {  
  "health_check.min_interval": {
   "final_value": "5",
   "layer_values": [
    "5",
    ""
   ]
  }
 },
 "layers": [
  "static_layer_0",
  "admin_layer_0"
 ]
}

```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
# curl -XPOST 172.26.1.2:9901/runtime_modify?Hi=true
OK
```

{% hint style="info" %}
runtime\_modify：是Envoy管理接口中的路径，用于修改运行时配置。这个路径不是标准的Envoy API路径；标准的修改运行时配置路径应该是`/runtime/admin`，但这里可能是为了简化说明或者基于特定环境的自定义路径。

?Hi=true`: 是附加在URL后的查询参数。在这里，`Hi`作为一个自定义的键，其值被设为`true`。在Envoy的标准用法中，修改运行时配置通常需要指定配置项的层级路径和新值，但这里的`Hi\`看起来像是一个简化的示例或者特定场景下的自定义配置项。
{% endhint %}

执行后输出的`OK`表明该POST请求成功执行，意味着请求被接收并且处理成功。如果这个`Hi`参数对应着Envoy内部某个可修改的运行时配置项，并且该配置项支持动态修改，那么这个请求就改变了Envoy的运行时行为。然而，由于`Hi`并不是Envoy标准配置项的一部分，这个示例是一个教学或演示目的的构造，旨在展示如何通过API修改运行时配置，而不是反映实际的Envoy配置操作。



这段内容依然是通过`curl`命令查询Envoy管理接口（位于IP地址`172.26.1.2`，端口`9901`）上`/runtime`端点所返回的运行时配置信息，以JSON格式展示。让我们逐行解析这个JSON结构：

```
# curl 172.26.1.2:9901/runtime
{
 "entries": { # 包含了各个运行时配置项的详细情况。
  "Hi": { # 这是一个新的运行时配置项，是之前通过某种方式动态添加或修改的。 表明`Hi`配置项在不同配置层中的设定值。
   "layer_values": [
    "", # 第一个值（空字符串）来自`static_layer_0`，表示这一层没有定义此配置项；
    "true" # 第二个值`"true"`来自`admin_layer_0`，这是该配置项最终取值的地方。
   ],
   "final_value": "true" # 确认了`Hi`配置项在所有层次合并之后的最终有效值为`true`。
  },
  "health_check.min_interval": {
   "final_value": "5",
   "layer_values": [
    "5",
    ""
   ]
  }
 },
 "layers": [
  "static_layer_0",
  "admin_layer_0"
 ]
}
```

综上所述，这次查询结果显示了Envoy的运行时配置中新增了一个名为`Hi`的配置项，其值为`true`，这是通过某次修改（可能就是之前提到的`curl -XPOST ... runtime_modify?Hi=true`命令）设置的。同时，原有的`health_check.min_interval`配置保持不变，进一步展示了Envoy如何动态管理和应用不同层次的配置设置
