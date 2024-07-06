# 容器访问外网

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
docker run -d --name web1 -p 8081:80 nginx:latest
```

```
iptables -t nat -vnL POSTROUTING
```

```
Chain POSTROUTING (policy ACCEPT 7 packets, 766 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80
```
