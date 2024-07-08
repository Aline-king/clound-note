# 四层TCP转发

```
[root@dockerhost-envoy ~]# mkdir envoy_tcp
[root@dockerhost-envoy ~]# cd envoy_tcp/
```

{% hint style="info" %}
## 准备工作

```bash


```

{% tabs %}
{% tab title="创建docker-compose运行文件" %}
<details>

<summary>文件内容</summary>

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
        ipv4_address: 172.21.1.254
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
        ipv4_address: 172.21.1.2
        aliases:
        - webserver01

  webserver02:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - PORT=8080
    hostname: webserver02
    networks:
      envoymesh:
        ipv4_address: 172.21.1.3
        aliases:
        - webserver02

networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.1.0/24
```

</details>

<mark style="color:purple;">**说明**</mark>：

<img alt="" class="gitbook-drawing">

用于定义和配置一个由多个服务（services）和网络（networks）组成的多容器 Docker 应用程序。

版本号 `version: '3.3'` 指定了使用的 Docker Compose 文件格式版本。

{% tabs %}
{% tab title="Envoy" %}
* **Image**: 使用的镜像是 `envoyproxy/envoy:v1.30.1`，这是Envoy代理的一个特定版本，常用于服务网格、API网关等场景，提供服务发现、负载均衡、TLS终止等功能。
* **Volumes**: 将主机上的 `./envoy.yaml` 文件挂载到容器内的 `/etc/envoy/envoy.yaml`，这个配置文件告诉Envoy如何路由请求、管理服务发现等。
* **Networks**: 加入名为 `envoymesh` 的网络，并指定静态IP地址为 `172.21.1.254`，同时给它一个别名 `front-proxy`，便于识别和路由。
* **Depends\_on**: 表示Envoy服务依赖于 `webserver01` 和 `webserver02` 启动。这意味着在启动Envoy之前，这两个Web服务器必须已经启动并运行。
{% endtab %}

{% tab title="Webserver01 & Webserver02" %}
* **Image**: 两个Web服务器使用相同的镜像`www.kubemsb.com/envoy/demoapp:v1.0`，这是一个示例应用的镜像，用于演示或测试Envoy的配置。
* **Environment**: 设置环境变量 `PORT=8080`，指示应用监听8080端口。
* **Hostname**: 分别为每个Web服务器指定了不同的主机名 (`webserver01`, `webserver02`)。
* **Networks**: 同样加入 `envoymesh` 网络，但各自有不同的静态IP地址（分别为 `172.21.1.2` 和 `172.21.1.3`），以及对应的别名以便识别。
{% endtab %}

{% tab title="网络（Networks）" %}
**EnvoyMesh**: 定义了一个名为 `envoymesh` 的自定义网络，使用桥接模式 (`driver: bridge`)，这意味着这个网络与宿主机的网络是隔离的，但容器间可以相互通信。

`ipam` 部分配置了网络的IP地址管理，指定了一个子网 `172.21.1.0/24`，这为网络中的容器分配了IP地址空间。
{% endtab %}
{% endtabs %}
{% endtab %}

{% tab title="创建envoy配置文件" %}
<details>

<summary>文件内容</summary>

```powershell
# vim envoy.yaml
# cat envoy.yaml
static_resources:
  listeners:
    name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy
          stat_prefix: tcp
          cluster: local_cluster

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
              socket_address: { address: 172.21.1.2, port_value: 8080 }
        - endpoint:
            address:
              socket_address: { address: 172.21.1.3, port_value: 8080 }
```



</details>

说明：

Envoy配置文件：主要描述了静态资源配置（`static_resources`），包括监听器（`listeners`）和集群（`clusters`）的配置。

<mark style="color:orange;">**static\_resources**</mark>

{% tabs %}
{% tab title="Listeners (监听器)" %}
<details>

<summary>这是折叠块标题提</summary>



</details>
{% endtab %}

{% tab title="OSX" %}
Here are the instructions for macOS
{% endtab %}

{% tab title="Linux" %}
Here are the instructions for Linux
{% endtab %}
{% endtabs %}
{% endtab %}

{% tab title="Windows" %}
2222
{% endtab %}
{% endtabs %}


{% endhint %}



<pre class="language-powershell"><code class="lang-powershell">上述文件说明：
<strong>
</strong>
# 

# 
- **name**: 监听器的名称，这里是 `listener_0`。
- **address**: 监听地址配置，`socket_address` 表示监听所有接口（`0.0.0.0`）上的端口 `80`，即HTTP默认端口。
- **filter_chains**:
  - 过滤链定义了到达监听器的流量如何被处理。这里只有一个过滤链。
    - **filters**: 过滤器列表。
      - **name**: 过滤器名称为 `envoy.tcp_proxy`，表示使用TCP代理过滤器来处理TCP连接。
      - **typed_config**: 配置该过滤器的具体类型和参数。
        - `@type`: 指定配置的类型，这里是Envoy TCP代理的特定类型。
        - **stat_prefix**: 统计前缀，用于标识与该TCP代理相关的统计信息。
        - **cluster**: 指定流量应转发到的集群名称，这里是 `local_cluster`。

# Clusters (集群)
- **name**: 集群名称，`local_cluster`。
- **connect_timeout**: 与集群内主机建立连接的超时时间，设为0.25秒。
- **type**: 集群类型，这里是 `STATIC`，意味着集群成员是静态配置的，而非动态发现。
- **lb_policy**: 负载均衡策略，采用 `ROUND_ROBIN`，即轮询策略，均匀地将请求分配给各成员。
- **load_assignment**: 负载分配配置。
  - **cluster_name**: 对应的集群名称，再次确认为 `local_cluster`。
  - **endpoints**: 集群中的终端点（后端实例）列表。
    - **lb_endpoints**: 负载均衡终端点列表。
      - 第一个终端点配置：地址为 `172.21.1.2`，端口为 `8080`。
      - 第二个终端点配置：地址为 `172.21.1.3`，端口同样为 `8080`。

该配置定义了一个监听在所有接口的80端口上的TCP监听器，使用TCP代理过滤器将接收到的连接路由到名为 `local_cluster` 的静态集群，该集群包含两个后端服务（`172.21.1.2:8080` 和 `172.21.1.3:8080`），并以轮询的方式进行负载均衡。
</code></pre>

**部署说明文档**

```powershell
# cat README.md
# TCP Proxy

### 环境说明
三个Service:
- envoy：Front Proxy,地址为172.21.1.254
- webserver01：第一个后端服务,地址为172.21.1.2
- webserver02：第二个后端服务,地址为172.21.1.3

### 运行和测试
1. 创建
```

docker-compose up

```
2. 测试
```

curl 172.21.1.254

```
3. 停止后清理
```

```
docker-compose down
```

```powershell
[root@dockerhost-envoy envoy_tcp]# ls
docker-compose.yaml  envoy.yaml
```

```powershell
# docker-compose up
[+] Running 17/17
 ✔ webserver02 5 layers [⣿⣿⣿⣿⣿]      0B/0B      Pulled                                     4.4s
   ✔ c9b1b535fdd9 Pull complete                                                            0.1s
   ✔ 3cbce035cd7c Pull complete                                                            0.2s
   ✔ b83463f478a5 Pull complete                                                            0.1s
   ✔ 34b1f286d5e2 Pull complete                                                            0.1s
   ✔ 6331f5dc1421 Pull complete                                                            0.1s
 ✔ envoy 9 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                      18.9s
   ✔ 3c645031de29 Pull complete                                                            4.4s
   ✔ 7ab97aeb917d Pull complete                                                            2.2s
   ✔ 2a39cc7bea2e Pull complete                                                            2.3s
   ✔ 7516628aee78 Pull complete                                                            5.6s
   ✔ 13639560348b Pull complete                                                            4.5s
   ✔ 5806e3fdc9eb Pull complete                                                            6.9s
   ✔ 68a59cac32b1 Pull complete                                                            6.6s
   ✔ 88dd63b8ec5a Pull complete                                                            9.5s
   ✔ 4f4fb700ef54 Pull complete                                                            9.0s
 ✔ webserver01 Pulled                                                                      4.4s
[+] Running 4/3
 ✔ Network envoy_tcp_envoymesh        Created                                              0.0s
 ✔ Container envoy_tcp-webserver02-1  Created                                              0.2s
 ✔ Container envoy_tcp-webserver01-1  Created                                              0.2s
 ✔ Container envoy_tcp-envoy-1        Created                                              0.0s
Attaching to envoy-1, webserver01-1, webserver02-1
webserver02-1  |  * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
webserver01-1  |  * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
envoy-1        | [2024-05-23 06:39:54.186][1][info][main] [source/server/server.cc:428] initializing epoch 0 (base id=0, hot restart version=11.104)
```

```powershell
[root@dockerhost-envoy ~]# curl http://172.21.1.254/
demoapp v1.0 !! ClientIP: 172.21.1.254, ServerName: webserver02, ServerIP: 172.21.1.3!


[root@dockerhost-envoy ~]# curl http://172.21.1.254/
demoapp v1.0 !! ClientIP: 172.21.1.254, ServerName: webserver01, ServerIP: 172.21.1.2!
```

```powershell
查看相关的日志
webserver02-1  | 172.21.1.254 - - [23/May/2024 06:58:59] "GET / HTTP/1.1" 200 -
webserver01-1  | 172.21.1.254 - - [23/May/2024 06:59:06] "GET / HTTP/1.1" 200 -
```

```powershell
# docker-compose down
```
