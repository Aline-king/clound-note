# 部署

```bash
wget https://github.com/antrea-io/antrea/releases/download/v1.11.0/antrea.yml
```

### 修改antrea部署文件

1、禁用overlay封装模式及SNAT模式

2、配置Service的IPv4及IPv6地址段。

vim antrea.yml

```
3022     trafficEncapMode: "encap"
3023
3024     # Whether or not to SNAT (using the Node IP) the egress traffic from a Pod to the external network.
3025     # This option is for the noEncap traffic mode only, and the default value is false. In the noEncap
3026     # mode, if the cluster's Pod CIDR is reachable from the external network, then the Pod traffic to
3027     # the external network needs not be SNAT'd. In the networkPolicyOnly mode, antrea-agent never
3028     # performs SNAT and this option will be ignored; for other modes it must be set to false.
3029     noSNAT: false


3022     trafficEncapMode: "noencap"
3023
3024     # Whether or not to SNAT (using the Node IP) the egress traffic from a Pod to the external network.
3025     # This option is for the noEncap traffic mode only, and the default value is false. In the noEncap
3026     # mode, if the cluster's Pod CIDR is reachable from the external network, then the Pod traffic to
3027     # the external network needs not be SNAT'd. In the networkPolicyOnly mode, antrea-agent never
3028     # performs SNAT and this option will be ignored; for other modes it must be set to false.
3029     noSNAT: true

```

```
3097     serviceCIDR: ""
3098
3099     # ClusterIP CIDR range for IPv6 Services. It's required when using kube-proxy to provide IPv6 Service in a Dual-Stack
3100     # cluster or an IPv6 only cluster. The value should be the same as the configuration for kube-apiserver specified by
3101     # --service-cluster-ip-range. When AntreaProxy is enabled, this parameter is not needed.
3102     # No default value for this field.
3103     serviceCIDRv6: ""


3097     serviceCIDR: "10.96.0.0/16"
3098
3099     # ClusterIP CIDR range for IPv6 Services. It's required when using kube-proxy to provide IPv6 Service in a Dual-Stack
3100     # cluster or an IPv6 only cluster. The value should be the same as the configuration for kube-apiserver specified by
3101     # --service-cluster-ip-range. When AntreaProxy is enabled, this parameter is not needed.
3102     # No default value for this field.
3103     serviceCIDRv6: "2005::/110"
```

### 应用antrea部署文件

kubectl create -f antrea.yml

### 验证antrea部署是否成功

```
[root@k8s-master01 ~]# kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
antrea-agent-2fzm4                     2/2     Running   0          99s
antrea-agent-9jp7g                     2/2     Running   0          99s
antrea-agent-vkmk4                     2/2     Running   0          99s
antrea-controller-789587f966-j62zq     1/1     Running   0          99s
coredns-787d4945fb-82tmg               1/1     Running   0          37m
coredns-787d4945fb-vdsln               1/1     Running   0          37m
etcd-k8s-master01                      1/1     Running   0          38m
kube-apiserver-k8s-master01            1/1     Running   0          38m
kube-controller-manager-k8s-master01   1/1     Running   0          38m
kube-proxy-4pvpv                       1/1     Running   0          37m
kube-proxy-4szqs                       1/1     Running   0          18m
kube-proxy-sl8h5                       1/1     Running   0          23m
kube-scheduler-k8s-master01            1/1     Running   0          38m
```

### 查看K8S集群节点主机路由信息

```
[root@k8s-master01 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 ens33
10.244.0.0      0.0.0.0         255.255.255.128 U     0      0        0 antrea-gw0
10.244.0.128    k8s-worker01    255.255.255.128 UG    0      0        0 ens33
10.244.1.0      k8s-worker02    255.255.255.128 UG    0      0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.10.0    0.0.0.0         255.255.255.0   U     100    0        0 ens33
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```

```
[root@k8s-master01 ~]# route -6
Kernel IPv6 routing table
Destination                    Next Hop                   Flag Met Ref Use If
[::]/96                        [::]                       !n   1024 2      0 lo
0.0.0.0/96                     [::]                       !n   1024 2      0 lo
2002:a00::/24                  [::]                       !n   1024 1      0 lo
2002:7f00::/24                 [::]                       !n   1024 2      0 lo
2002:a9fe::/32                 [::]                       !n   1024 1      0 lo
2002:ac10::/28                 [::]                       !n   1024 2      0 lo
2002:c0a8::/32                 [::]                       !n   1024 1      0 lo
2002:e000::/19                 [::]                       !n   1024 4      0 lo
2003::/64                      [::]                       U    100 7      0 ens33
2004::/80                      [::]                       U    256 3      0 antrea-gw0
2004::1:0:0:0/80               k8s-worker01               UG   1024 1      0 ens33
2004::2:0:0:0/80               k8s-worker02               UG   1024 1      0 ens33
3ffe:ffff::/32                 [::]                       !n   1024 1      0 lo
fe80::/64                      [::]                       U    100 1      0 ens33
[::]/0                         gateway                    UG   100 5      0 ens33
localhost/128                  [::]                       Un   0   7      0 lo
2003::/128                     [::]                       Un   0   3      0 ens33
k8s-master01/128               [::]                       Un   0   8      0 ens33
2004::/128                     [::]                       Un   0   3      0 antrea-gw0
k8s-master01/128               [::]                       Un   0   3      0 antrea-gw0
fe80::/128                     [::]                       Un   0   3      0 ens33
fe80::/128                     [::]                       Un   0   3      0 antrea-gw0
k8s-master01/128               [::]                       Un   0   4      0 ens33
k8s-master01/128               [::]                       Un   0   3      0 antrea-gw0
ff00::/8                       [::]                       U    256 3      0 ens33
ff00::/8                       [::]                       U    256 2      0 antrea-gw0
[::]/0                         [::]                       !n   -1  1      0 lo
```

## 测试双栈协议可用性

### 6.1 IPv6访问测试

```
[root@k8s-master01 ~]# vim deployment.yaml
[root@k8s-master01 ~]# cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginxweb
  name: nginxweb
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginxweb
  template:
    metadata:
      labels:
        app: nginxweb
    spec:
      containers:
      - name: nginxweb
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
```

```
[root@k8s-master01 ~]# kubectl apply -f deployment.yaml
deployment.apps/nginxweb created
```

```
[root@k8s-master01 ~]# kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
nginxweb-5c77c86f4d-9fdgh   1/1     Running   0          4s
nginxweb-5c77c86f4d-hzk6x   1/1     Running   0          4s
```

```
[root@k8s-master01 ~]# kubectl get pods -o yaml | grep ip
    - ip: 10.244.0.130
    - ip: 2004::1:0:0:2
    - ip: 10.244.1.2
    - ip: 2004::2:0:0:2
```

```
[root@k8s-master01 ~]# kubectl describe pods nginxweb-5c77c86f4d-9fdgh
Name:         nginxweb-5c77c86f4d-9fdgh
Namespace:    default
Priority:     0
Node:         k8s-worker01/192.168.10.161
Start Time:   Fri, 24 Mar 2023 19:56:16 +0800
Labels:       app=nginxweb
              pod-template-hash=5c77c86f4d
Annotations:  <none>
Status:       Running
IP:           10.244.0.130
IPs:
  IP:           10.244.0.130
  IP:           2004::1:0:0:2
Controlled By:  ReplicaSet/nginxweb-5c77c86f4d
Containers:
  nginxweb:
    Container ID:   docker://68139623df054e7eb47f8a0fdb3891dc36c926ef36edc5b4c4dc25e81ffe3d01
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:617661ae7846a63de069a85333bb4778a822f853df67fe46a688b3f0e4d9cb87
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 24 Mar 2023 19:56:18 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dgjq4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-dgjq4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  75s   default-scheduler  Successfully assigned default/nginxweb-5c77c86f4d-9fdgh to k8s-worker01
  Normal  Pulled     74s   kubelet            Container image "nginx:latest" already present on machine
  Normal  Created    74s   kubelet            Created container nginxweb
  Normal  Started    73s   kubelet            Started container nginxweb
```

```
[root@k8s-master01 ~]# ping6 2004::1:0:0:2
PING 2004::1:0:0:2(2004::1:0:0:2) 56 data bytes
64 bytes from 2004::1:0:0:2: icmp_seq=8 ttl=62 time=2049 ms
64 bytes from 2004::1:0:0:2: icmp_seq=9 ttl=62 time=1024 ms
64 bytes from 2004::1:0:0:2: icmp_seq=10 ttl=62 time=2.42 ms
64 bytes from 2004::1:0:0:2: icmp_seq=11 ttl=62 time=0.370 ms
64 bytes from 2004::1:0:0:2: icmp_seq=12 ttl=62 time=0.651 ms
64 bytes from 2004::1:0:0:2: icmp_seq=13 ttl=62 time=1.30 ms
```

```
[root@k8s-master01 ~]# ping6 2004::2:0:0:2
PING 2004::2:0:0:2(2004::2:0:0:2) 56 data bytes
64 bytes from 2004::2:0:0:2: icmp_seq=1 ttl=62 time=1.35 ms
64 bytes from 2004::2:0:0:2: icmp_seq=2 ttl=62 time=1.24 ms
```

```
[root@k8s-master01 ~]# curl -g -6 [2004::1:0:0:2]
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
​
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
​
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```
[root@k8s-master01 ~]# curl -g -6 [2004::2:0:0:2]
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
​
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
​
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 6.2 双栈service测试

Kubernetes Service 的 IPv4 family 支持下列选项，默认为 IPv4 SingleStack：

* SingleStack
* PreferDualStack
* RequireDualStack

```
[root@k8s-master01 ~]# vim service.yaml
[root@k8s-master01 ~]# cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxweb-v6
spec:
  selector:
    app: nginxweb
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
  ipFamilyPolicy: RequireDualStack
```

```
[root@k8s-master01 ~]# kubectl create -f service.yaml
service/nginxweb-v6 created
```

```
[root@k8s-master01 ~]# kubectl get svc
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP        63m
nginxweb-v6   NodePort    10.96.53.221   <none>        80:32697/TCP   22s
```

```
[root@k8s-master01 ~]# kubectl get svc -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: "2023-03-24T11:12:51Z"
    labels:
      component: apiserver
      provider: kubernetes
    name: kubernetes
    namespace: default
    resourceVersion: "205"
    uid: d1884296-7060-4dab-961b-3537de7490d0
  spec:
    clusterIP: 10.96.0.1
    clusterIPs:
    - 10.96.0.1
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: 6443
    sessionAffinity: None
    type: ClusterIP
  status:
    loadBalancer: {}
- apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: "2023-03-24T12:16:26Z"
    name: nginxweb-v6
    namespace: default
    resourceVersion: "6905"
    uid: c2baa4f5-1ec8-43d3-9fc9-77f9236a4eba
  spec:
    clusterIP: 10.96.53.221
    clusterIPs:
    - 10.96.53.221
    - 2005::2c47
    externalTrafficPolicy: Cluster
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    - IPv6
    ipFamilyPolicy: RequireDualStack
    ports:
    - nodePort: 32697
      port: 80
      protocol: TCP
      targetPort: 80
    selector:
      app: nginxweb
    sessionAffinity: None
    type: NodePort
  status:
    loadBalancer: {}
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

```
[root@k8s-master01 ~]# kubectl describe svc nginxweb-v6
Name:                     nginxweb-v6
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginxweb
Type:                     NodePort
IP Family Policy:         RequireDualStack
IP Families:              IPv4,IPv6
IP:                       10.96.53.221
IPs:                      10.96.53.221,2005::2c47
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32697/TCP
Endpoints:                10.244.0.130:80,10.244.1.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

```
[root@k8s-master01 ~]# curl -g -6 [2005::2c47]
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
​
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
​
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

\
