# CRD（自定义资源对象）

## **介绍**

**自定义资源 CRD**（Custom Resource Definition）可以扩展 Kubernetes API，它是Kubernetes强大扩展能力的一处体现，类比到编程场景，CRD就相当于class类，而自定义资源就是基于class类实例出来的对象，通过CRD我们可以创建自定义的资源类型，而且api-server能够直接支持，借助于kubectl命令，我们可以直接对这些资源进行增删改查操作。&#x20;

在Kubernetes中，CRD主要以声明式 API的样式存在。 CRD是 v1.7 + 新增的无需改变k8s源代码的前提下就可以扩展 Kubernetes API 的机制，用来管理自定义对象。

## 主要的应用场景是：

提供/管理外部数据存储/数据库(例如 CloudSQL/RDS 实例)；对k8s基础资源进行更高层次的抽象(比如定义一个etcd集群)

## CRD步骤

1. 编写 CRD 并将其部署到 K8s 集群里；&#x20;
   1. 这一步的作用就是让 K8s 知道有这个资源及其结构属性，在用户提交该自定义资源的定义时（通常是 YAML 文件定义），K8s 能够成功校验该资源并创建出对应的 Go struct 进行持久化，同时触发控制器的调谐逻辑。
2. &#x20;编写 Controller 并将其部署到 K8s 集群里。 这一步的作用就是实现K8s整合的逻辑。

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<table><thead><tr><th width="179"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>kube-aggregator</td><td> k8s提供了一种聚合器的能力，用于扩展用户需要的资源对象。<br>简单来说，就是用户为例实现自定义资源来扩展k8s功能，用户按照API Server的模式创建自己的API Server，然后通过aggregate这一层组件实现功能聚合。 </td><td></td></tr><tr><td>apiextensions-apiserver</td><td><p>当在k8s集群中注册CRD资源后，kube-apiserver内部的apiextensions-apiserver就会检查CRD的名字和语法是否满足需求，如果满足就可以注册成功了。</p><p>当然了如果想要实现更加复杂的创建校验，可以通过准入控制器来进行校验。 </p></td><td></td></tr><tr><td>discover information</td><td>当我们创建完毕CRD之后，kubectl就会通过API服务器的discover information来查找新资源的信息。</td><td></td></tr></tbody></table>





&#x20;

:&#x20;
