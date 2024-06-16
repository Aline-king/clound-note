# 实践

修改实践

```
修改配置文件
[root@kubernetes-master1 ~]# kubectl edit cm coredns -n kube-system
...
        forward . /etc/resolv.conf {
           max_concurrent 1000
           except www.baidu.com.
        }
        hosts {
           10.0.0.20 harbor.superopsmsb.com
           fallthrough
        }
        ...
        注意：
            多个dns地址间用空格隔开
            排除的域名最好在末尾添加 “.”，对于之前的旧版本来说可能会出现无法保存的现象
```

测试效果

```
同步dns的配置信息
[root@kubernetes-master1 ~]# kubectl delete pod -l k8s-app=kube-dns -n kube-system
pod "coredns-5d555c984-fmrht" deleted
pod "coredns-5d555c984-scvjh" deleted
 
删除旧pod，使用新pod测试 （可以忽略）
[root@kubernetes-master1 /data/kubernetes/service]# kubectl  get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6944855df5-8zjdn   1/1     Running   0          70m
[root@kubernetes-master1 /data/kubernetes/service]# kubectl  delete pod nginx-6944855df5-8zjdn
pod "nginx-6944855df5-8zjdn" deleted
```

```
pod中测试效果
[root@kubernetes-master1 /data/kubernetes/service]# kubectl  exec -it nginx-6944855df5-9s4bd -- /bin/bash
root@nginx-6944855df5-9s4bd:/# apt update ; apt install dnsutils -y
​
测试效果
root@nginx-6944855df5-9s4bd:/# nslookup www.baidu.com
Server:         10.96.0.10
Address:        10.96.0.10#53
​
** server can't find www.baidu.com: SERVFAIL
​
root@nginx-6944855df5-9s4bd:/# nslookup harbor.superopsmsb.com
Server:         10.96.0.10
Address:        10.96.0.10#53
​
Name:   harbor.superopsmsb.com
Address: 10.0.0.20
```
