# metrics

在k8s中他们主要是以指标的样式来获取的，这些包括核心指标和自定义指标。

在k8s的系统上包含各种各样的指标数据，早期的k8s系统，为[kubelet](../../../zu-jian/kubelet/)集成了一个 [CAdvsior](../../../zu-jian/kubelet/cadvsior.md)工具可以获取kubelet所在节点上的相关指标，包括容器指标。

但是CAdvsior的缺陷在于，我们仅能够获取，指定节点上的指标信息，而无法获取集群管理的统一指标。





