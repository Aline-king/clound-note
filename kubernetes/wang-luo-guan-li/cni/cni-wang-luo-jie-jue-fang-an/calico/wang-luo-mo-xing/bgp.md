# BGP

## **bgp改造**

对于现有的calico环境，我们如果需要使用BGP环境，我们可以直接使用一个配置清单来进行修改calico环境即可。

我们这里先来演示一下如何使用calicoctl修改配置属性。

{% tabs %}
{% tab title="文件改造" %}
获取当前的配置属性

```bash
# kubectl calico get ipPools
NAME                  CIDR            SELECTOR
default-ipv4-ippool   10.244.0.0/16   all()

# kubectl calico get ipPools default-ipv4-ippool -o yaml
apiVersion: projectcalico.org/v3
kind: IPPool
...
spec:
  blockSize: 24
  cidr: 10.244.0.0/16
  ipipMode: Always
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Never
```

配置解析：&#x20;

仅仅将原来的Always 更换成 CrossSubnet(跨节点子网) 模式即可

&#x20;vxlanMode 的两个值可以实现所谓的 BGP with vxlan的效果

```bash
定制资源配置文件
kubectl calico get ipPools default-ipv4-ippool -o yaml > default-ipv4-ippool.yaml

修改配置文件
# cat default-ipv4-ippool.yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  blockSize: 24
  cidr: 10.244.0.0/16
  ipipMode: CrossSubnet
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Never
```

应用资源配置文件

```bash
# kubectl calico apply -f default-ipv4-ippool.yaml
Successfully applied 1 'IPPool' resource(s)
```


{% endtab %}

{% tab title="检查效果" %}
查看路由信息

```bash
[root@kubernetes-master1 ~]# ip route list | grep via
default via 10.0.0.2 dev eth0
10.244.1.0/24 via 10.0.0.13 dev eth0 proto bird
10.244.2.0/24 via 10.0.0.14 dev eth0 proto bird
10.244.3.0/24 via 10.0.0.15 dev eth0 proto bird
10.244.4.0/24 via 10.0.0.16 dev eth0 proto bird
10.244.5.0/24 via 10.0.0.17 dev eth0 proto bird
```

更新完毕配置后，动态路由的信息就发生改变了，不再完全是 tunl0 网卡了，而是变成了通过具体的物理网卡eth0 转发出去了

```bash
在master1上ping在节点node2上的pod
[root@kubernetes-master1 ~]# ping -c 1 10.244.4.3
PING 10.244.4.3 (10.244.4.3) 56(84) bytes of data.
64 bytes from 10.244.4.3: icmp_seq=1 ttl=63 time=0.671 ms

由于这次是直接通过节点进行转发的，所以我们在node2节点上抓包的时候，直接通过内层ip抓取即可。
[root@kubernetes-node2 ~]# tcpdump -i eth0 -nn ip host 10.244.4.3
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
15:47:32.906462 IP 10.0.0.12 > 10.244.4.3: ICMP echo request, id 28248, seq 1, length 64
15:47:32.906689 IP 10.244.4.3 > 10.0.0.12: ICMP echo reply, id 28248, seq 1, length 64
```

他们实现了直接的连通，无需进行数据包的转换了，效率更高一点
{% endtab %}
{% endtabs %}















