# 网络与通信模型

> 注意：本章属于 `Docker` 高级配置，如果您是初学者，您可以暂时跳过本章节，直接学习 [Docker Compose](../compose/) 一节。

docker0

* 是一个二层网络设备，即网桥
* 通过网桥可以将Linux支持的不同的端口连接起来
* 实现类交换机多对多的通信

veth pair

* 虚拟以太网（Ethernet）设备
* 成对出现,用于解决网络命名空间之间的隔离
* 一端连接Container network namespace，另一端连接host network namespace

\


<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

当 Docker 启动时，会自动在主机上创建一个 `docker0` 虚拟网桥，实际上是 Linux 的一个 bridge，可以理解为一个软件交换机。它会在挂载到它的网口之间进行转发。

同时，Docker 随机分配一个本地未占用的私有网段（在 [RFC1918](https://datatracker.ietf.org/doc/html/rfc1918) 中定义）中的一个地址给 `docker0` 接口。比如典型的 `172.17.42.1`，掩码为 `255.255.0.0`。此后启动的容器内的网口也会自动分配一个同一网段（`172.17.0.0/16`）的地址。

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

当创建一个 Docker 容器的时候，同时会创建了一对 `veth pair` 接口（当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包）。这对接口一端在容器内，即 `eth0`；另一端在本地并被挂载到 `docker0` 网桥，名称以 `veth` 开头（例如 `vethAQI2QT`）。

通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。Docker 就创建了在主机和所有容器之间一个虚拟共享网络。

![Docker 网络](../../advanced\_network/\_images/network.png)

接下来的部分将介绍在一些场景中，Docker 所有的网络定制配置。以及通过 Linux 命令来调整、补充、甚至替换 Docker 默认的网络配置。



在介绍自定义网络拓扑之前，你可能会对一些外部工具和例子感兴趣：

> 1.  pipework
>
>     Jérôme Petazzoni 编写了一个叫 [pipework](https://github.com/jpetazzo/pipework) 的 shell 脚本，可以帮助用户在比较复杂的场景中完成容器的连接。
> 2.  playground
>
>     Brandon Rhodes 创建了一个提供完整的 Docker 容器网络拓扑管理的 [Python库](https://github.com/brandon-rhodes/fopnp/tree/m/playground)，包括路由、NAT 防火墙；以及一些提供 `HTTP` `SMTP` `POP` `IMAP` `Telnet` `SSH` `FTP` 的服务器。
