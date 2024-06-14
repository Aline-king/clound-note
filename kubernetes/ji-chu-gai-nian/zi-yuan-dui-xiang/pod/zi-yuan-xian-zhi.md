# 资源限制

Kubernetes中，对于每种资源的配额限定都需要两个参数：

Resquests和Limits 申请配额(Requests)： 业务运行时最小的资源申请使用量，该参数的值必须满足，若不满足，业务运行不起来。&#x20;

最大配额(Limits)： 业务运行时最大的资源允许使用量，该参数的值不能被突破，若突破，该业务的资源对象会被重启或删除等意外操作

<figure><img src="../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

kubernetes为了方便统一管理各种容器的资源使用情况，提供了一种资源限制对象 LimitRange，它可以针对我们的请求的资源进行简单的控制。
