# 授权

## 授权机制

Kubernetes的基本特性就是它的所有资源对象都是模型化的 API 对象，我们可以基于api-server对各种资源进行增、删、改、查等操作，

但是这些操作涉及到的不仅仅是资源本身和动作，而且还涉及到资源和动作之间的内容，比如api的分组版本和资源和api的关联即权限授权等。

### 授权方式

* <mark style="color:yellow;">**Node**</mark> 主要针对节点的基本通信授权认证，比如kubelet的正常通信。&#x20;
* <mark style="color:yellow;">**ABAC(Attribute Based Access Control)**</mark>： 基于属性的访问控制
  * 与apiserver服务的本地策略规则紧密相关，一旦变动需要重启。&#x20;
* <mark style="color:yellow;">**RBAC(Role Based Access Control)**</mark>： 基于角色的访问控制，
  * 这是1.6+版本主推的授权策略，可以使用API自定义角色和集群角色，并将角色和特定的用户，用户组，Service Account关联起来，可以用来实现[**多租户隔离功能**](#user-content-fn-1)[^1]

<figure><img src="../../../../.gitbook/assets/image (8) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<div align="left">

<figure><img src="../../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

</div>

### 授权级别

<figure><img src="../../../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="namespace级别" %}
* <mark style="color:orange;">**rules**</mark>
  * 规则，是一组属于不同 API Group 资源上的权限操作的集合
* <mark style="color:yellow;">**role**</mark>
  * &#x20;表示在一个namespace中基于rules使用资源的权限，主要涉及到操作和对象
* <mark style="color:orange;">**RoleBingding**</mark>&#x20;
  * 将Subject和Role绑定在一起，表示Subject具有指定资源的role操作权限
{% endtab %}

{% tab title="cluster级别" %}
* <mark style="color:yellow;">**ClusterRole：**</mark>表示在一个Cluster中基于rules使用资源的权限&#x20;
* <mark style="color:yellow;">**ClusterRoleBingding：**</mark>将Subject和ClusterRole绑定在一起，表示Subject指定资源的ClusterRole操作权限
{% endtab %}

{% tab title="混合级别" %}
<mark style="color:yellow;">**RoleBingding**</mark>：将Subject基于RoleBingding与ClusterRole绑定在一起&#x20;

* 表示Subject可以使用所有ClusterRole中指定资源的role角色 ，但是限制在了只能操作某个命名空间的资源，也就是所谓的"权限降级"
{% endtab %}
{% endtabs %}

#### 使用场景

<mark style="color:purple;">**场景1：**</mark> 多个namespace中的role角色都一致，如果都使用内部的RoleBingding的话，每个namespace都必须单独创建role，而使用ClusterRole的话，只需要一个就可以了，大大的减轻批量使用namespace中的RoleBingding 操作。&#x20;

<mark style="color:purple;">**场景2：**</mark> 我们对A用户要提升权限，但是，由于A处于考察期，那么我们暂时给他分配一个区域，测试一下它的运行效果。生活中的场景，提升张三为公司副总裁，但是由于是新手，所以加了一个限制 -- 主管销售范围的副总裁。

## 属性解析

<details>

<summary>查看Role的属性信息</summary>

```bash
kubectl explain role
```

对于role来说，其核心的内容主要是rules的权限规则       &#x20;

在这么多rules属性中，最重要的是verbs权限条目，而且所有的属性都是可以以列表的形式累加存在

```yaml
    apiVersion <string>
    kind <string>
    metadata     <Object>
    rules        <[]Object>
      apiGroups             <[]string>
      nonResourceURLs       <[]string>
      resourceNames         <[]string>
      resources             <[]string>
      verbs                 <[]string> -required-  # 最重要
```

### Role核心

[关于api组的信息获取](https://kubernetes.io/docs/reference/#api-reference)

命令行创建role，查看具有pod资源的get、list权限的属性信息

```yaml
# kubectl create role pods-reader --verb=get,list --resource=pods --dry-run -o yaml
```

对于一个role必备的rules来说，他主要有三部分组成：   &#x20;

* <mark style="color:red;">**apiGroups：**</mark>设定包含资源的api组，如果是多个，表示只要属于api组范围中任意资源都可以操作   &#x20;
* <mark style="color:green;">**resources**</mark>：位于apiGroup范围中的某些具体的资源对象   &#x20;
* <mark style="color:purple;">**verbs：**</mark>针对具体资源对象的一些具体操作

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null       # 时间信息
  name: pods-reader             # role的名称
rules:                          # 授权规则
- apiGroups:                    # 操作的对象
  - ""                          # 所有权限
  resources:                    # 资源对象
  - pods                        # pod的对象
  verbs:                        # 对pod允许的权限
  - get                         # 获取
  - list                        # 查看
```

</details>

<details>

<summary>clusterrole属性信息</summary>

```bash
# kubectl explain clusterrole
```

clusterrole相对于role的属性多了一个集中控制器的属性aggregationRule，这是一个可选的属性

```bash
aggregationRule     <Object>
apiVersion <string>
kind <string>
metadata     <Object>
rules        <[]Object>
    apiGroups               <[]string>
    nonResourceURLs         <[]string>
    resourceNames           <[]string>
    resources               <[]string>
    verbs                   <[]string> -required-
```

命令行创建clusterrole，查看具有pod资源的get、list权限的属性信息

<pre><code><strong># kubectl create clusterrole myclusterrole --verb=get,list --resource=pods --dry-run -o yaml
</strong></code></pre>

&#x20;ClusterRole与role的配置一样，也由三部分组成：apiGroup、resources、verbs

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null       # 时间信息
  name: myclusterrole           # role的名称
rules:                          # 授权规则
- apiGroups:                    # 操作的对象
  - ""                          # 所有权限
  resources:                    # 资源对象
  - pods                        # pod的对象
  verbs:                        # 对pod允许的权限
  - get                          获取
  - list                          查看
```

</details>

## 实践

<details>

<summary>Role用户授权实践</summary>

```bash
限定用户只能访问命名空间资源
# kubectl create rolebinding super-rolebind --role=myrole --user=superopsmsb
rolebinding.rbac.authorization.k8s.io/super-rolebind created
​
查看资源效果
# kubectl  get pod --kubeconfig=/tmp/superopsmsb.conf
NAME        READY   STATUS    RESTARTS   AGE
nginx-web   1/1     Running   0          56m
# kubectl  get svc --kubeconfig=/tmp/superopsmsb.conf
Error from server (Forbidden): services is forbidden: User "superopsmsb" cannot list resource "services" in API group "" in the namespace "default"
# kubectl  get svc --kubeconfig=/tmp/superopsmsb.conf -n kube-system
Error from server (Forbidden): services is forbidden: User "superopsmsb" cannot list resource "services" in API group "" in the namespace "kube-system"
​
清理授权
# kubectl delete rolebindings super-rolebind
rolebinding.rbac.authorization.k8s.io "super-rolebind" deleted
```



</details>

<details>

<summary><strong>ClusterRole实践</strong></summary>

写一个clusterrole资源文件,允许用户操作 Deployment、Pod、RS 的所有权限

```
# 03_kubernetes_secure_clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: myclusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

```
创建资源对象
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  apply -f 03_kubernetes_secure_clusterrole.yaml
clusterrole.rbac.authorization.k8s.io/myclusterrole created
```

```
查看效果
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl describe clusterrole myclusterrole
Name:         myclusterrole
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get list watch]
```

```
限定用户只能访问命名空间资源
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl create clusterrolebinding super-clusterrolebind --clusterrole=myclusterrole --user=superopsmsb
​
查看资源效果
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  get pod --kubeconfig=/tmp/superopsmsb.conf
NAME        READY   STATUS    RESTARTS   AGE
nginx-web   1/1     Running   0          68m
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  get svc --kubeconfig=/tmp/superopsmsb.conf
Error from server (Forbidden): services is forbidden: User "superopsmsb" cannot list resource "services" in API group "" in the namespace "default"
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  get pod --kubeconfig=/tmp/superopsmsb.conf -n kube-system
NAME                                         READY   STATUS    RESTARTS         AGE
coredns-5d555c984-hzq8q                      1/1     Running   0                9h
​
清理授权
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl delete clusterrolebinding super-clusterrolebind
rolebinding.rbac.authorization.k8s.io "super-clusterrolebind" deleted
```



</details>





简单实践

[^1]: 基于namespace资源
