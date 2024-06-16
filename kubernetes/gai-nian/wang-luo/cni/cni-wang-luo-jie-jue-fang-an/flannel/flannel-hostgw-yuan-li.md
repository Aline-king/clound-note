# flannel host-gw原理

Pod与Pod不经隧道封装而直接通信，要求各节点位于同一个二层网络

<figure><img src="../../../../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

1 节点上的pod通过虚拟网卡对，连接到cni0的虚拟网络交换机上。

2 pod向外通信的时候，到达CNI0的时候，不再直接交给flannel.1由flanneld来进行打包处理了。

3 cni0直接借助于内核中的路由表 ，通过宿主机的网卡交给同网段的其他主机节点

4 对端节点查看内核中的路由表，发现目标就是当前节点，所以交给对应的cni0，进而找到对应的pod。
