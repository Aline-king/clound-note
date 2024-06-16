# 控制器

在k8s中，一切行为接被控制器所控制。也就是说，k8s主要有 控制层面和数据层面来组成：&#x20;

1. 控制层面&#x20;
   * API Server - 提供数据的注册和监视各种资源对象的功能&#x20;
   * Scheduler - 将资源对象调度到合适的节点中。&#x20;
   * Controller - 大量控制功能的集合,与API Server结合起来完成控制层面操作。&#x20;
2. 数据层面&#x20;
   * Etcd - 提供各种数据的存储

{% hint style="info" %}
控制器是通过标签和标签选择器来找到pod，进而控制 管理pod
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

节点资源管理

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

k8s集群中的所有资源对象都是 通过控制器来进行管控

{% hint style="info" %}
master

* 主机上的各种组件进行统一管理

node

* kubelet组件实现信息的交流。

&#x20;pod内部

* 基于CRI让应用本身是运行在容器内部。&#x20;
* 基于CSI实现各种持久化数据的保存。
* 基于CNI实现多个pod应用程序之间的通信交流。
{% endhint %}

资源访问

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

**控制流程**

在k8s中，要想对资源进行管控，需要先定义资源，然后再进行初始化，最后形成资源对象。资源对象有两种形式

文件形态 - 编写出来的资源对象文件，有大量属性组成。&#x20;

对象形态 - 基于资源对象文件初始化后出来的应用对象。

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

API Server 从某种层面上来说，它仅仅是一个数据库的接口，对外提供数据库的访问

\- 它支持对资源数据条目的 watch/notify 的功能，&#x20;

\- 它对于数据存储的格式进行了限制定制，也就是说，只能接收固定格式的数据规范。

我们将相关对象的属性信息插入到 APIServer上，而APIServer会驱动kube-controller-manager，将一个个的具体对象落地。



## 资源状态

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

1. 用户向 APIserver中插入一个应用资源的数据形态&#x20;
   * 这个数据形态中定义了该资源对象的 "期望"状态，&#x20;
   * 数据经由 APIserver 保存到 ETCD 中。
2. kube-controller-manager 中的各种控制器会监视 Apiserver上与自己相关的资源对象的变动&#x20;
   * 比如 Service Controller只负责Service资源的控制，Pod Controller只负责Pod资源的控制等。&#x20;
3. 一旦APIServer中的资源对象发生变动，对应的Controller执行相关的配置代码，到对应的node节点上运行&#x20;
   * 该资源对象会在当前节点上，按照用户的"期望"进行运行&#x20;
   * 这些实体对象的运行状态我们称为 "实际状态" &#x20;
     * 控制器的作用就是确保 "期望状态" 与 "实际状态" 相一致&#x20;
4. Controller将这些实际的资源对象状态，通过APIServer存储到ETCD的同一个数据条目的status的字段中&#x20;
5. 资源对象在运行过程中，Controller 会循环的方式向 APIServer 监控 spec 和 status 的值是否一致&#x20;
   * 如果两个状态不一致，那么就指挥node节点的资源进行修改，保证 两个状态一致&#x20;
   * 状态一致后，通过APIServer同步更新当前资源对象在ETCD上的数据

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
