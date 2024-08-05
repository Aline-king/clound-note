# 工作模型

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

userspace模型是k8s(1.1-1.2)最早的一种工作模型，作用就是将service的策略转换成iptables规则，这些规则仅仅做请求的拦截，而不对请求进行调度处理。 该模型中，请求流量到达内核空间后，由套接字送往用户空间的kube-proxy，再由它送回内核空间，并调度至后端Pod。因为涉及到来回转发，效率不高，另外用户空间的转发，默认开启了会话粘滞，会导致流量转发给无效的pod上。

iptables模式是k8s(1.2-至今)默认的一种模式，作用是将service的策略转换成iptables规则，不仅仅包括拦截，还包括调度，捕获到达ClusterIP和Port的流量，并重定向至当前Service的代理的后端Pod资源。性能比userspace更加高效和可靠缺点： 不会在后端Pod无响应时自动重定向，而userspace可以 中量级k8s集群(service有几百个)能够承受，但是大量级k8s集群(service有几千个)维护达几万条规则，难度较大



请求流量的转发和调度功能由ipvs实现，余下的其他功能仍由iptables完成。ipvs流量转发速度快，规则同步性能好，且支持众多调度算法，如rr/lc/dh/sh/sed/nq等。

注意： 对于我们kubeadm方式安装k8s集群来说，他会首先检测当前主机上是否已经包含了ipvs模块，如果加载了，就直接用ipvs模式，如果没有加载ipvs模块的话，会自动使用iptables模式。
