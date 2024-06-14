# 部署

## 安装calico

{% tabs %}
{% tab title="获取资源清单文件" %}
```bash
mkdir /data/kubernetes/network/calico -p
cd /data/kubernetes/network/calico/
curl https://docs.projectcalico.org/manifests/calico.yaml -O
cp calico.yaml{,.bak}
```


{% endtab %}

{% tab title="配置资源清单文件" %}
开放默认注释的CALICO\_IPV4POOL\_CIDR变量，然后定制我们当前的pod的网段范围即可

```bash
# vim calico.yaml
---- 官网推荐的修改内容
4546             - name: CALICO_IPV4POOL_CIDR
4547               value: "10.244.0.0/16"
---- 方便我们的后续实验，新增调小子网段的分配
4548             - name: CALICO_IPV4POOL_BLOCK_SIZE
4549               value: "24"
```


{% endtab %}

{% tab title="分配网络" %}
```
# vim calico.yaml
---- 修改下面的内容
  64           "ipam": {
  65               "type": "calico-ipam"
  66           },
---- 修改后的内容
  64           "ipam": {
  65               "type": "host-local",
  66               "subnet": "usePodCidr"
  67           },
---- 定制calico使用k8s集群节点的地址
4551             - name: USE_POD_CIDR
4552               value: "true"
```

Calico默认并不会从Node.Spec.PodCIDR中分配地址，但可通过**USE\_POD\_CIDR** 变量并结合host-local这IPAM插件以强制从PodCIDR中分配地址


{% endtab %}

{% tab title="应用资源配置文件" %}
为了避免后续calico网络测试的异常，我们这里最好只留下一个网卡 eth0

```
清理之前的flannel插件
kubectl delete -f kube-flannel.yml
kubectl get pod -n kube-system | grep flannel

这个时候，先清除旧网卡，然后最好重启一下主机，直接清空所有的路由表信息
ifconfig flannel.1
reboot
```


{% endtab %}

{% tab title="应用" %}
```bash
kubectl apply -f calico.yaml
 
在calico-node部署的时候，会启动多个进程
# kubectl get pod -n kube-system | egrep 'NAME|calico'
NAME                                        READY   STATUS              RESTARTS       AGE
calico-kube-controllers-549f7748b5-xqz8j    0/1     ContainerCreating   0              9s
calico-node-74c5w                           0/1     Init:0/3            0              9s
...

环境部署完毕后，查看效果
# kubectl get pod -n kube-system | egrep 'NAME|calico'
NAME                                        READY   STATUS    RESTARTS       AGE
calico-kube-controllers-549f7748b5-xqz8j    0/1     Running   0              39s
calico-node-74c5w                           0/1     Running   0              39s
...
```

展示

```
每个calico节点上都有多个进程，分别来处理不同的功能
# ps aux | egrep 'NAME | calico'
root       9315  0.5  1.1 1524284 43360 ?       Sl   15:29   0:00 calico-node -confd
root       9316  0.2  0.8 1155624 32924 ?       Sl   15:29   0:00 calico-node -monitor-token
root       9317  2.8  1.0 1598528 40992 ?       Sl   15:29   0:02 calico-node -felix
root       9318  0.3  0.9 1155624 35236 ?       Sl   15:29   0:00 calico-node -monitor-addresses
root       9319  0.3  0.8 1155624 33460 ?       Sl   15:29   0:00 calico-node -status-reporter
root       9320  0.2  0.7 1155624 30364 ?       Sl   15:29   0:00 calico-node -allocate-tunnel-addrs
```

测试

```
获取镜像
docker pull nginx
docket tag nginx kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.1
docker push kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.1
docker rmi nginx
```

```
创建一个deployment
kubectl create deployment pod-deployment --image=kubernetes-register.superopsmsb.com/superopsmsb/nginx:1.23.1 --replicas=3

查看pod
# kubectl get pod
NAME                             READY   STATUS    RESTARTS   AGE
pod-deployment-554dff674-267gq   1/1     Running   0          48s
pod-deployment-554dff674-c8cjs   1/1     Running   0          48s
pod-deployment-554dff674-pxrwb   1/1     Running   0          48s

暴露一个service
kubectl expose deployment pod-deployment --port=80 --target-port=80

确认效果
kubectl get service
curl 10.108.138.97
```


{% endtab %}

{% tab title="命令完善" %}
```bash
cd /usr/local/bin/
curl -L https://github.com/projectcalico/calico/releases/download/v3.23.3/calicoctl-linux-amd64 -o calicoctl
mv calicoctl kubectl-calico
chmod +x kubectl-calico

测试效果
kubectl calico node status
```
{% endtab %}
{% endtabs %}

## 测试

{% tabs %}
{% tab title="First Tab" %}
```
自动生成一个api版本信息
# kubectl api-versions  | grep crd
crd.projectcalico.org/v1

该api版本信息中有大量的配置属性
kubectl api-resources  | grep crd.pro
```

> calico采用的模型就是 ipip模型,
>
> 分配的网段是使我们定制的 cidr网段，
>
> 子网段也是我们定制的 24位掩码

```bash
# kubectl get ippools   // 里面包含了calico相关的网络属性信息
NAME                  AGE
default-ipv4-ippool   37m

查看这里配置的calico相关的信息
# kubectl get ippools default-ipv4-ippool -o yaml
apiVersion: crd.projectcalico.org/v1
kind: IPPool
...
spec:
  allowedUses:
  - Workload
  - Tunnel
  blockSize: 24
  cidr: 10.244.0.0/16
  ipipMode: Always
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Never
```


{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

环境创建完毕后，会生成一个tunl0的网卡，所有的流量会走这个tunl0网卡

```
确认网卡和路由信息
]# ifconfig | grep flags
]# ip route list | grep tun
10.244.1.0/24 via 10.0.0.13 dev tunl0 proto bird onlink
10.244.2.0/24 via 10.0.0.14 dev tunl0 proto bird onlink
10.244.3.0/24 via 10.0.0.15 dev tunl0 proto bird onlink
10.244.4.0/24 via 10.0.0.16 dev tunl0 proto bird onlink
10.244.5.0/24 via 10.0.0.17 dev tunl0 proto bird onlink
```

*   #### 测试效果

    calico的模型是 IPIP（默认网络），所以我们在进行数据包测试的时候，可以通过直接抓取宿主机数据包，来发现双层ip效果

    `kubectl get pod -o wide`

    在master1上采用ping的方式来测试 node2上的节点pod

    ```
    [root@kubernetes-master1 /data/kubernetes/network/calico]# ping -c 1 10.244.4.3
    PING 10.244.4.3 (10.244.4.3) 56(84) bytes of data.
    64 bytes from 10.244.4.3: icmp_seq=1 ttl=63 time=0.794 ms
    ```

    在node2上检测数据包的效果

    > 每个数据包都是基于双层ip嵌套的方式来进行传输，而且协议是 ipip-proto-4
    >
    > 具体的操作效果
    >
    > 10.244.0.1 -> 10.0.0.12 -> 10.0.0.16 -> 10.244.6.3

    ```
    [root@kubernetes-node2 ~]# tcpdump -i eth0 -nn ip host 10.0.0.16 and host 10.0.0.12
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
    15:38:52.231680 IP 10.0.0.12 > 10.0.0.16: IP 10.244.0.1 > 10.244.4.3: ICMP echo request, id 19189, seq 1, length 64 (ipip-proto-4)
    15:38:52.231989 IP 10.0.0.16 > 10.0.0.12: IP 10.244.4.3 > 10.244.0.1: ICMP echo reply, id 19189, seq 1, length 64 (ipip-proto-4)
    15:38:54.666538 IP 10.0.0.16.33992 > 10.0.0.12.179: Flags [P.], seq 4281052787:4281052806, ack 3643645463, win 58, length 19: BGP
    15:38:54.666962 IP 10.0.0.12.179 > 10.0.0.16.33992: Flags [.], ack 19, win 58, length 0
    ```

