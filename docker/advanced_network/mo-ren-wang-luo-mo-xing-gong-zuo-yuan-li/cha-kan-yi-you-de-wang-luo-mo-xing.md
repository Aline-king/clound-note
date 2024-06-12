# 查看已有的网络模型

<pre class="language-bash" data-title="查看已有的网络模型"><code class="lang-bash"><strong># docker network ls
</strong>NETWORK ID     NAME      DRIVER    SCOPE
a26c79961d8c   bridge    bridge    local
d04ce0d0e6ca   host      host      local
a369d8e58a41   none      null      local
</code></pre>

{% code title="查看已有网络模型详细信息" %}
```bash
# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "a26c79961d8c3a5f66a7de782b773291e4902badc60d0614745e01b18f506907",
        "Created": "2022-02-08T11:45:25.607195911+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "dbac5dd601b960c91bee8fafcabc0a6e6091bff14d5fccfa80ca2c74df8891ad": {
                "Name": "web1",
                "EndpointID": "2c1d8c66f7f46d6d76e5c384b1729a90441e1398496b3112124ba65d255432a1",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
{% endcode %}

{% code title="查看docker支持的网络模型" %}
```
# docker info | grep Network
  Network: bridge host ipvlan macvlan null overlay
```
{% endcode %}

\
