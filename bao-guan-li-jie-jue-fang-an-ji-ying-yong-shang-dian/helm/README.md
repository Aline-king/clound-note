# Helm

## 引入helm原因

当今的软件开发，随着云原生技术的普及，我们的工程应用进行微服务化和容器化的现象也变得越来越普遍。而Kubernetes几乎已经成了云原生服务编排绕不开的标准和技术。

假设我们需要在K8s中简单部署一个nginx，必要步骤如下：



{% tabs %}
{% tab title="创建" %}
```
# kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```


{% endtab %}

{% tab title="pod" %}
启动pod

```
# kubectl apply -f deployment.yaml
```

检查pod服务

```
# kubectl get pod
```
{% endtab %}

{% tab title="service" %}
创建service

```
# kubectl expose deployment  nginx --port=8099 --target-port=80 --type=NodePort --dry-run=client -o yaml > service.yaml
```

启动service服务

```
# kubectl apply -f service.yaml
```

检查service端口

```
# kubectl get svc
```


{% endtab %}

{% tab title="访问" %}
访问nginx服务

<figure><img src="../../.gitbook/assets/image-20220728133433498.png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

实际生产中，微服务项目可能有十几个模块，若还需要进行安全访问和控制，那么需要创建诸如Role、ServiceAccount等资源。部署和版本升级时也往往需要修改或添加配置文件中的一些参数（例如：服务占用的CPU、内存、副本数、端口等），维护大量的yaml文件极为不便，所以，我们需要将这些YAML文件作为一个**整体**管理，并高效复用。

* 在Linux操作系统软件部署中，我们可以使用批量管理工具完成软件的批量管理等，例如yum、dnf等；
* 在容器应用中Docker使用Dockerfile文件解决了容器镜像制作难题；
* 在kubernetes应用中，通过YAML格式文件解决容器编排部署难题，例如可以通过YAML格式的资源清单文件，非常方便部署不同控制器类型的应用，但是如何维护大量的，系统性的YAML文件，需要我们拥有更好的工具，不能简单使用YAML资源清单托管服务器就可以解决的，

那么在CNCF的体系中是否存在这样的强力“工具”，能够简化我们部署安装过程呢？答案是存在的，Helm就是这样一款工具。
