# 简介

![](../\_images/etcd\_logo.png)

`etcd` 是 `CoreOS` 团队于 2013 年 6 月发起的开源项目，

它的目标是构建一个高可用的分布式键值（`key-value`）数据库，基于 `Go` 语言实现。

etcd内部采用[`raft`](../../yi-zhi-xing-suan-fa/raft.md)协议作为一致性算法

我们知道，在分布式系统中，各种服务的配置信息的管理分享，服务的发现是一个很基本同时也是很重要的问题。`CoreOS` 项目就希望基于 `etcd` 来解决这一问题。

[`github.com/etcd-io/etcd`](https://github.com/etcd-io/etcd)

受到 [Apache ZooKeeper](https://zookeeper.apache.org/) 项目和 [doozer](https://github.com/ha/doozerd) 项目的启发，`etcd` 在设计的时候重点考虑了下面四个要素：

* 简单：具有定义良好、面向用户的 `API` ([gRPC](https://github.com/grpc/grpc))
* 安全：支持 `HTTPS` 方式的访问
* 快速：支持并发 `10 k/s` 的写操作
* 可靠：支持分布式结构，基于 `Raft` 的一致性算法

