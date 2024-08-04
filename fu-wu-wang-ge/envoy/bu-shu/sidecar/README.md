# sidecar

Envoy通常用于以容器编排系统为底层环境的服务网格中，并以sidecar的形式与主程序容器运行为单个pod；

在非编排系统环境中测试时，可以将主程序与Envoy运行于同一容器，或手动组织主程序容器与Envoy容器共享同一个网络名称空间

具体使用时常见的部署类型如下图所示：

<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 用户流量通过Front Proxy把流量转到对应服务的Ingress Listener即可对对应的服务进行访问
* 当Service A需要对Service B、C、D服务进行访问时，是通过Egress Listener实现的，即Service A的Envoy SideCar中Egress Listener为正向代理实现对B、C、D进行访问。
