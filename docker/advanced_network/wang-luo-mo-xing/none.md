# None

#### 4.2.3 none

```
查看none类型的网络模型
# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
......
a369d8e58a41   none      null      local
```

```
查看none网络模型详细信息
# docker network inspect none
[
    {
        "Name": "none",
        "Id": "a369d8e58a41ce2e3c25f2273b059e984dd561bfa7e79077a0cce9b3a925b9c9",
        "Created": "2022-01-21T16:12:05.217801814+08:00",
        "Scope": "local",
        "Driver": "null",
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
创建容器使用none网络模型，并查看其网络状态
# docker run -it --network none --rm busybox:latest
/ # ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
​
/ # exit
```

\
