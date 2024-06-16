# 容器访问外网

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
# iptables -t nat -vnL DOCKER
```

```
输出：
Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8081 to:172.17.0.2:80
```

\
