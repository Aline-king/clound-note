# Service to Service egress listener

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

这张图是一个服务容器的示意图，展示了应用程序客户端与Envoy Sidecar（Egress）之间的交互。在服务容器中，有一个名为"App Client"的应用程序客户端和一个名为"Envoy Sidecar (Egress)"的服务代理。

应用程序客户端通过监听器（监听地址为127.0.0.1：8080）发送请求到Envoy Sidecar（Egress）。然后，Envoy Sidecar根据路由规则将请求转发给名为"some\_service"的集群。最后，集群将响应返回给Envoy Sidecar，再由Envoy Sidecar将响应传递回应用程序客户端。

这个过程展示了Envoy Sidecar作为服务容器内部组件之间通信的中介角色，并且能够对进出容器的数据流进行控制和管理。
