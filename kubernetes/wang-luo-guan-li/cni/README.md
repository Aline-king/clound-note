# CNI

CNI- **Container Network Interface**，Google和CoreOS联合定制的网络标准，这个标准基于rkt实现多容器通信的网络模型。

## 原因及规范

生产中的网络环境可能是多种多样的，有可能是二层连通的，也可能用的公有云的环境，所以各个厂商的网络解决方案百花争鸣，解决方案也不能全都集成在kubelet的代码中，所以CNI就是能让各个网络厂商对接进来的接口

* CNI插件负责连接容器，容器就是linux network namespace。
* CNI的网络定义以json的格式存储
* 有关网络的配置通过STDIN的方式传递给CNI插件，其他的参数通过环境变量的方式传递

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

## 原理解释&#x20;

1. 所有节点的内核都启用了VXLAN的功能模块 每个节点都启动一个cni网卡，并维护所有节点所在的网段的路由列表&#x20;
2. node上的pod发出请求到达cni0&#x20;
   1. 根据内核的路由列表判断对端网段的节点位置&#x20;
   2. 经由 隧道设备 对数据包进行封装标识，对端节点的隧道设备解封标识数据包，&#x20;
   3. 当前数据包一看当前节点的路由表发现有自身的ip地址，这直接交给本地的pod&#x20;
3. 多个节点上的路由表信息维护，就是各种网络解决方案的工作位置

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

## 被谁管理

Kubernetes 1.24 之前，CNI 插件也可以由 kubelet 使用命令行参数 cni-bin-dir 和 network-plugin 管理。

Kubernetes 1.24之后 移除了这些命令行参数，

CNI 的管理不再是 kubelet 的工作。而变成下层的容器引擎需要做的事情了，比如 cri-dockerd 服务的启动文件
