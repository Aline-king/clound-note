# Service（Client） to Service ingress listener

<figure><img src="../../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

这张图展示了服务容器的架构，其中包含一个Envoy Sidecar（ingress）和一个App Service local\_service。Envoy Sidecar监听0.0.0.0:10000，并通过路由将请求转发到名为local\_service的集群中。然后，这个集群会将请求发送给运行在127.0.0.1:80上的App Service local\_service实例。这种架构通常用于微服务环境中，以提供网络代理、负载均衡和其他服务发现功能。
