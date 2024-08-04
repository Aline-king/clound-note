# 功能和对象

Istio的很多功能在k8s上面都是以[CRD](https://y04h4pmzvxx.feishu.cn/wiki/Ebyhw2gSMiuovskxFeQcQbzdnrf) 的样式存在的，包括各种流量的路由规则和控制策略等，这些内容都会被kube-apiserver检验之后，存储到后端的Etcd中。这些自定义的CRD对象，

在Istio中，主要是由Galley负责从kube-apiserver中加载的。 默认情况下，Istio提供了50多种CRD资源对象，这些资源对象包括Network、Authentication、Config等群组。

网络相关的资源对象（全称为 名称.networking.istio.io）

<table data-header-hidden><thead><tr><th width="140">名称</th><th width="115">分类</th><th width="444">用途</th><th>归属</th></tr></thead><tbody><tr><td>VirtualService</td><td>networking</td><td>VirtualService 定义了一系列针对指定服务的流量路由规则。每个路由规则都针对特定协议的匹配规则。如果流量符合这些特征，就会根据规则发送到服务注册表中的目标服务（或者目标服务的子集或版本）</td><td>pilot</td></tr><tr><td>DestinationRule</td><td>networking</td><td>DestinationRule 所定义的策略，决定了经过路由处理之后的流量的访问策略。这些策略中可以定义负载均衡配置、连接池尺寸以及外部检测（用于在负载均衡池中对不健康主机进行识别和驱逐）配置</td><td>pilot</td></tr><tr><td>Gateway</td><td>networking</td><td>Gateway 描述了一个负载均衡器，用于承载网格边缘的进入和发出连接。这一规范中描述了一系列开放端口，以及这些端口所使用的协议、负载均衡的 SNI 配置等内容。</td><td>pilot</td></tr><tr><td>ServiceEntry</td><td>networking</td><td>ServiceEntry 能够在 Istio 内部的服务注册表中加入额外的条目，从而让网格中自动发现的服务能够访问和路由到这些手工加入的服务。</td><td>pilot</td></tr><tr><td>EnvoyFilter</td><td>networking</td><td>EnvoyFilter 对象描述了针对代理服务的过滤器，这些过滤器可以定制由 Istio Pilot 生成的代理配置。</td><td>pilot</td></tr><tr><td>ServiceDependency</td><td>networking</td><td>ServiceDependency描述工作负载依赖的服务集合。默认情况下，Istio建立的服务网格将具有完整的网格连接 - 即每个工作负载都拥有访问网格中的每个其他工作负载的代理配置。但是，大多数连接图在实践中都很稀疏。ServiceDependency提供了一种的方法来声明与每个工作负载关联的服务依赖关系，以便发送到sidecars的配置数量可以限定为必要的依赖。</td><td>pilot</td></tr></tbody></table>
