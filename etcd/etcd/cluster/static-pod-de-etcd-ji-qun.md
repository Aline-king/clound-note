---
description: 特点：折中方式。
---

# static pod的etcd集群



将多台Kubernetes Master上的静态pod组成etcd集群，各个服务器的etcd实例被注册进了Kubernetes当中，虽然无法直接使用kubectl来管理这部分实例，但是监控以及日志搜集组件均可正常工作。在这一模式运行下的etcd可管理性更强。
