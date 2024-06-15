# metrics

在k8s中他们主要是以指标的样式来获取的，这些包括核心指标和自定义指标。

在k8s的系统上包含各种各样的指标数据，早期的k8s系统，为[kubelet](../../../zu-jian/kubelet/)集成了一个 [CAdvsior](../../../zu-jian/kubelet/cadvsior.md)工具可以获取kubelet所在节点上的相关指标，包括容器指标。

但是CAdvsior的缺陷在于，我们仅能够获取，指定节点上的指标信息，而无法获取集群管理的统一指标。

<table data-header-hidden><thead><tr><th width="147">方式</th><th>解析</th></tr></thead><tbody><tr><td>监控代理程序</td><td>如node_exporter，收集标准的主机指标数据，包括平均负载、CPU、Memory、Disk、Network及诸多其他维度的数据</td></tr><tr><td>kubelet</td><td>收集容器指标数据，它们也是Kubernetes“核心指标”，每个容器的相关指标数据主要有CPU利用率（user和system）及限额、文件系统读/写/限额、内存利用率及限额、网络报文发送/接收/丢弃速率等。</td></tr><tr><td>API Server</td><td>收集API Server的性能指标数据，包括控制工作队列的性能、请求速率与延迟时长、etcd缓存工作队列及缓存性能、普通进程状态（文件描述符、内存、CPU等）、Golang状态（GC、内存和线程等）</td></tr><tr><td>etcd</td><td>收集etcd存储集群的相关指标数据，包括领导节点及领域变动速率、提交/应用/挂起/错误的提案次数、磁盘写入性能、网络与gRPC计数器等</td></tr><tr><td>kube-state-metrics</td><td>该组件用于根据Kubernetes API Server中的资源派生出多种资源指标，它们主要是资源类型相关的计数器和元数据信息，包括指定类型的对象总数、资源限额、容器状态（ready/restart/running/terminated/waiting）以及Pod资源的标签系列等</td></tr></tbody></table>





