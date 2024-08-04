# IPVS

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

关键点： ipvs会在每个节点上创建一个名为kube-ipvs0的虚拟接口，并将集群所有Service对象的ClusterIP和ExternalIP都配置在该接口； - 所以每增加一个ClusterIP 或者 EternalIP，就相当于为 kube-ipvs0 关联了一个地址罢了。 kube-proxy为每个service生成一个虚拟服务器( IPVS Virtual Server)的定义。'

基本流程： 所以当前节点接收到外部流量后，如果该数据包是交给当前节点上的clusterIP，则会直接将数据包交给kube-ipvs0，而这个接口是内核虚拟出来的，而kube-proxy定义的VS直接关联到kube-ipvs0上。 如果是本地节点pod发送的请求，基本上属于本地通信，效率是非常高的。 默认情况下，这里的ipvs使用的是nat转发模型，而且支持更多的后端调度算法。仅仅在涉及到源地址转换的场景中，会涉及到极少量的iptables规则(应该不会超过20条)



前提：当前操作系统需要提前加载ipvs模块 yum install ipvsadm -y



