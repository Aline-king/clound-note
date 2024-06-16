# kube-proxy

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

kube-proxy 默认支持iptables规则，

查看模式

```
通过kube-proxy-b8dpc的pod日志查看模式
[root@kubernetes-master1 /data/kubernetes/service]# kubectl logs kube-proxy-b8dpc -n kube-system
...
I0719 08:39:44.410078       1 server_others.go:561] "Unknown proxy mode, assuming iptables proxy" proxyMode=""
I0719 08:39:44.462438       1 server_others.go:206] "Using iptables Proxier"
```
