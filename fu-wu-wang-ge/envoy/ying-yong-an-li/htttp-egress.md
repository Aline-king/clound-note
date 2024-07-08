# 🫡 htttp egress

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```
[root@dockerhost-envoy ~]# mkdir envoy_http_egress
[root@dockerhost-envoy ~]# cd envoy_http_egress/
```



<details>

<summary>docker-compose.yaml</summary>

```yaml
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
        ipv4_address: 172.23.1.2
        aliases:
        - front-proxy
    depends_on:
    - webserver01
    - webserver02
  client:
    image: www.kubemsb.com/envoy/admin-toolbox:v1.0
    network_mode: "service:envoy"
    depends_on:
    - envoy
  webserver01:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    hostname: webserver01
    networks:
      envoymesh:
        ipv4_address: 172.23.1.3
        aliases:
        - webserver01
  webserver02:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    hostname: webserver02
    networks:
      envoymesh:
        ipv4_address: 172.23.1.4
        aliases:
        - webserver02
networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.23.1.0/24
```



</details>

<details>

<summary>envoy</summary>

```
# vim envoy.yaml
# cat envoy.yaml
```

<pre class="language-yaml"><code class="lang-yaml"><strong>static_resources:
</strong>  listeners:
  - name: listener_0
    address:
      socket_address: { address: 127.0.0.1, port_value: 80 }
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
                route: { cluster: web_cluster }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
  clusters:
  - name: web_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: web_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: 172.23.1.3, port_value: 80 }
        - endpoint:
            address:
              socket_address: { address: 172.23.1.4, port_value: 80 }
</code></pre>

![](<../../../.gitbook/assets/image (6).png>)

</details>

<details>

<summary>运行并测试</summary>

docker-compose up

```bash
# docker-compose up
[+] Running 3/3
 ✔ client 2 layers [⣿⣿]      0B/0B      Pulled                                             0.9s
   ✔ c9b1b535fdd9 Already exists                                                           0.0s
   ✔ 1de2c4c6c672 Pull complete                                                            0.1s
[+] Running 5/3
 ✔ Network envoy_http_egress_envoymesh        Cre...                                       0.0s
 ✔ Container envoy_http_egress-webserver02-1  Created                                      0.1s
 ✔ Container envoy_http_egress-webserver01-1  Created                                      0.1s
 ✔ Container envoy_http_egress-envoy-1        Cre...                                       0.0s
 ✔ Container envoy_http_egress-client-1       Cr...                                        0.0s
Attaching to client-1, envoy-1, webserver01-1, webserver02-1
webserver01-1  |  * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
webserver02-1  |  * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

```powershell
[root@dockerhost-envoy ~]# docker ps
CONTAINER ID   IMAGE                                      COMMAND                   CREATED          STATUS          PORTS       NAMES
f66cee88e26a   www.kubemsb.com/envoy/admin-toolbox:v1.0   "/bin/sh -c 'sleep 9…"   42 seconds ago   Up 41 seconds               envoy_http_egress-client-1
422fe27d14a3   envoyproxy/envoy:v1.30.1                   "/docker-entrypoint.…"   42 seconds ago   Up 41 seconds   10000/tcp   envoy_http_egress-envoy-1
a8a96135975d   www.kubemsb.com/envoy/demoapp:v1.0         "/bin/sh -c 'python3…"   42 seconds ago   Up 41 seconds               envoy_http_egress-webserver01-1
b6698f0eefad   www.kubemsb.com/envoy/demoapp:v1.0         "/bin/sh -c 'python3…"   42 seconds ago   Up 41 seconds               envoy_http_egress-webserver02-1
```

```powershell
[root@dockerhost-envoy ~]# docker exec -it envoy_http_egress-client-1  /bin/sh

[root@422fe27d14a3 /]# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.23.1.2      422fe27d14a3

[root@422fe27d14a3 /]# curl http://127.0.0.1
demoapp v1.0 !! ClientIP: 172.23.1.2, ServerName: webserver01, ServerIP: 172.23.1.3!

[root@422fe27d14a3 /]# curl http://127.0.0.1
demoapp v1.0 !! ClientIP: 172.23.1.2, ServerName: webserver02, ServerIP: 172.23.1.4!
```

</details>
