# flannal vxlan原理

## 要求

跨网段通信机制 默认 pod与Pod经由隧道封装后通信，各节点彼此间能通信就行，不要求在同一个二层网络

<figure><img src="../../../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

1. 节点上的pod通过虚拟网卡对，连接到cni0的虚拟网络交换机上，当有外部网络通信的时候，借助于 flannel.1网卡向外发出数据包
2. 经过 flannel.1 网卡的数据包，借助于flanneld实现数据包的封装和解封，最后送给宿主机的物理接口，发送出去
3. 对于pod来说，它以为是通过 flannel.x -> vxlan tunnel -> flannel.x 实现数据通信

因为它们的隧道标识都是".1"，所以认为是一个vxlan，直接路由过去了，没有意识到底层的通信机制。

{% hint style="warning" %}
注意：

由于这种方式，是对数据报文进行了多次的封装，降低了当个数据包的有效载荷。所以效率降低了
{% endhint %}

