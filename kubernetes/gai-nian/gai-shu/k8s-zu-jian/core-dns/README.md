# core DNS

coredns是一个用go语言编写的开源的DNS服务，coredns是首批加入CNCF组织的云原生开源项目，并且作为已经在CNCF毕业的项目，coredns还是目前kubernetes中默认的dns服务。同时，由于coredns可以集成插件，它还能够实现服务发现的功能。

coredns和其他的诸如bind、knot、powerdns、unbound等DNS服务不同的是：coredns非常的灵活，并且几乎把所有的核心功能实现都外包给了插件。如果你想要在coredns中加入Prometheus的监控支持，那么只需要安装对应的prometheus插件并且启用即可。

配置解析

```
coredns的配置依然是存放在 configmap中
[root@kubernetes-master1 ~]# kubectl get cm coredns -n kube-system
NAME      DATA   AGE
coredns   1      2d2h
```

```
查看配置详情
[root@kubernetes-master1 ~]# kubectl describe cm coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>
​
Data
====
Corefile:
----
.:53 {
    errors
    health {                                            # 健康检测
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {    # 解析配置
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {                        # 转发配置  
       max_concurrent 1000
    }
    cache 30
    loop
    reload                                              # 自动加载
    loadbalance
}
​
BinaryData
====
​
Events:  <none>
​
```

```
其他属性和示例：
    except  domain                  排除的域名
​
添加dns解析
    hosts {
        192.168.8.100 www.example.com
        fallthrough         # 在CoreDNS里面表示如果自己无法处理，则交由下个插件处理。
    }
​
```
