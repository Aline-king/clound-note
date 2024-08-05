# 存储实践

## 存储术语

<table data-header-hidden><thead><tr><th width="84"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>Pool</td><td>RADOS存储集群提供的基础存储服务需要由“存储池（pool）”分割为<mark style="color:orange;"><strong>逻辑存储 区域</strong></mark>，此类的逻辑区域亦是对象数据的名称空间。</td><td></td></tr><tr><td>PG</td><td><p>归置组（Placement Group）是用于跨OSD将数据存储在某个存储池中的内部数据 </p><p>结构 相对于存储池来说，PG是一个虚拟组件，它是对象映射到存储池时使用的虚拟层 是实现大容量集群的关键效率技术</p></td><td></td></tr><tr><td>PGP</td><td>(Placement Group for Placement)是用于维持PG和OSD的一种策略。 防止OSD重新分配时候，PG找不到之前的OSD，从而引起大范围的数据迁移</td><td></td></tr><tr><td>CRUSH</td><td><p>把对象直接映射到OSD之上会导致二者之间的紧密耦合关系，在OSD设备变动时不可避免地对整个集群产生扰动。所以需要一种策略算法来处理这种问题。 Ceph将一个对象映射进RADOS集群的过程分为两步 </p><ul><li>首先是以一致性哈希算法将对象名称映射到PG </li><li><p>而后是将PG ID基于<mark style="color:purple;"><strong>CRUSH算法</strong></mark>映射到OSD CRUSH(Controlled Replication Under Scalable Hashing)</p><blockquote><p>它是一种数据分布式算法，类似于一致性哈希算法，用于为RADOS存储集群控制数据分布。</p></blockquote></li></ul></td><td></td></tr></tbody></table>

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 数据存储

{% tabs %}
{% tab title="创建存储池" %}
```bash

ceph osd pool create <pool-name> <pg-num> [pgp-num] [replicated] \ [crush-rule-name] [expected-num-objects]
		pool-name：存储池名称，在一个RADOS存储集群上必须具有唯一性；
		pg-num：当前存储池中的PG数量，一定要合理
		pgp-num ：用于归置的PG数量，其值应该等于PG的数量
		replicated：存储池类型；副本存储池需更多原始存储空间，但已实现Ceph支持的所有操 作
		crush-ruleset-name：此存储池所用的CRUSH规则集的名称，引用的规则集必须事先存在
查看命令
	ceph osd pool ls
	rados lspools
```



```
创建一个存储池，名称为mypool，pg数量和pgp数量都是16
[cephadm@admin ceph-cluster]$ ceph osd pool create mypool 16 16
pool 'mypool' created

查看存储池的列表
[cephadm@admin ceph-cluster]$ ceph osd pool ls
mypool
[cephadm@admin ceph-cluster]$ rados lspools
mypool
```
{% endtab %}

{% tab title="数据的上传" %}
虽然我们目前没有形成专用的数据接口，但是ceph提供了一个原理的文件测试接口 -- rados命令 `rados put 文件对象名(id) /path/to/file --pool=存储池`

```bash
提交文件到对应的osd里面
[cephadm@admin ceph-cluster]$ rados put ceph-file /home/cephadm/ceph-cluster/ceph.conf --pool=mypool

确认上传数据效果
[cephadm@admin ceph-cluster]$ rados ls --pool=mypool
ceph-file
```
{% endtab %}

{% tab title="查看数据的存储关系" %}
通过属性的方式获取到存储池中数据对象的具体位置信息&#x20;

`ceph osd map 存储池 文件对象名(id)`

```
查看ceph-file文件对象的内部属性关系
[cephadm@admin ceph-cluster]$ ceph osd map mypool ceph-file
osdmap e51 pool 'mypool' (1) object 'ceph-file' -> pg 1.7753490d (1.d) -> up ([2,1,5], p2) acting ([2,1,5], p2)
```

可以看到文件对象的内部属性关系 \[ num ] 是副本所存储的osd id的值
{% endtab %}

{% tab title="数据删除实践" %}
将文件对象从pool里面删除&#x20;

`rados rm 文件对象名(id) --pool=存储池`

```bash
将刚才添加的文件对象从存储池里面移除
[cephadm@admin ceph-cluster]$ rados rm ceph-file --pool=mypool

查看存储池的内容
[cephadm@admin ceph-cluster]$ rados ls --pool=mypool
[cephadm@admin ceph-cluster]$
```
{% endtab %}
{% endtabs %}

## 存储解析

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>pool 是ceph存储数据时的逻辑分区，它起到据不同的用户场景，基于namespace实现隔离故障域的作用。 每个pool包含一定数量的PG, PG里的对象被映射到不同的OSD上。 OSD分散到所有的主机磁盘上</p></figcaption></figure>

{% tabs %}
{% tab title="存储池" %}
```
ceph osd pool ls [detail]
ceph osd pool stats {<poolname>}
```

对于mypool来说，它的id是1，该存储池里面的所有pg都是以1为开头的

```
查看存储池名称
[cephadm@admin ceph-cluster]$ ceph osd pool ls
mypool

查看存储池详情
[cephadm@admin ceph-cluster]$ ceph osd pool ls detail
pool 1 'mypool' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 16 pgp_num 16 autoscale_mode warn last_change 51 flags hashpspool stripe_width 0

确认存储池状态
[cephadm@admin ceph-cluster]$ ceph osd pool stats mypool
pool mypool id 1
  nothing is going on
```
{% endtab %}

{% tab title="存储池&PG" %}
```
通过 pg 查找 pool
	ceph pg dump | grep "^{poolid}\."
 
通过 pool 查找 pg
	ceph pg ls-by-pool {poolname}
	ceph pg ls {poolid}
```

```bash
# 根据pg查找pools
[cephadm@admin ceph-cluster]$ ceph pg dump | grep "^1. "
...
[cephadm@admin ceph-cluster]$ ceph pg dump pools
POOLID OBJECTS ...
1            0 ...

# 根据存储池找PG
[cephadm@admin ceph-cluster]$ ceph pg ls-by-pool mypool | awk '{print $1,$2,$15}'
PG OBJECTS ACTING
1.0 0 [3,5,0]p3
...
1.e 0 [2,1,5]p2
1.f 0 [3,1,4]p3

# 看出mypool的pg都是以1开头的,确定pg的分布情况
[cephadm@admin ceph-cluster]$ ceph pg dump pgs|grep ^1|awk '{print $1,$2,$15,$19}'
dumped pgs
1.f 0 0'0 [3,1,4]
...
1.9 0 0'0 [3,4,1]
```

每个pg都会分布在三个osd上，整个集群有6个osd
{% endtab %}

{% tab title="PG & OSD" %}
```bash
通过 pg 查找 osd
	ceph pg map {pgid}
通过 osd 查找 pg
	ceph pg ls-by-osd osd.{osdid}
```

```bash
# 根据pg找osd的分布
[cephadm@admin ceph-cluster]$ ceph pg map 1.1
osdmap e51 pg 1.1 (1.1) -> up [5,0,2] acting [5,0,2]

# 根据osd找pg的分布
[cephadm@admin ceph-cluster]$ ceph pg ls-by-osd osd.1 | awk '{print $1,$2,$10,$14,$15}'
PG OBJECTS STATE UP ACTING
1.3 0 active+clean [1,2,5]p1 [1,2,5]p1
...
1.f 0 active+clean [3,1,4]p3 [3,1,4]p3
* NOTE: and soon afterwards
```
{% endtab %}
{% endtabs %}

### **存储删除**

删除存储池命令存在数据丢失的风险，Ceph于是默认禁止此类操作。&#x20;

管理员需要在ceph.conf配置文件中启用支持删除动作&#x20;

`ceph osd pool rm`` `<mark style="color:purple;">`存储池名`</mark> <mark style="color:purple;">`存储池名`</mark>` ``--yes-i-really-really-mean-it`&#x20;

注意： 存储池名称必须出现两遍，后面的 参数代表是强制

<mark style="color:purple;">删除实践</mark>

{% code title="默认情况下删除存储池" %}
```bash
$ ceph osd pool rm mypool mypool --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
```
{% endcode %}

```bash
主节点上修改ceph.conf文件，增加两行配置，让其运行删除pool
$ tail -n2 ceph.conf
[mon]
mon allow pool delete = true
​
同步ceph.conf文件到所有的ceph节点上
$ ceph-deploy --overwrite-conf config push admin mon01 mon02 mon03
​
重启所有的mon节点上的ceph-mon服务
$ for i in mon0{1..3}; do ssh $i "sudo systemctl restart ceph-mon.target"; done
```

```bash
将刚才添加的文件对象从存储池里面移除
$ ceph osd pool rm mypool mypool --yes-i-really-really-mean-it
pool 'mypool' removed
​
确认效果
$ ceph osd pool stats
there are no pools!
$ ceph osd pool ls
```

**小结**

