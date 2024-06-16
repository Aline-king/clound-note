# 网络策略

网络策略一般有

集群级别、namespace级别、Pod级别、ip级别、端口级别

## 属性介绍

入栈和出栈哪个策略生效，由 policyTypes 来决定。

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

部署calico并正常使用，并使用calico来进行流量策略

{% tabs %}
{% tab title="namespace流量限制" %}
配置解析：&#x20;

虽然设置了egress和ingress属性，但是下面的podSelector没有选择节点，表示只有当前命名空间所有节点不受限制

{% code title="查看资源清单文件" %}
```bash
[root@kubernetes-master1 /data/kubernetes/secure]# cat 08_kubernetes_secure_networkpolicy_ns.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-current-ns
  namespace: superopsmsb
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
  egress:
  - to:
    - podSelector: {}
  ingress:
  - from:
    - podSelector: {}
```
{% endcode %}

{% code title="应用资源清单文件" %}
```bash
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  apply -f 08_kubernetes_secure_networkpolicy_ns.yaml
networkpolicy.networking.k8s.io/allow-current-ns created
```
{% endcode %}

```bash
default资源访问superopsmsb资源
root@nginx-web-5865bb968d-759lc:/# curl nginx-web.superopsmsb.svc.cluster.local
curl: (28) Failed to connect to nginx-web.superopsmsb.svc.cluster.local port 80: Connection timed out
root@nginx-web-5865bb968d-759lc:/# ping -c 1 10.244.1.5
PING 10.244.1.5 (10.244.1.5) 56(84) bytes of data.
```

访问成功

```bash
superopsmsb资源访问同命名空间的其他资源
root@nginx-web-65d688fd6-h6sbp:/# ping 10.244.1.2
PING 10.244.1.2 (10.244.1.2) 56(84) bytes of data.
64 bytes from 10.244.1.2: icmp_seq=1 ttl=63 time=0.206 ms
```


{% endtab %}

{% tab title="默认拒绝" %}
查看资源清单文件

```bash
[root@kubernetes-master1 /data/kubernetes/secure]# cat 06_kubernetes_secure_networkpolicy_refuse.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: superopsmsb
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
```

{% code title="应用资源清单文件" %}
```bash
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  apply -f 06_kubernetes_secure_networkpolicy_refuse.yaml
networkpolicy.networking.k8s.io/deny-all-ingress created
```
{% endcode %}

```
尝试default空间资源访问superopsmsb空间资源
root@nginx-web-5865bb968d-759lc:/# ping -c 1 10.244.1.5
PING 10.244.1.5 (10.244.1.5) 56(84) bytes of data.

--- 10.244.1.5 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

root@nginx-web-5865bb968d-759lc:/# curl nginx-web.superopsmsb.svc.cluster.local
curl: (28) Failed to connect to nginx-web.superopsmsb.svc.cluster.local port 80: Connection timed out
结果显示:
	访问失败
	
default空间资源访问非superopsmsb空间资源正常。
root@nginx-web-5865bb968d-759lc:/# curl nginx-web
Hello Nginx, nginx-web-5865bb968d-759lc-1.23.0
```
{% endtab %}

{% tab title="默认开启" %}
查看资源清单文件

```
[root@kubernetes-master1 /data/kubernetes/secure]# cat 07_kubernetes_secure_networkpolicy_allow.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: superopsmsb
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
  egress:
  - {}
  ingress:
  - {}
```

```
应用资源清单文件
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  apply -f 07_kubernetes_secure_networkpolicy_allow.yaml
networkpolicy.networking.k8s.io/allow-all-ingress created
```

结果显示：&#x20;

网络策略成功了，而且多个网络策略可以叠加 注意：仅仅同名策略或者同类型策略可以覆盖

```
在default空间访问superopsmsb空间资源
root@nginx-web-5865bb968d-759lc:/# curl nginx-web.superopsmsb.svc.cluster.local
Hello Nginx, nginx-web-65d688fd6-h6sbp-1.23.0
root@nginx-web-5865bb968d-759lc:/# ping -c 1 10.244.1.5
PING 10.244.1.5 (10.244.1.5) 56(84) bytes of data.
64 bytes from 10.244.1.5: icmp_seq=1 ttl=63 time=0.181 ms
```

<pre class="language-bash" data-title="清理网络策略"><code class="lang-bash"><strong>[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  delete -f 07_kubernetes_secure_networkpolicy_allow.yaml
</strong></code></pre>
{% endtab %}
{% endtabs %}



