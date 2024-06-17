# 证书认证



Kubernetes 集群内部为了实现高质量的安全通信，需要 PKI 证书才能进行基于 TLS 的身份验证。kubeadm 部署的 Kubernetes集群， 会自动生成集群所需的证书。如果我们定制自己的证书，在kubernetes环境中也可以使用。

## 证书管理

{% tabs %}
{% tab title="证书位置" %}
{% hint style="info" %}
kubeadm 部署的 Kubernetes集群，

* 大部分证书都存储在 <mark style="color:yellow;">**/etc/kubernetes/pki**</mark> 目录中，
* 只有kubernetes集群的用户证书文件在 <mark style="color:yellow;">**/etc/kubernetes**</mark> 目录中。
{% endhint %}

{% code title="核心的证书和私钥" %}
```bash
# ls /etc/kubernetes/pki/{ca.*,sa.*,etcd/ca.*}
/etc/kubernetes/pki/ca.crt  /etc/kubernetes/pki/etcd/ca.crt  /etc/kubernetes/pki/sa.pub
/etc/kubernetes/pki/ca.key  /etc/kubernetes/pki/etcd/ca.key
/etc/kubernetes/pki/ca.srl  /etc/kubernetes/pki/sa.key
```
{% endcode %}


{% endtab %}

{% tab title="查看证书有效期" %}
由 kubeadm 默认生成的客户端证书在 1 年后到期，如果需要更新证书的话，kubeadm支持自定义证书来实现更新，前提是必须将证书文件放置在通过

* &#x20;<mark style="color:green;">**--cert-dir**</mark> 命令行参数
* kubeadm 配置中的 certificatesDir 配置项指明的目录中。&#x20;

默认的值是 <mark style="color:blue;">**/etc/kubernetes/pki**</mark>

## <mark style="color:blue;">**方法1**</mark>

{% code title="检测证书是否过期" %}
```bash
# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
```
{% endcode %}

{% hint style="warning" %}
kubeadm 不能管理由外部 CA 签名的证书，否则需要自己手动去更新外部证书&#x20;

kubeadm 和 kubelet 之间的证书同步是自动方式来实现的 默认证书有效期为 1 年，CA 根证书是 10 年
{% endhint %}

```
CERTIFICATE                EXPIRES                  RESIDUAL TIME ...
admin.conf                 Jul 28, 2063 15:17 UTC   364d          ...
apiserver                  Jul 28, 2063 15:15 UTC   364d          ...
apiserver-etcd-client      Jul 28, 2063 15:15 UTC   364d          ...
apiserver-kubelet-client   Jul 28, 2063 15:15 UTC   364d          ...
controller-manager.conf    Jul 28, 2063 15:16 UTC   364d          ...
etcd-healthcheck-client    Jul 24, 2063 11:24 UTC   360d          ...
etcd-peer                  Jul 24, 2063 11:24 UTC   360d          ...
etcd-server                Jul 24, 2063 11:24 UTC   360d          ...
front-proxy-client         Jul 28, 2063 15:15 UTC   364d          ...
scheduler.conf             Jul 28, 2063 15:16 UTC   364d          ...

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   ...
ca                      Jul 21, 2072 11:24 UTC   9y              ...
etcd-ca                 Jul 21, 2072 11:24 UTC   9y              ...
front-proxy-ca          Jul 21, 2072 11:24 UTC   9y              ...
```

## &#x20;方法2

```bash
# openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text |grep ' Not '
            Not Before: Jul 24 11:24:51 2062 GMT
            Not After : Jul 28 15:15:56 2063 GMT
```
{% endtab %}

{% tab title="延长证书有效期" %}
1. 为了避免证书更新对于环境的异常影响，我们这里首先将相关文件进行备份

所有的master节点的操作内容一致

{% hint style="danger" %}
kubeadm alpha 延长证书有效期 ，不推荐使用这个命令

参考：[https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-alpha/](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-alpha/)
{% endhint %}

<pre><code><strong>mkdir /etc/kubernetes-bak
</strong>cd /etc/kubernetes
cp -r $(ls | grep -v tmp) ../kubernetes-bak/
cp -r /var/lib/etcd /etc/kubernetes-bak/lib-etcd
</code></pre>

手动延长

```bash
kubeadm certs renew -h
```

{% hint style="danger" %}
<mark style="color:red;">**注意：**</mark>&#x20;

* &#x20;指定证书，则更新指定的证书
* all 代表更新所有证书&#x20;
* certs renew 使用现有的证书作为属性(CN/O/L等)的权威来源&#x20;
  * 无论证书的到期时间如何，都会无条件地续订一年。&#x20;
  * renew执行后，需要重启各组件，才能生效。
{% endhint %}

<pre><code><strong>This command is not meant to be run on its own. See list of available subcommands.
</strong>Usage:
  kubeadm certs renew [flags]
  kubeadm certs renew [command]
​
Available Commands:
  admin.conf               Renew the certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself
  all                      Renew all available certificates
  ...
</code></pre>

{% code title="所有master节点上更新所有证书" %}
```bash
[root@kubernetes-master1 ~]# kubeadm certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
​
certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
​
Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.
```
{% endcode %}

```
确认效果(两条命令任选其一执行)
[root@kubernetes-master1 ~]# kubeadm certs check-expiration
[root@kubernetes-master1 ~]# openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text |grep ' Not '
```

```
重启组件服务
# kubectl delete pod -n kube-system -l component
​
确认效果
# kubectl get pod -n kube-system -l component
```

\

{% endtab %}
{% endtabs %}



基于账号的token信息，创建kubeconfig文件，实现长久的认证方式。

{% tabs %}
{% tab title="SA" %}
```yaml
apiVersion: v1  			# ServiceAccount所属的API群组及版本
kind: ServiceAccount  			# 资源类型标识
metadata
  name <string>  			# 资源名称
  namespace <string>  			# ServiceAccount是名称空间级别的资源
automountServiceAccountToken <boolean>  # 是否让Pod自动挂载API令牌
secrets <[]Object>   		# 以该SA运行的Pod所要使用的Secret对象组成的列表
  apiVersion <string>  		# 引用的Secret对象所属的API群组及版本，可省略
  kind <string>   		# 引用的资源的类型，这里是指Secret，可省略
  name <string>   		# 引用的Secret对象的名称，通常仅给出该字段即可
  namespace <string>  		# 引用的Secret对象所属的名称空间
  uid  <string>   		# 引用的Secret对象的标识符；
imagePullSecrets <[]Object> 	# 引用的用于下载Pod中容器镜像的Secret对象列表
  name <string>   		# docker-registry类型的Secret资源的名称
```
{% endtab %}

{% tab title="UA实践" %}
**UA实践**

```
    所谓的UA其实就是我们平常做CA时候的基本步骤，主要包括如下几步
    1 创建私钥文件
    2 基于私钥文件创建证书签名请求
    3 基于私钥和签名请求生成证书文件
```

1 创建私钥文件

```
给用户 superopsmsb 创建一个私钥，命名成：superopsmsb.key(无加密)
[root@kubernetes-master1 ~]# cd /etc/kubernetes/pki/
[root@kubernetes-master1 /etc/kubernetes/pki]# (umask 077; openssl genrsa -out superopsmsb.key 2048)
G
命令解析：
    genrsa          该子命令用于生成RSA私钥，不会生成公钥，因为公钥提取自私钥
    -out filename   生成的私钥保存至filename文件，若未指定输出文件，则为标准输出
    -numbits        指定私钥的长度，默认1024，该项必须为命令行的最后一项参数
```

2 签名请求

```
用刚创建的私钥创建一个证书签名请求文件：superopsmsb.csr
[root@kubernetes-master1 /etc/kubernetes/pki]# openssl req -new -key  superopsmsb.key -out superopsmsb.csr -subj "/CN=superopsmsb/O=superopsmsb"
参数说明：
    -new    生成证书请求文件
    -key    指定已有的秘钥文件生成签名请求，必须与-new配合使用
    -out    输出证书文件名称
    -subj   输入证书拥有者信息，这里指定 CN 以及 O 的值，/表示内容分隔
        CN以及O的值对于kubernetes很重要，因为kubernetes会从证书这两个值对应获取相关信息：
            "CN"：Common Name，用于从证书中提取该字段作为请求的用户名 (User Name)；
                浏览器使用该字段验证网站是否合法；
            "O"：Organization，用于分组认证
​
检查效果：
[root@kubernetes-master1 /etc/kubernetes/pki]# ls superopsmsb.*
superopsmsb.csr  superopsmsb.key
结果显示：
    *.key 是我们的私钥，*.csr是我们的签名请求文件
```

3 生成证书

```
利用Kubernetes集群的CA相关证书对UA文件进行认证
[root@kubernetes-master1 /etc/kubernetes/pki]# openssl x509 -req -in superopsmsb.csr -CA ./ca.crt  -CAkey ./ca.key  -CAcreateserial -out superopsmsb.crt -days 365
Signature ok
subject=/CN=superopsmsb/O=superopsmsb
Getting CA Private Key
参数说明：
    -req                产生证书签发申请命令
    -in                 指定需要签名的请求文件
    -CA                 指定CA证书文件
    -CAkey          指定CA证书的秘钥文件
    -CAcreateserial 生成唯一的证书序列号
    -x509               表示输出一个X509格式的证书
    -days               指定证书过期时间为365天
    -out                输出证书文件
检查文件效果
[root@kubernetes-master1 /etc/kubernetes/pki]# ls superopsmsb.*
superopsmsb.crt  superopsmsb.csr  superopsmsb.key
结果显示：
    *.crt就是我们最终生成的签证证书
```

```
提取信息效果
]# openssl x509 -in superopsmsb.crt -text -noout
Certificate:
   ...
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        ...
        Subject: CN=superopsmsb, O=superopsmsb
结果显示：
    Issuer: 表示是哪个CA机构帮我们认证的
    我们关注的重点在于Subject内容中的请求用户所属的组信息
```

**小结**
{% endtab %}
{% endtabs %}

命令解析

```
kubectl create serviceaccount NAME [--dry-run] [options]
```

```
作用：创建一个"服务账号"
参数
    --dry-run=false                     模拟创建模式
    --generator='serviceaccount/v1'     设定api版本信息
    -o, --output=''                     设定输出信息格式，常见的有：
                                            json|yaml|name|template|...
    --save-config=false                 保存配置信息
    --template=''                       设定配置模板文件
```

简单实践

```
创建专属目录
[root@kubernetes-master1 ~]# mkdir /data/kubernetes/secure -p; cd /data/kubernetes/secure
```

```
手工创建SA账号
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl create serviceaccount mysa
serviceaccount/mysa created
​
检查效果
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  get sa mysa
NAME   SECRETS   AGE
mysa   1         9s
```

```
资源清单创建SA
[root@kubernetes-master1 /data/kubernetes/secure]# 01_kubernetes_secure_sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: superopsmsb
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-web
spec:
  containers:
  - name: nginx-web
    image: kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1
  serviceAccountName: superopsmsb
​
应用资源清单
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl  apply -f 01_kubernetes_secure_sa.yaml
serviceaccount/superopsmsb created
pod/nginx-web created
```

```
查看资源属性
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl describe pod nginx-web
Name:               nginx-web
... 
Containers:
  nginx-web:
    ...
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wqb6w (ro)
...
结果显示：
    用户信息挂载到容器的 /var/run/secrets/kubernetes.io/serviceaccount 目录下了
```

```
容器内部的账号身份信息
[root@kubernetes-master1 /data/kubernetes/secure]# kubectl exec -it nginx-web -- /bin/sh
# ls /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt  namespace  token
```

