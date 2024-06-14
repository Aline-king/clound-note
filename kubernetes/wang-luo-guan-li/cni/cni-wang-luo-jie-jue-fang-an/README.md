# CNI网络解决方案

[**Flannel**](https://github.com/coreos/flannel)：提供叠加网络，基于linux TUN/TAP，使用UDP封装IP报文来创建叠加网络，并借助etcd维护网络分配情况

[**Calico**](https://github.com/projectcalico/cni-plugin)：基于BGP的三层网络，支持网络策略实现网络的访问控制。在每台机器上运行一个vRouter，利用内核转发数据包，并借助iptables实现防火墙等功能

**kube-router：**K8s网络一体化解决方案，可取代kube-proxy实现基于ipvs的Service，支持网络策略、完美兼容BGP的高级特性

其他网络解决方案：

[Canal](https://github.com/projectcalico/canal): 由Flannel和Calico联合发布的一个统一网络插件，支持网络策略

[Weave Net](https://www.weave.works/oss/net/): 多主机容器的网络方案，支持去中心化的控制平面

Contiv:思科方案，直接提供多租户网络，支持L2(VLAN)、L3(BGP)、Overlay(VXLAN)

[更多解决方案](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/addons/)
