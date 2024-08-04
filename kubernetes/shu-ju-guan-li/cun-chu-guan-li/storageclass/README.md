# StorageClass

在StorageClass的环境基础上，它可以让我们用户将存储资源定义为某种类型的资源，比如存储质量、快速存储、慢速存储等,然后自由的去申请合适的存储资源了。

StorageClass根据自由申请后端存储的类型，有很多不同的实现方式，在我们的试验环境中，我们这里用nfs-client-provisioner的方式来演示StorageClass

<figure><img src="../../../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
自从kubernetes 1.20版本禁用了 selfLink，所以默认情况下，我们在创建SC提供者环境的时候，会发生报错，只能首先修改kubernetes的启动属性。

参考链接：

[https://stackoverflow.com/questions/65376314/kubernetes-nfs-provider-selflink-was- empty](https://stackoverflow.com/questions/65376314/kubernetes-nfs-provider-selflink-was-emptyhttps://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/issues/25)[\
https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/issues/25](https://stackoverflow.com/questions/65376314/kubernetes-nfs-provider-selflink-was-emptyhttps://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/issues/25)
{% endhint %}

## 解决办法

```yaml
vi /etc/kubernetes/manifests/kube-apiserver.yaml
...
spec:
containers:
- command:
- kube-apiserver
- --feature-gates=RemoveSelfLink=false # 添加这条配置
```

由于是静态文件，所以该pod会自动更改属性，查看效果

```bash
[root@kubernetes-master1 ~]# kubectl get pod -n kube-system -l component=kube-apiserver
NAME READY STATUS RESTARTS AGE
kube-apiserver-kubernetes-master1 1/1 Running 0 79s
kube-apiserver-kubernetes-master2 1/1 Running 0 41s
kube-apiserver-kubernetes-master3 1/1 Running 0 40s
```
