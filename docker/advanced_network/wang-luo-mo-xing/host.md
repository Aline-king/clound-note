# host

```bash
查看host类型的网络模型
# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
......
d04ce0d0e6ca   host      host      local
......
```

```
查看host网络模型的详细信息
# docker network inspect host
[
    {
        "Name": "host",
        "Id": "d04ce0d0e6ca8e6226937f19033ef2c3f05b47ed63e06492d5c3071904fbb80b",
        "Created": "2022-01-21T16:12:05.30970114+08:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

```
创建容器使用host网络模型，并查看其网络信息
# docker run -it --network host --rm busybox
/ # ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:11:B8:9A:C5
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:11ff:feb8:9ac5/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:53 errors:0 dropped:0 overruns:0 frame:0
          TX packets:94 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:6924 (6.7 KiB)  TX bytes:7868 (7.6 KiB)
​
docker1   Link encap:Ethernet  HWaddr 02:42:14:AA:F5:04
          inet addr:192.168.100.1  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
​
​
ens33     Link encap:Ethernet  HWaddr 00:0C:29:AF:89:0B
          inet addr:192.168.255.161  Bcast:192.168.255.255  Mask:255.255.255.0
          inet6 addr: fe80::44fc:2662:bfab:2b93/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:157763 errors:0 dropped:0 overruns:0 frame:0
          TX packets:50865 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:205504721 (195.9 MiB)  TX bytes:3626119 (3.4 MiB)
​
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:88 errors:0 dropped:0 overruns:0 frame:0
          TX packets:88 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:8196 (8.0 KiB)  TX bytes:8196 (8.0 KiB)
​
virbr0    Link encap:Ethernet  HWaddr 52:54:00:EB:01:E5
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
​
/ # exit
```

**运行Nginx服务**

{% tabs %}
{% tab title="First Tab" %}
```
创建用于运行nginx应用的容器，使用host网络模型
# docker run -d --network host nginx:latest
```


{% endtab %}

{% tab title="Second Tab" %}
```
查看容器运行状态
# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
f6677b213271   nginx:latest   "/docker-entrypoint.…"   7 seconds ago   Up 6 seconds             youthful_shtern
```

```
查看docker host 80端口状态
# ss -anput | grep ":80"
tcp    LISTEN     0      511       *:80                    *:*                   users:(("nginx",pid=42866,fd=7),("nginx",pid=42826,fd=7))
tcp    LISTEN     0      511      :::80                   :::*                   users:(("nginx",pid=42866,fd=8),("nginx",pid=42826,fd=8))
```


{% endtab %}

{% tab title="Untitled" %}
```
使用curl命令访问docker host主机IP地址，验证是否可以对nginx进行访问，如可访问，则说明容器与docker host共享网络命名空间
# curl http://192.168.255.161
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
​
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
​
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


{% endtab %}

{% tab title="Untitled" %}
<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}



\
