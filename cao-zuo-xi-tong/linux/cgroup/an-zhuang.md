---
coverY: 0
---

# 安装

## 安装及开启

```bash
[root@localhost ~]# yum -y install libcgroup-tools
[root@localhost ~]# systemctl start cgconfig.service 	
[root@localhost ~]# systemctl enable cgconfig.service
```



## 准备cpu 子系统限制进程的cpu

```
mkdir /sys/fs/cgroup/cpu/my_cpu
```

{% hint style="info" %}
cgroups 的文件系统会在创建文件目录的时候自动创建这些配置文件
{% endhint %}

```bash
[root@k8s-master01 my_cpu]# ll /sys/fs/cgroup/cpu/my_cpu
total 0
-rw-r--r-- 1 root root 0 Sep 26 18:57 cgroup.clone_children
--w--w--w- 1 root root 0 Sep 26 18:57 cgroup.event_control
-rw-r--r-- 1 root root 0 Sep 26 18:57 cgroup.procs
-r--r--r-- 1 root root 0 Sep 26 18:57 cpuacct.stat
-rw-r--r-- 1 root root 0 Sep 26 18:57 cpuacct.usage
-r--r--r-- 1 root root 0 Sep 26 18:57 cpuacct.usage_percpu
-rw-r--r-- 1 root root 0 Sep 26 19:05 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Sep 26 19:05 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Sep 26 18:57 cpu.rt_period_us
-rw-r--r-- 1 root root 0 Sep 26 18:57 cpu.rt_runtime_us
-rw-r--r-- 1 root root 0 Sep 26 18:57 cpu.shares
-r--r--r-- 1 root root 0 Sep 26 18:57 cpu.stat
-rw-r--r-- 1 root root 0 Sep 26 18:57 notify_on_release
-rw-r--r-- 1 root root 0 Sep 26 19:09 tasks
```
