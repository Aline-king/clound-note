# 数据管理



## 存储方式

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="host存储" %}
通过数据卷 或者 数据卷容器，将当前宿主机中的文件系统目录与容器里面的工作目录进行绑定

支持类型

emptyDir、hostPath、local等
{% endtab %}

{% tab title="网络存储方式" %}
通过网络的方式，将外部的存储空间挂载到当前宿主机，然后借助于host机制实现容器数据的可持久化。

支持类型

* 云存储数据卷 ：gcePersistentDisk、awsElasticBlockStore等
* 网络存储卷：NFS、gitRepo、NAS、SAN等
* 分布式存储卷：GlusterFS、CephFS、rdb(块设备)、iscsi等
* 信息数据卷：flocker、secret等
{% endtab %}
{% endtabs %}

## 资源对象属性

```bash
spec:
  volumes:
  - name <string>  	# 存储卷名称标识，仅可使用DNS标签格式的字符，在当前Pod中必须唯一
    VOL_TYPE <Object>  		# 存储卷插件及具体的目标存储供给方的相关配置
  containers:
  - name: …
    image: …
    volumeMounts:
    - name <string>  		# 要挂载的存储卷的名称，必须匹配存储卷列表中某项的定义
      mountPath <string> 	# 容器文件系统上的挂载点路径
      readOnly <boolean>  	# 是否挂载为只读模式，默认为“否”
      subPath <string>     	# 挂载存储卷上的一个子目录至指定的挂载点
      subPathExpr <string>  	# 挂载由指定的模式匹配到的存储卷的文件或目录至挂载点
```

### 两种资源持久化解决方案

手工方式： PVC + PV&#x20;

自动方式： VCT + SC

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. 由专业的存储管理员来管理所有的存储后端：&#x20;

\- 在专用的存储设备上，创建各种类型级别的PV(Persistent Volume)&#x20;

\- 或者通过存储模板文件SC(storageclasses)来自动创建大量不同类型的PV对象。&#x20;

2. 由开发人员定制需要的PVC(Persistent Volume Claim)，然后关联到pod上 3 Pod通过PVC到PV上请求一块独立大小的网络存储空间，然后直接使用

本质上存储工程师与开发工程师的技能栈不一样，
