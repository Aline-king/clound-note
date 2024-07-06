# 网络

## K8s的网络模型

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

主机网络 -> 本地网络&#x20;

服务网络 -> service网络

应用网络 -> pod网络



## 容器A和容器B在同一个pod里

它会在每个 Pod 里，额外起一个 Infra container 小容器来共享整个 Pod 的 Network Namespace。

> Infra container 是一个非常小的镜像，大概 700KB 左右，是一个C语言写的、永远处于“暂停”状态的容器 ，[官方的pause容器代码](https://github.com/kubernetes/kubernetes/tree/master/build/pause)

由于有了这样一个 Infra container 之后，其他所有容器都会通过 Join Namespace 的方式加入到 Infra container 的 Network Namespace 中。



## Infra container

由于需要有一个相当于说中间的容器存在，所以整个 Pod 里面，必然是 Infra container 第一个启动

并且整个 Pod 的生命周期是等同于 Infra container 的生命周期的，与容器 A 和 B 是无关的

这也是为什么在 Kubernetes 里面，它是允许去单独更新 Pod 里的某一个镜像的，即：做这个操作，整个 Pod 不会重建，也不会重启，这是非常重要的一个设计。

pause容器的示意图

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>网络策略</summary>

1. &#x20;集群级别、
2. namespace级别、
3. Pod级别、
4. p级别、
5. 端口级别

## 属性介绍

入栈和出栈哪个策略生效，由 <mark style="color:yellow;">**policyTypes**</mark> 来决定。

如果仅配置了podSelector，表明，当前限制仅限于当前的命名空间

```yaml
apiVersion: networking.k8s.io/v1  	# 资源隶属的API群组及版本号
kind: NetworkPolicy  			# 资源类型的名称，名称空间级别的资源；
metadata:  				# 资源元数据
  	name <string>  			# 资源名称标识
  	namespace <string>  		# NetworkPolicy是名称空间级别的资源
spec:  					# 期望的状态
  	podSelector <Object>  		# 当前规则生效的一组目标Pod对象，必选字段；空值表示当前名称空间中的所有Pod资源
  	policyTypes <[]string>  	# Ingress表示生效ingress字段；Egress表示生效egress字段，同时提供表示二者均有效
	ingress <[]Object>  		# 入站流量源端点对象列表，白名单，空值表示“所有”
	- from <[]Object>  		# 具体的端点对象列表，空值表示所有合法端点
	  - ipBlock  <Object> 		# IP地址块范围内的端点，不能与另外两个字段同时使用
	  - namespaceSelector <Object> 	# 匹配的名称空间内的端点
	    podSelector <Object>	# 由Pod标签选择器匹配到的端点，空值表示<none>
	  ports <[]Object>  		# 具体的端口对象列表，空值表示所有合法端口
	egress <[]Object>  		# 出站流量目标端点对象列表，白名单，空值表示“所有”
	- to <[]Object>  		# 具体的端点对象列表，空值表示所有合法端点，格式同ingres.from；
	  ports <[]Object>  		# 具体的端口对象列表，空值表示所有合法端口
```

关于更多网络策略相关功能，请参考calico流量管理

</details>
