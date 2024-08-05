# http front proxy

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
[root@dockerhost-envoy ~]# mkdir envoy_http_front_proxy 
[root@dockerhost-envoy ~]# cd envoy_http_front_proxy/
```



<details>

<summary>docker-compose</summary>

<pre class="language-yaml"><code class="lang-yaml"><strong>version: '3.3'
</strong>services:
  envoy:
    image: envoyproxy/envoy:v1.30.1
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    networks:
      envoymesh:
        ipv4_address: 172.24.1.2
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
        ipv4_address: 172.24.1.3
        aliases:
        - webserver01
  webserver02:
    image: www.kubemsb.com/envoy/demoapp:v1.0
    environment:
      - PORT=8080
    hostname: webserver02
    networks:
      envoymesh:
        ipv4_address: 172.24.1.4
        aliases:
        - webserver02
networks:
  envoymesh:
    driver: bridge
    ipam:
      config:
        - subnet: 172.24.1.0/24
</code></pre>



</details>

<details>

<summary>envoy</summary>

```
# vim envoy.yaml

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
              socket_address: { address: 172.24.1.3, port_value: 8080 }
        - endpoint:
            address:
              socket_address: { address: 172.24.1.4, port_value: 8080 }

```



</details>



<details>

<summary>运行并测试</summary>



docker-compose up

curl -H "host: www.kubemsb.com" 172.24.1.2\
curl -I -H "host: www.kubex.com" 172.24.1.2

```powershell
# docker-compose up
[+] Running 4/0
 ✔ Network envoy_http_front_proxy_envoymesh        Created                                 0.0s
 ✔ Container envoy_http_front_proxy-webserver02-1  Created                                 0.0s
 ✔ Container envoy_http_front_proxy-webserver01-1  Created                                 0.0s
 ✔ Container envoy_http_front_proxy-envoy-1        Created                                 0.0s
Attaching to envoy-1, webserver01-1, webserver02-1
webserver01-1  |  * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
webserver02-1  |  * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
envoy-1        | [2024-05-23 14:07:55.523][1][info][main] [source/server/server.cc:428] initializing epoch 0 (base id=0, hot restart version=11.104)
```

```powershell
# docker ps
CONTAINER ID   IMAGE                                COMMAND                   CREATED          STATUS          PORTS       NAMES
8490437ed506   envoyproxy/envoy:v1.30.1             "/docker-entrypoint.…"   38 seconds ago   Up 38 seconds   10000/tcp   envoy_http_front_proxy-envoy-1
154519832fc0   www.kubemsb.com/envoy/demoapp:v1.0   "/bin/sh -c 'python3…"   38 seconds ago   Up 38 seconds               envoy_http_front_proxy-webserver01-1
1dd6efae4f46   www.kubemsb.com/envoy/demoapp:v1.0   "/bin/sh -c 'python3…"   38 seconds ago   Up 38 seconds               envoy_http_front_proxy-webserver02-1
```

```powershell
# docker inspect 849043
.......
   "Networks": {
                "envoy_http_front_proxy_envoymesh": {
                    "IPAMConfig": {
                        "IPv4Address": "172.24.1.2"
                    },
                    "Links": null,
                    "Aliases": [
                        "envoy_http_front_proxy-envoy-1",
                        "envoy",
                        "front-proxy"
                    ],
                    "MacAddress": "02:42:ac:18:01:02",
                    "NetworkID": "a80ac1008ee8acafecfbe1d3d9e6e9253e2692634927724c49badb2c3b2f9776",
                    "EndpointID": "3d09e8482b51556bcd0c2617329164e09fe7829d7cf958928e309e168ab88337",
                    "Gateway": "172.24.1.1",
                    "IPAddress": "172.24.1.2",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DriverOpts": null,
                    "DNSNames": [
                        "envoy_http_front_proxy-envoy-1",
                        "envoy",
                        "front-proxy",
                        "8490437ed506"
                    ]
                }
            }
        }
    }
]
```

```powershell
[root@dockerhost-envoy ~]# curl -H "Host: www.kubemsb.com" http://172.24.1.2
demoapp v1.0 !! ClientIP: 172.24.1.2, ServerName: webserver02, ServerIP: 172.24.1.4!

[root@dockerhost-envoy ~]# curl -H "Host: www.kubemsb.com" http://172.24.1.2
demoapp v1.0 !! ClientIP: 172.24.1.2, ServerName: webserver01, ServerIP: 172.24.1.3!
```

```powershell
# curl -H "Host: www.kubex.com" http://172.24.1.2
看不到任何响应数据

# curl -I -H "Host: www.kubex.com" http://172.24.1.2
HTTP/1.1 301 Moved Permanently
location: http://www.kubemsb.com/
date: Thu, 23 May 2024 14:13:03 GMT
server: envoy
transfer-encoding: chunked
```

```powershell
# curl -vL -H "Host: www.kubex.com" http://172.24.1.2
* About to connect() to 172.24.1.2 port 80 (#0)
*   Trying 172.24.1.2...
* Connected to 172.24.1.2 (172.24.1.2) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Accept: */*
> Host: www.kubex.com
>
< HTTP/1.1 301 Moved Permanently
< location: http://www.kubemsb.com/
< date: Thu, 23 May 2024 14:14:11 GMT
< server: envoy
< content-length: 0
<
* Connection #0 to host 172.24.1.2 left intact
* Issue another request to this URL: 'http://www.kubemsb.com/'
* About to connect() to www.kubemsb.com port 80 (#1)
*   Trying 192.168.10.191...
* Connected to www.kubemsb.com (192.168.10.191) port 80 (#1)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.kubemsb.com
> Accept: */*
>
< HTTP/1.1 308 Permanent Redirect
< Server: nginx/1.25.2
< Date: Thu, 23 May 2024 14:14:11 GMT
< Content-Type: text/html
< Content-Length: 171
< Connection: keep-alive
< Location: https://www.kubemsb.com:443/
<

如果www.kubemsb.com对应一个web站点，则可以进行访问。
```

</details>
