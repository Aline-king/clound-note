# 无头服务

## handless service

在kubernetes中，有一种特殊的无头service，它本身是Service，但是没有ClusterIP，这种svc在一些有状态的场景中非常重要。

### 使用场景

无头服务场景下，k8s会将集群内部提供唯一的<mark style="color:yellow;">**DNS域名**</mark>作为每个成员的网络标识，集群内部成员之间<mark style="color:orange;">**使用域名通信**</mark>，这个时候，就特别依赖service的selector属性配置了。&#x20;

#### 无头服务的域名格式

<mark style="color:green;">**$(service\_name).$(k8s\_namespace).svc.cluster.local**</mark>

### pod资源对象的解析记录

dns解析记录&#x20;

svc\_name的解析结果从常规Service的ClusterIP，转为各个Pod的IP地址；

&#x20;反解，则从常规的clusterip解析为service name，转为从podip到hostname, ---...svc. 指的是a-b-c-d格式，而非Pod自己的主机名；

```
A记录
<a>-<b>-<c>-<d>.<service>.<ns>.svc.<zone>  A  PodIP
```
