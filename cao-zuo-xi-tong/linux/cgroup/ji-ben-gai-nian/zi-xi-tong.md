# 子系统



1. cpu: 限制进程的 cpu 使用率。
   1. cpu子系统：限制对CPU的访问，每个参数独立存在于cgroups虚拟文件系统的伪文件中
   2.
      * **cpu.shares**: cgroup对时间的分配。比如cgroup A设置的是1，cgroup B设置的是2，那么B中的任务获取cpu的时间，是A中任务的2倍。
      * **cpu.cfs\_period\_us**: 完全公平调度器的调整时间配额的周期。
      * **cpu.cfs\_quota\_us**: 完全公平调度器的周期当中可以占用的时间。
      * **cpu.stat** 统计值
        * **nr\_periods** 进入周期的次数
        * **nr\_throttled** 运行时间被调整的次数
        * **throttled\_time** 用于调整的时间
2. cpuacct：子系统，可以统计 cgroups 中的进程的 cpu 使用报告。
   1. 子系统生成cgroup任务所使用的CPU资源报告，不做资源限制功能。
   2.
      * cpuacct.usage: 该cgroup中所有任务总共使用的CPU时间（ns纳秒）
      * cpuacct.stat: 该cgroup中所有任务总共使用的CPU时间，区分user和system时间。
      * cpuacct.usage\_percpu: 该cgroup中所有任务使用各个CPU核数的时间。
   3. 可以使用 cpuacct.usage的rate计算cpu利用率
   4.
      * 用cpuacct.usage的差值除以时间的差值 就是利用率了
   5. (cpuacct.usage\_t1 - cpuacct.usage\_t0) / (t1 - t0) \* 100
3. cpuset: 为cgroups中的进程分配单独的cpu节点或者内存节点。
   1. 适用于分配独立的CPU节点和Mem节点，比如将进程绑定在指定的CPU或者内存节点上运行，cpuset.cpus: 可以使用的cpu节点 cpuset.mems: 可以使用的mem节点cpuset.memory\_migrate: 内存节点改变是否要迁移？cpuset.cpu\_exclusive: 此cgroup里的任务是否独享cpu？cpuset.mem\_exclusive： 此cgroup里的任务是否独享mem节点？cpuset.mem\_hardwall: 限制内核内存分配的节点（mems是用户态的分配）cpuset.memory\_pressure: 计算换页的压力。cpuset.memory\_spread\_page: 将page cache分配到各个节点中，而不是当前内存节点。cpuset.memory\_spread\_slab: 将slab对象(inode和dentry)分散到节点中。cpuset.sched\_load\_balance: 打开cpu set中的cpu的负载均衡。cpuset.sched\_relax\_domain\_level: 迁移任务时的搜索范围cpuset.memory\_pressure\_enabled: 是否需要计算 memory\_pressure?
4. memory: 限制进程的memory使用量。
   1. memory子系统主要涉及内存一些的限制和操作，主要有以下参数：
   2.
      * memory.usage\_in\_bytes : 当前内存中的使用量
      * memory.memsw.usage\_in\_bytes : 当前内存和交换空间中的使用量
      * memory.limit\_in\_bytes : 设置or查看内存使用量
      * memory.memsw.limit\_in\_bytes : 设置or查看 内存加交换空间使用量
      * memory.failcnt : 查看内存使用量被限制的次数
      * memory.memsw.failcnt : - 查看内存和交换空间使用量被限制的次数
      * memory.max\_usage\_in\_bytes : 查看内存最大使用量
      * memory.memsw.max\_usage\_in\_bytes : 查看最大内存和交换空间使用量
      * memory.soft\_limit\_in\_bytes : 设置or查看内存的soft limit
      * memory.stat : 统计信息
      * memory.use\_hierarchy : 设置or查看层级统计的功能
      * memory.force\_empty : 触发强制page回收
      * memory.pressure\_level : 设置内存压力通知
      * memory.swappiness : 设置or查看vmscan swappiness 参数
      * memory.move\_charge\_at\_immigrate : 设置or查看 controls of moving charges?
      * memory.oom\_control : 设置or查看内存超限控制信息(OOM killer)
      * memory.numa\_stat : 每个numa节点的内存使用数量
      * memory.kmem.limit\_in\_bytes : 设置or查看 内核内存限制的硬限
      * memory.kmem.usage\_in\_bytes : 读取当前内核内存的分配
      * memory.kmem.failcnt : 读取当前内核内存分配受限的次数
      * memory.kmem.max\_usage\_in\_bytes : 读取最大内核内存使用量
      * memory.kmem.tcp.limit\_in\_bytes : 设置tcp 缓存内存的hard limit
      * memory.kmem.tcp.usage\_in\_bytes : 读取tcp 缓存内存的使用量
      * memory.kmem.tcp.failcnt : tcp 缓存内存分配的受限次数
      * memory.kmem.tcp.max\_usage\_in\_bytes : tcp 缓存内存的最大使用量
5. blkio: 限制进程的块设备io。
   1. 主要用于控制设备IO的访问，有两种限制方式：权重和上限，
   2.
      1. **权重**是给不同的应用一个权重值，按百分比使用IO资源，
         1. &#x20; 按权重分配IO资源：
         2. blkio.weight：填写 100-1000 的一个整数值，作为相对权重比率，作为通用的设备分配比。
         3. blkio.weight\_device： 针对特定设备的权重比，写入格式为 device\_types:node\_numbers weight，空格前的参数段指定设备，weight参数与blkio.weight相同并覆盖原有的通用分配比。
      2. 上限是控制应用读写速率的最大值。
         1. &#x20; 按上限限制读写速度：
         2. blkio.throttle.read\_bps\_device：按每秒读取块设备的数据量设定上限，格式device\_types:node\_numbers bytes\_per\_second。
         3. blkio.throttle.write\_bps\_device：按每秒写入块设备的数据量设定上限，格式device\_types:node\_numbers bytes\_per\_second。
         4. blkio.throttle.read\_iops\_device：按每秒读操作次数设定上限，格式device\_types:node\_numbers operations\_per\_second。
         5. blkio.throttle.write\_iops\_device：按每秒写操作次数设定上限，格式device\_types:node\_numbers operations\_per\_second
   3. 针对特定操作 (read, write, sync, 或 async) 设定读写速度上限
   4.
      * blkio.throttle.io\_serviced：针对特定操作按每秒操作次数设定上限，格式device\_types:node\_numbers operation operations\_per\_second
      * blkio.throttle.io\_service\_bytes：针对特定操作按每秒数据量设定上限，格式device\_types:node\_numbers operation bytes\_per\_second
6. devices: 控制进程能够访问某些设备。
7. net\_cls: 标记cgroups中进程的网络数据包，然后可以使用tc模块（traffic control）对数据包进行控制。
8. net\_prio: 限制进程网络流量的优先级。
9. huge\_tlb: 限制HugeTLB的使用。
10. freezer:挂起或者恢复cgroups中的进程。
11. ns: 控制cgroups中的进程使用不同的namespace。
