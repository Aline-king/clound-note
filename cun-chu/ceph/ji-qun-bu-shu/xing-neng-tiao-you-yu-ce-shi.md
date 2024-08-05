# 性能调优与测试

## 性能调优

性能优化没有一个标准的定义，如果用大白话来说的话，他就是在现有主机资源的前提下，发挥业务最大的处理能力。

理解为：通过空间(资源消耗)优化换取时间(业务处理)效益 它主要体现在几个方面：

改善业务逻辑处理的速度、提升业务数据吞吐量、优化主机资源的消耗量。

性能优化没有一个基准值，我们只能通过自我或者普遍的场景工作经验，设定一个阶段性的目标，然后通过物理手段、配置手段、代码手段等方式提高主机资源的业务处理能力。

### 通用处理措施

<mark style="color:purple;">**基础设施**</mark>

1. 合适设备 - 多云环境、新旧服务器结合、合适网络等&#x20;
2. 研发环境 - 应用架构、研发平台、代码规范等

<mark style="color:purple;">**应用软件**</mark>

1. 多级缓存 - 浏览器缓存、缓存服务器、服务器缓存、应用缓存、代码缓存、数据缓存等。&#x20;
2. 数据压缩 - 数据构建压缩、数据传输压缩、缓存数据压缩、数据对象压缩、清理无效数据等。
3. 数据预热 - 缓冲数据、预取数据、预置主机、数据同步等。

<mark style="color:purple;">**业务处理**</mark>

1. 削峰填谷 - 延时加载、分批发布、限流控制、异步处理、超时降级等&#x20;
2. 任务批处理 - 数据打包传输、数据批量传输、延迟数据处理等

### **常见策略**

{% tabs %}
{% tab title="基础设施" %}
Ceph集群的各个服务守护进程在 Linux 服务器上运行，所以适当调优操作系统能够对ceph存储集群的性能产生积极的影响。它主要涉及到以下几个方面：

<mark style="color:purple;">**选择合适的CPU和内存。**</mark>

1. 不同角色具有不同的 CPU 需求，MDS(4)>OSD(2)>MON(1)，纠删代码功能需要更多 CPU。&#x20;
2. 不同角色具有不同的 内存 需求，MON>OSD

<mark style="color:purple;">**选择合适的磁盘容量**</mark>

1. 合理评估阶段性业务量，计算节点数量及各个节点的磁盘容量需求。&#x20;
2. 选择合适的 SAS、SATA、SSD，实现分级存储的能力。&#x20;
3. OS系统、数据、日志使用独立的SSD硬盘，使整体吞吐量最大化

<mark style="color:purple;">**选择合适的网络设备**</mark>

1. Ceph集群对于网络数据传输能力要求高，需要支持<mark style="color:red;">**巨型帧**</mark>(9000字节，默认1500字节)功能
2. &#x20;Ceph集群数据传输量很大，需要所有设备的 MTU 设置必须一致&#x20;
3. 如果可以的话，网络带宽一定要大，最好以3倍需求量的方式采购

<mark style="color:purple;">**配置合适的文件系统**</mark>

1. 推荐使用  <mark style="color:orange;">**vfs**</mark> 文件系统，ceph借助于xfs扩展属性在创建和挂载文件系统场景中提高索引数据缓存
2. osd\_mkfs\_options\_xfs、osd\_mount\_options\_xfs

<mark style="color:purple;">**搭建内部时间服务器**</mark>

ceph集群对于时间要求比较高，为了保证数据的通信一致性，最好采用内部时间服务器

<mark style="color:purple;">**合理采用多云环境**</mark>

将合适的业务负载转移到公有云环境。
{% endtab %}

{% tab title="应用软件" %}
Ceph集群的很多功能都是以各种<mark style="color:orange;">**服务配置**</mark>的方式来进行实现的，所以我们有必要结合不同的配置属性，实现ceph集群环境级别的性能优化：

<mark style="color:purple;">**数据访问**</mark>

采用public网络和cluster网络机制，实现数据操作网络的可靠性，避免外部网络攻击的安全性。

配置属性：public\_network、cluster\_network、max\_open\_file等

<mark style="color:purple;">**数据存储**</mark>

设定合理的存储池的PG 数量，集群PGs总数 = (OSD总数 \* 100 / 最大副本数)

更多数据存储规划：https://docs.ceph.com/en/latest/rados/operations/placement-groups/#choosing-the-number-of-placement-groups

<mark style="color:purple;">**多级缓存**</mark>

根据实际情况开启RDBCache(rbd\_cache\_xxx)、RGWCache(rgw\_cache\_xxx)、OSDCache、MDSCache

建立合理的SSD缓存pool，设定合理的cache age、target size 等参数

理清业务场景，根据缓存模式(write-back、readproxy、readonly等)，启用cache-tiering机制。
{% endtab %}

{% tab title="业务处理" %}
ceph集群在大规模数据处理的时候，可以通过数据批处理能力来实现数据的高效传输。

<mark style="color:purple;">**数据批处理**</mark>

**filestore\_max|min\_sync\_interval**：从日志到数据盘最大|小同步间隔(seconds) **filestore\_queue\_max\_ops|bytes**：数据盘单次最大接受的操作数|字节数 **filestore\_queue\_committing\_max\_ops|bytes**：数据盘单次最大处理的操作数|字节数 **journal\_max\_write\_ops|bytes**：日志一次性写入的最大操作数|字节数(bytes) **journal\_queue\_max\_ops|bytes**：日志一次性查询的最大操作数|字节数(bytes)

osd\_max\_write\_size：OSD一次可写入的最大值(MB)&#x20;

osd\_recovery\_max\_active：同一时间内活跃的恢复请求数
{% endtab %}
{% endtabs %}

## 性能测试

### 基准测试

{% tabs %}
{% tab title="磁盘测试" %}
```
准备工作-清理缓存
# echo 3 > /proc/sys/vm/drop_caches^C
# cat /proc/sys/vm/drop_caches
0
# echo 3 > /proc/sys/vm/drop_caches
​
查看磁盘
# ls /var/lib/ceph/osd/
ceph-4  ceph-5
```

```
OSD 磁盘写性能
# dd if=/dev/zero of=/var/lib/ceph/osd/ceph-4/write_test bs=1M count=100
...
104857600字节(105 MB)已复制，0.4009 秒，262 MB/秒
​
OSD 磁盘读性能
# dd if=/var/lib/ceph/osd/ceph-4/write_test of=/dev/null bs=1M count=100
...
104857600字节(105 MB)已复制，0.0233439 秒，4.5 GB/秒
​
清理环境
# rm -f /var/lib/ceph/osd/ceph-4/write_test
```

参考资料：https://www.thomas-krenn.com/en/wiki/Linux\_I/O\_Performance\_Tests\_using\_dd
{% endtab %}

{% tab title="网络测试" %}
在多台主机上准备网络基准测试工具&#x20;

yum install iperf -y&#x20;

<mark style="color:purple;">**服务端命令参数**</mark>&#x20;

\-s 以server模式启动&#x20;

\-p 指定服务器端使用的端口或客户端所连接的端口&#x20;

<mark style="color:purple;">**客户端命令参数**</mark>&#x20;

\-d 同时进行双向传输测试&#x20;

\-n 指定传输的字节数

```
节点1执行命令
[root@admin ~]# iperf -s -p 6900
```

```
客户端执行命令
[root@mon03 ~]# iperf -c admin -p 6900
------------------------------------------------------------
Client connecting to admin, TCP port 6900
TCP window size:  552 KByte (default)
------------------------------------------------------------
[  3] local 10.0.0.15 port 48376 connected with 10.0.0.12 port 6900
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  2.58 GBytes  2.21 Gbits/sec
```

无论是磁盘测试还是网络测试，都需要多次测试后，获取平均值效果
{% endtab %}
{% endtabs %}

### rados测试



{% tabs %}
{% tab title="bench性能测试工具" %}
它是Ceph 自带的 rados bench 测试工具，

命令格式如下： rados bench -p <存储池> <秒数> <操作模式> -b <块大小> -t 并行数 --no-cleanup

```
操作模式：包括三种，分别是
    write：写，
    seq：顺序读；
    rand：随机读 
块大小：默认为4M 
并行数: 读/写并行数，默认为 16 
--no-cleanup 表示测试完成后不删除测试用数据。
    - 在做读测试之前，需要使用该参数来运行一遍写测试来产生测试数据， 
    - 在测试结束后运行 rados -p <存储池> cleanup 清理测试数据。
```

{% hint style="warning" %}
注意

曾经的bench-write已经被整合到bench中了
{% endhint %}

创建测试存储池

```

[root@admin ~]# ceph osd pool create pool-test 32 32
pool 'pool-test' created
```

写测试

```
[root@admin ~]# rados bench -p pool-test 10 write --no-cleanup
hints = 1
Maintaining 16 concurrent writes of 4194304 bytes to objects of size 4194304 for up to 10 seconds or 0 objects
Object prefix: benchmark_data_admin_5683
  sec Cur ops   started  finished  avg MB/s  cur MB/s last lat(s)  avg lat(s)
    0       0         0         0         0         0           -           0
    ...
   10      16       147       131   52.3689        64     1.18095     1.16048
Total time run:         10.9577
...
Min latency(s):         0.202
```

顺序读测试

```
[root@admin ~]# rados bench -p pool-test 10 seq
hints = 1
  sec Cur ops   started  finished  avg MB/s  cur MB/s last lat(s)  avg lat(s)
    0       0         0         0         0         0           -           0
    ...
   10      15       139       124   45.0608         8     9.09797     0.93913
Total time run:       11.5093
...
Min latency(s):       0.0240264
```

随机读测试

```
[root@admin ~]# rados bench -p pool-test 10 rand
hints = 1
  sec Cur ops   started  finished  avg MB/s  cur MB/s last lat(s)  avg lat(s)
    0       0         0         0         0         0           -           0
    ...
   10      16       589       573   229.081       244     0.20209    0.272096
Total time run:       10.2428
...
Min latency(s):       0.0225745
```

清理环境

<pre><code><strong>[root@admin ~]# rados -p pool-test cleanup
</strong>Removed 148 objects
</code></pre>


{% endtab %}

{% tab title="load-gen性能测试工具" %}
Ceph 自带的 rados 性能测试工具，可在集群内产生混合类型的负载，相较于bench，load-gen只做吞吐量测试。命令格式如下：

`rados -p 存储池 load-gen`

```
--num-objects     	# 初始生成测试用的对象数
--min-object-size 	# 最小对象大小
--max-object-size 	# 最大对象大小
--max-ops         	# 最大操作数目
--min-op-len      	# 最小操作长度，相当于iodepth
--max-op-len      	# 最大操作长度
--read-percent    	# 读操作的百分比
--target-throughput     # 单次提交IO的累计吞吐量上线，单位 MB/s
--run-length      	# 运行时长，单位秒
--percent	        # 读写混合中，读操作占用比例
--max_backlog		# 单次提交IO的吞吐量上限
```

iodepth就是io队列深度(io队列里面允许出现几个命令等待操作)，

作用：**控制硬盘操作的过程中的负载情况，避免系统一直发送指令，出现磁盘受不了导致系统崩溃的现象**

在pool-test存储池中，初始对象50个，单次io的最大数量16，顺序写入4M的数据块,测试时间10秒。

```
[root@mon03 ~]# rados -p pool-test load-gen --num-objects 50 --min-object-size 4M --max-object-size 4M --max-object-size 4M --max-ops 16 --min-op-len 4M  --max-op-len 4M --percent 5 --target-throughput 40 --run-length 10
run length 10 seconds
preparing 50 objects
load-gen will run 10 seconds
    1: throughput=0MB/sec pending data=0
READ : oid=obj-AF5uiLtTtxSNSNR off=0 len=4194304
op 0 completed, throughput=3.86MB/sec
    ...
    9: throughput=0.442MB/sec pending data=0
waiting for all operations to complete
cleaning up objects
```
{% endtab %}
{% endtabs %}
