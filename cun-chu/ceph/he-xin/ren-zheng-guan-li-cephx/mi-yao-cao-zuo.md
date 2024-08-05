# 秘钥操作

密钥环文件是“存储机密、密码、密钥、证书并使它们可用于应用程序的组件的集合”。

密钥环文件存储一个或多个 Ceph 身份验证密钥以及可能的相关功能规范。 ​

{% hint style="warning" %}
每个键都与一个实体名称相关联，形式为 **{client,mon,mds,osd}.name**。&#x20;

<mark style="color:purple;">ceph-authtool</mark> 是一个用于创建、查看和修改 Ceph 密钥环文件的实用程序。
{% endhint %}

## 秘钥环文件信息

访问Ceph集群时，客户端会于本地查找密钥环，只有，认证成功的对象，才可以正常使用。

默认情况下，Ceph会使用以下四个密钥环名称预设密钥环&#x20;

1. /etc/ceph/cluster-name.user-name.keyring：保存单个用户的keyring&#x20;
2. /etc/ceph/cluster.keyring：保存多个用户的keyring&#x20;
3. /etc/ceph/keyring&#x20;
4. /etc/ceph/keyring.bin

{% hint style="info" %}
<mark style="color:purple;">**cluster-name**</mark>是为集群名称，

<mark style="color:orange;">**user-name**</mark>是为用户标识（TYPE.ID）&#x20;

<mark style="color:red;">**client.admin**</mark>用户的在名为ceph的集群上的密钥环文件名为ceph.client.admin.keyring
{% endhint %}

## keyring的管理

{% tabs %}
{% tab title="创建keyring" %}
ceph auth add等命令添加的用户还需要额外使用ceph-authtool命令为其创建用户密钥环文件，ceph客户端通过keyring文件查找用户名并检索密钥&#x20;

命令：ceph-authtool --create-keyring /path/to/kerying

keyring文件一般应该保存于/etc/ceph目录中，以便客户端能自动查找&#x20;

* 创建包含多个用户的keyring文件时，应该使用cluster-name.keyring作为文件名&#x20;
* 创建仅包含单个用户的kerying文件时，应该使用cluster-name.user-name.keyring作为文件名
{% endtab %}

{% tab title="将用户添加至keyring" %}
1. 可将某个用户从包含多个用户的keyring中导出，并保存于一个专用的keyring文件&#x20;

命令：ceph auth get TYPE.ID -o /etc/ceph/cluster-name.user-name.keyring

2. &#x20;也可将用户的keyring合并至一个统一的keyring文件中&#x20;

命令：ceph-authtool /etc/ceph/cluster-name.keyring --import-key /etc/ceph/clustername. user-name.keyring
{% endtab %}

{% tab title="ceph-authtool创建keyring" %}
ceph-authtool命令可直接创建用户、授予caps并创建keyring&#x20;

```bash
ceph-authtool keyringfile [-C | --create-keyring] [-n | --name entityname] [--genkey] [-a | --add-key base64_key] [--cap | --caps capfile]
```

此种方式添加的用户仅存在于keyring文件中，管理员还需要额外将其添加至Ceph集群上；&#x20;

命令： ceph auth add TYPE.ID -i /PATH/TO/keyring
{% endtab %}
{% endtabs %}

**简单实践**

{% tabs %}
{% tab title="创建携带秘钥环的账号" %}
```
创建普通格式的用户
[cephadm@admin ceph-cluster]$ ceph auth get-or-create client.kube mon 'allow r' osd 'allow * pool=kube'
[client.kube]
        key = AQA11TBjzq6NEhAAF2tyvEY0L+XYMP7IOykkcQ==
```

```
查看用户信息
[cephadm@admin ceph-cluster]$ ceph auth get client.kube
[client.kube]
        key = AQA11TBjzq6NEhAAF2tyvEY0L+XYMP7IOykkcQ==
        caps mon = "allow r"
        caps osd = "allow * pool=kube"
exported keyring for client.kube
```

没有生成对应的用户秘钥环文件

```
查看文件
[cephadm@admin ceph-cluster]$ ls
ceph.bootstrap-mds.keyring  ceph.bootstrap-rgw.keyring  ceph-deploy-ceph.log
ceph.bootstrap-mgr.keyring  ceph.client.admin.keyring   ceph.mon.keyring
ceph.bootstrap-osd.keyring  ceph.conf                   testuser.file                  
```
{% endtab %}

{% tab title="导出秘钥环文件" %}
```
将普通的用户导出为keyring
[cephadm@admin ceph-cluster]$ ceph auth get client.kube -o ceph.client.kube.keyring
exported keyring for client.kube
```

```
查看效果
[cephadm@admin ceph-cluster]$ ll *kube*
-rw-rw-r-- 1 cephadm cephadm 117 9月  26 06:26 ceph.client.kube.keyring
[cephadm@admin ceph-cluster]$ cat ceph.client.kube.keyring
[client.kube]
        key = AQA11TBjzq6NEhAAF2tyvEY0L+XYMP7IOykkcQ==
        caps mon = "allow r"
        caps osd = "allow * pool=kube"
```


{% endtab %}

{% tab title="合并秘钥环文件" %}
```
创建要合并的文件
[cephadm@admin ceph-cluster]$ ceph-authtool --create-keyring cluster.keyring
creating cluster.keyring
[cephadm@admin ceph-cluster]$ ll cluster.keyring
-rw------- 1 cephadm cephadm 0 9月  26 06:28 cluster.keyring
​
合并要导入的keyring文件
[cephadm@admin ceph-cluster]$ ceph-authtool cluster.keyring --import-keyring ceph.client.kube.keyring
importing contents of ceph.client.kube.keyring into cluster.keyring
[cephadm@admin ceph-cluster]$ cat cluster.keyring
[client.kube]
        key = AQA11TBjzq6NEhAAF2tyvEY0L+XYMP7IOykkcQ==
        caps mon = "allow r"
        caps osd = "allow * pool=kube"
```

```
再来合并一个用户
c[cephadm@admin ceph-cluster]$ ceph-authtool cluster.keyring --import-keyring ceph.client.admin.keyring
importing contents of ceph.client.admin.keyring into cluster.keyring
​
查看合并后效果
[cephadm@admin ceph-cluster]$ cat cluster.keyring
[client.admin]
        key = AQAr0S9jK8RNLxAAD/xpDZgnAx1kQEV3+/utWw==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
[client.kube]
        key = AQA11TBjzq6NEhAAF2tyvEY0L+XYMP7IOykkcQ==
        caps mon = "allow r"
        caps osd = "allow * pool=kube"
```

```
专用查看keyring的内容
[cephadm@admin ceph-cluster]$ ceph-authtool -l cluster.keyring
[client.admin]
        key = AQAr0S9jK8RNLxAAD/xpDZgnAx1kQEV3+/utWw==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
[client.kube]
        key = AQA11TBjzq6NEhAAF2tyvEY0L+XYMP7IOykkcQ==
        caps mon = "allow r"
        caps osd = "allow * pool=kube"
```


{% endtab %}

{% tab title="Untitled" %}

{% endtab %}
{% endtabs %}
