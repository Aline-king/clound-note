# ✴️ http ingress

```
# vim docker-compose.yaml
# cat docker-compose.yaml
```

{% tabs %}
{% tab title="docker-compose" %}
```yaml
version: '3'

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
        ipv4_address: 172.22.1.2
        aliases:
        - ingress

  webserver01:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - PORT=8080
      - HOST=127.0.0.1
    network_mode: "service:envoy"
    depends_on:
    - envoy

networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.1.0/24
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>


{% endtab %}

{% tab title="envoy" %}
```yaml
# cat envoy.yaml
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

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}



<details>

<summary>运行并测试</summary>

```bash
# docker-compose up
[+] Running 3/0
 ✔ Network envoy_http_ingress_envoymesh        C...         0.0s
 ✔ Container envoy_http_ingress-envoy-1        C...         0.0s
 ✔ Container envoy_http_ingress-webserver01-1  Created      0.0s
Attaching to envoy-1, webserver01-1
webserver01-1  |  * Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)
```

```bash
[root@dockerhost-envoy ~]# docker ps
CONTAINER ID   IMAGE                                COMMAND                   CREATED         STATUS         PORTS       NAMES
62997bf0dc2a   www.kubemsb.com/envoy/demoapp:v1.0   "/bin/sh -c 'python3…"   8 seconds ago   Up 7 seconds               envoy_http_ingress-webserver01-1
b23507f2767d   envoyproxy/envoy:v1.30.1             "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds   10000/tcp   envoy_http_ingress-envoy-1     http-ingress-envoy-1
```

```
[root@dockerhost-envoy ~]# docker exec -it 6299 /bin/sh
[root@b23507f2767d /]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
20: eth0@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:16:01:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.22.1.2/24 brd 172.22.1.255 scope global eth0
       valid_lft forever preferred_lft forever

```

```
[root@dockerhost-envoy ~]# curl http://172.22.1.2
demoapp v1.0 !! ClientIP: 127.0.0.1, ServerName: b23507f2767d, ServerIP: 172.22.1.2!
```

docker-compose down

</details>
