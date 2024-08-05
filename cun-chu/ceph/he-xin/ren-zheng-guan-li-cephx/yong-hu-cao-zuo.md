# 用户操作

Ceph集群管理员能够直接在Ceph集群中创建、更新和删除用户

{% hint style="warning" %}
创建用户时，可能需要将密钥分发到客户端，以便将密钥添加到密钥环
{% endhint %}

## 常见命令

<table><thead><tr><th width="121"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>列出用户</td><td><p><code>ceph auth list</code> </p><p><strong>用户标识</strong>：TYPE.ID，因此，osd.0表示OSD类型的用户，用户ID为0</p></td><td></td></tr><tr><td>检索特定用户</td><td><p>二者选其一导入用户</p><p>ceph auth get TYPE.ID</p><p>ceph auth export TYPE.ID</p></td><td></td></tr><tr><td>列出用户的密钥</td><td>ceph auth print-key TYPE.ID</td><td></td></tr><tr><td>添加用户</td><td><p><mark style="color:purple;"><strong>ceph auth add</strong></mark>：创建用户、生成密钥并添加指定的</p><p><mark style="color:purple;"><strong>caps ceph auth get-or-create</strong></mark>：创建用户并返回密钥文件格式的密钥信息，用户存在时返回密钥信息 </p><p><mark style="color:purple;"><strong>ceph auth get-or-create-key</strong></mark>：创建用户并返回密钥信息，用户存在时返回密钥信息<br><mark style="background-color:orange;">典型的用户至少对 Ceph monitor 具有读取功能，并对 Ceph OSD 具有读取和写入功能；</mark> </p><p><mark style="background-color:orange;">另外，用户的 OSD 权限通常应该限制为只能访问特定的存储池， 否则，他将具有访问集群中所有存储池的权限</mark></p></td><td></td></tr><tr><td>导入用户</td><td>ceph auth import</td><td></td></tr><tr><td>删除用户</td><td>ceph auth del TYPE.ID</td><td></td></tr><tr><td>修改用户caps</td><td><p>ceph auth caps<br>会覆盖用户现有的caps，因此建立事先使用ceph auth get TYPE.ID命令查看用户的caps </p><p>若是为添加caps，则需要先指定现有的caps,命令格式如下： </p><p><strong>ceph auth caps TYPE.ID daemon 'allow [r|w|x|*|...] [pool=pool-name]' ...</strong></p></td><td></td></tr></tbody></table>

{% hint style="warning" %}

{% endhint %}

## 实践

{% tabs %}
{% tab title="查看用户账号" %}
```
[cephadm@admin ceph-cluster]$ ceph auth list
osd.0
        key: AQCv3i9j34BWKhAAII3+THENuLJTv5Yr2NwDxA==
        caps: [mgr] allow profile osd
        caps: [mon] allow profile osd
        caps: [osd] allow *
...
```

每个osd都有类似的caps权限信息&#x20;

client.admin具备的权限是非常多的
{% endtab %}

{% tab title="查看制定的认证信息" %}
```
[cephadm@admin ceph-cluster]$ ceph auth get osd.0
[osd.0]
        key = AQCv3i9j34BWKhAAII3+THENuLJTv5Yr2NwDxA==
        caps mgr = "allow profile osd"
        caps mon = "allow profile osd"
        caps osd = "allow *"
exported keyring for osd.0
```
{% endtab %}

{% tab title="秘钥文件信息" %}
<mark style="color:purple;">**查看秘钥文件信息**</mark>

```
$ ls /etc/ceph/
ceph.client.admin.keyring  ceph.conf  rbdmap  tmpr0oj5H    
```

<mark style="color:purple;">**查看秘钥环相关的信息**</mark>

一个秘钥环里面可以存放很多秘钥信息的

```
$ cat /etc/ceph/ceph.client.admin.keyring
[client.admin]
        key = AQAr0S9jK8RNLxAAD/xpDZgnAx1kQEV3+/utWw==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
```

<mark style="color:purple;">**列出用户秘钥信息**</mark>

```
$ ceph auth print-key mgr.mon01
AQC12C9j/hAgKhAA6lOZiUpz5kqA55PFv3eHng==
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="添加用户" %}
<mark style="color:purple;">**创建普通用户**</mark>

```bash
[cephadm@admin ceph-cluster]$ ceph auth add client.testuser mon 'allow r' osd 'allow rw pool=rdbpool'
added key for client.testuser
```

<mark style="color:purple;">**获取创建的用户信息**</mark>

```bash
[cephadm@admin ceph-cluster]$ ceph auth get client.testuser
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
        caps mon = "allow r"
        caps osd = "allow rw pool=rdbpool"
exported keyring for client.testuser
```

<mark style="color:purple;">**列出用户的秘钥信息**</mark>

```bash
[cephadm@admin ceph-cluster]$ ceph auth print-key client.testuser
AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
```
{% endtab %}

{% tab title="用户授权" %}
```bash
修改用户的授权
[cephadm@admin ceph-cluster]$ ceph auth caps client.testuser mon 'allow rw' osd 'allow rw pool=rdbpool1'
updated caps for client.testuser
​
查看修改后的授权信息
[cephadm@admin ceph-cluster]$ ceph auth get client.testuser
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
        caps mon = "allow rw"
        caps osd = "allow rw pool=rdbpool1"
exported keyring for client.testuser
```


{% endtab %}

{% tab title="导出用户" %}
```bash
查看导出信息
[cephadm@admin ceph-cluster]$ ceph auth export client.testuser
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
        caps mon = "allow rw"
        caps osd = "allow rw pool=rdbpool1"
export auth(key=AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==)
```

```
导出信息到一个备份文件
[cephadm@admin ceph-cluster]$ ceph auth export client.testuser > testuser.file
export auth(key=AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==)
​
[cephadm@admin ceph-cluster]$ cat testuser.file
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
        caps mon = "allow rw"
        caps osd = "allow rw pool=rdbpool1"
```


{% endtab %}

{% tab title="删除用户" %}
```bash
删除用户信息
[cephadm@admin ceph-cluster]$ ceph auth del client.testuser
updated
​
查看效果
[cephadm@admin ceph-cluster]$ ceph auth get client.testuser
Error ENOENT: failed to find client.testuser in keyring
```
{% endtab %}

{% tab title="导入用户" %}
```bash
# 导入用户文件
[cephadm@admin ceph-cluster]$ ceph auth  import -i testuser.file
imported keyring
​
# 查看文件效果
[cephadm@admin ceph-cluster]$ ceph auth get client.testuser
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
        caps mon = "allow rw"
        caps osd = "allow rw pool=rdbpool1"
exported keyring for client.testuser
```


{% endtab %}

{% tab title="尝试创建一个未知的用户" %}
```bash
创建一个已知的用户
[cephadm@admin ceph-cluster]$ ceph auth get-or-create client.testuser
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
结果显示：
    如果是一个已存在的用户名，则会返回具体的信息，而且不会覆盖现有的用户信息
    
[cephadm@admin ceph-cluster]$ ceph auth get client.testuser
[client.testuser]
        key = AQDfOTBjsmRaEhAATcpXwDluXBpshy4420/zgg==
        caps mon = "allow rw"
        caps osd = "allow rw pool=rdbpool1"
exported keyring for client.testuser
```

```
创建一个未知的用户
[cephadm@admin ceph-cluster]$ ceph auth get-or-create client.testuser2 mon 'allow r' osd 'allow rw pool=rdbpool'
[client.testuser2]
        key = AQAwPDBjHPByABAA+nMEvAXztrzE8FSCZkNGgg==
        
如果从未出现过的用户，则直接创建新的用户
[cephadm@admin ceph-cluster]$ ceph auth get client.testuser2
[client.testuser2]
        key = AQAwPDBjHPByABAA+nMEvAXztrzE8FSCZkNGgg==
        caps mon = "allow r"
        caps osd = "allow rw pool=rdbpool"
exported keyring for client.testuser2
```
{% endtab %}
{% endtabs %}
