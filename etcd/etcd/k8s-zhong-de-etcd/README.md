# k8s中的etcd

## 查看k8s集群里面的etcd

{% code title="查看etcd的pod" %}
```bash
[root@kubernetes-master1 ~]# kubectl get pod -n kube-system | grep etcd
etcd-kubernetes-master1                      1/1     Running   44 (19h ago)   29h
etcd-kubernetes-master2                      1/1     Running   2 (19h ago)    29h
etcd-kubernetes-master3                      1/1     Running   2 (19h ago)    29h

集群的etcd服务是以静态pod方式来管理的
[root@kubernetes-master1 ~]# ls /etc/kubernetes/manifests/etcd.yaml
/etc/kubernetes/manifests/etcd.yaml
```
{% endcode %}

检查etcd的运行日志

```bash
kubectl -n kube-system logs etcd-kubernetes-master1
```

节点内部实现了对端主机的信息数据同步

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
