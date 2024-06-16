# stateful  Set

StatefulSets 旨在与有状态的应用及分布式系统一起使用。然而在 Kubernetes 上管理有状态应用和分布式系统是一个宽泛而复杂的话题。StatefulSet有以下特点[^1]：

* 每个Pod都有稳定、唯一的网络标识，彼此间可以通信
* StatefulSet控制的Pod副本启动、扩展、删除、更新等操作都是有顺序的。
* StatefulSet里的Pod采用独立的持久化存储卷，存储状态数据

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

[Headless service](#user-content-fn-2)[^2]会给一个StatufulSet控制的Pod提供一个唯一的DNS域名来作为每个成员的网络标识，集群内部成员之间使用域名通信。

DNS域名格式： $(podname).$(headless service name).$(namespace name).svc.cluster.local

[^1]: 使用场景

[^2]: 无头服务
