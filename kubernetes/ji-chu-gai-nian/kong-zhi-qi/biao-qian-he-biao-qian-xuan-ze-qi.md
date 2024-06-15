# 标签和标签选择器

本质：label是一个key/value键值对，其中key与value由用户自己指定

注意： key的命名：由"字母、数字、\_、.、-"这五类组成,只能以字符或数字作为开头和结尾。&#x20;

标签名称不能多于63个字符

作用： 一个资源对象可以定义多个Lable，同一个Lable也可以关联多个资源对象上去 通过对Lable的管理从而达到对同Lable的资源进行分组管理(分配、调度、配置、部署等)。

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## 使用方法

{% tabs %}
{% tab title="资源清单方法" %}
在特定的属性后面按照指定方式增加内容即可，

格式如下： labels: key: value&#x20;

注意： labels 是 一个复数格式。
{% endtab %}

{% tab title="命令行方法" %}
查看标签

```bash
kubectl get pods -l label_name=label_value
```

{% hint style="info" %}
参数：&#x20;

* \-l 就是指定标签条件，获取指定资源对象
* \=表示匹配，
* != 表示不匹配 ，
  * 如果后面的选择标签有多个的话，使用逗号隔开&#x20;
*   如果针对标签的值进行范围过滤的话，可以使用如下格式：&#x20;

    <mark style="color:orange;">**-l "label\_name in|notin (value1, value2, value3, ...)"**</mark>
{% endhint %}

增加标签

```bash
kubectl label 资源类型 资源名称 label_name=label_value
```

{% hint style="info" %}
参数：

同时增加多个标签，只需要在后面多写几个就可以了，使用空格隔开&#x20;

默认情况下，已存在的标签是不能修改的，使用 <mark style="color:yellow;">--overwrite=true</mark> 表示强制覆盖 label\_name=label\_value样式写成 <mark style="color:red;">label\_name-</mark> ，表示删除label
{% endhint %}
{% endtab %}
{% endtabs %}

## 标签选择器

Lablel附加到Kubernetes集群中的各种资源对象上，目的就是对这些资源对象进行分组管理，而分组管理的核心就是：Lablel Selector。

Lablel Selector跟Label一样，不能单独定义，必须附加在一些资源对象的定义文件上。一般附加在RC和Service等控制器资源对象文件中。

### 使用方法

{% hint style="success" %}
基础Label Selector表达式

等式：&#x20;

name = nginx 匹配所有具有标签 name = nginx 的资源对象&#x20;

name != nginx 匹配所有不具有标签 name = nginx 的资源对象&#x20;

集合：&#x20;

env in (dev, test) 匹配所有具有标签 env = dev 或者 env = test 的资源对象&#x20;

name not in (frontend) 匹配所有不具有标签 name = frontend 的资源对象
{% endhint %}

{% hint style="info" %}
扩展匹配Label Selector表达式

匹配标签：

&#x20;matchLabels: name: nginx&#x20;

匹配表达式：&#x20;

matchExpressions: - {key: name, operator: NotIn, values: \[frontend]}
{% endhint %}
