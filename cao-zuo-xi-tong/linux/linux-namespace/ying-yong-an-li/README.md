# 应用案例

在 Linux 中，网络命名空间可以被认为是隔离的拥有单独网络栈（网卡、路由转发表、iptables）的环境。网络命名空间经常用来隔离网络设备和服务，只有拥有同样网络命名空间的设备，才能看到彼此。

* 从逻辑上说，网络命名空间是网络栈的副本，拥有自己的网络设备、路由选择表、邻接表、Netfilter表、网络套接字、网络procfs条目、网络sysfs条目和其他网络资源。
* 从系统的角度来看，当通过clone()系统调用创建新进程时，传递标志CLONE\_NEWNET将在新进程中创建一个全新的网络命名空间。
* 从用户的角度来看，我们只需使用工具ip（package is iproute2）来创建一个新的持久网络命名空间。

<figure><img src="../../../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>
