# 使用

<details>

<summary>设置CPU 周期限制为总量的1/10</summary>

echo 100000 > /sys/fs/cgroup/cpu/my\_cpu/cpu.cfs\_period\_us

echo 10000 > /sys/fs/cgroup/cpu/my\_cpu/cpu.cfs\_quota\_us

编写一个go的cpu密集型代码

```Go
cat <<EOF>cpu_use.go
package main
func cpuUse() {
    n := 1024 * 1024 * 1024 * 10
    for i := 0; i < n; i++ {
        i++
    }
}
func main() {
    cpuUse()
}
EOF
```

编译

```Go
go build cpu_use.go
time ./cpu_use
real    0m2.130s // 耗时2s多
user    0m2.131s
sys     0m0.004s
```

使用 cgexec 加入my\_cpu 的cgroup运行

```bash
time cgexec -g cpu:my_cpu ./cpu_use
real    0m21.538s  // 耗时已经上涨到了21秒
user    0m2.158s   // 用户态运行就2秒
sys     0m0.010s
// 说明其他时间一直被限制
```

同时在运行过程中可以看到 /sys/fs/cgroup/cpu/my\_cpu/tasks 中被写入 cpu\_use的线程id

```Go
ps -efT |grep cpu_use
root     13772 13772 26216  8 19:17 pts/1    00:00:00 ./cpu_use
root     13772 13773 26216  0 19:17 pts/1    00:00:00 ./cpu_use
root     13772 13774 26216  0 19:17 pts/1    00:00:00 ./cpu_use
root     13772 13775 26216  0 19:17 pts/1    00:00:00 ./cpu_use
root     13772 13776 26216  0 19:17 pts/1    00:00:00 ./cpu_use
root     13772 13777 26216  0 19:17 pts/1    00:00:00 ./cpu_use
root     13830 13830 19237  0 19:18 pts/0    00:00:00 grep --color=auto cpu_use

 cat /sys/fs/cgroup/cpu/my_cpu/tasks 
13772
13773
13774
13775
13776
13777
```

</details>

### 限制进程可用的内存

1. 准备subsystem 目录
   1. mkdir -pv /sys/fs/cgroup/memory/my\_mem
   2. 下面的设置把进程的可用内存限制在最大 300M，并且不使用 swap：
      1. echo 314572800 > /sys/fs/cgroup/memory/my\_mem/memory.limit\_in\_bytes
      2. echo 0 > /sys/fs/cgroup/memory/my\_mem/memory.swappiness
2. 准备go代码
   1. 共申请5次内存，每次100MB
      1. ```Go
         cat <<EOF > mem_use.go
         package main
         import (
             "fmt"
             "unsafe"
         )
         func memUse() {
             for i := 0; i < 5; i++ {
                 // arr代表100MB内存大小的arr，因为一个int64是8byte，128就是1024 
                 arr := [128 * 1024 * 1024]int64{}
                 fmt.Printf("[index:%v][size:%v]\n", i+1, unsafe.Sizeof(arr))
             }
         }
         func main() {
             memUse()
         }
         EOF
         ```
      2. 正常编译运行
      3. ```Go
         go build mem_use.go 
         ./mem_use 
         [index:1][size:1073741824]
         [index:2][size:1073741824]
         [index:3][size:1073741824]
         [index:4][size:1073741824]
         [index:5][size:1073741824]
         ```
      4. 添加memory cgroup 限制 后运行
      5. ```Go
         cgexec -g memory:my_mem ./mem_use
         [index:1][size:1073741824]
         [index:2][size:1073741824]
         ```
      6. Killed
         1. 程序中途被kill了，因为申请的总量已经超过cgroup的限制了
         2. 同时观察 /var/log/messages 中oom的信息
         3. ```Go
            Sep 26 19:38:54 k8s-master01 kernel: mem_use invoked oom-killer: gfp_mask=0xd0, order=0, oom_score_adj=0
            Sep 26 19:38:54 k8s-master01 kernel: mem_use cpuset=/ mems_allowed=0
            Sep 26 19:38:54 k8s-master01 kernel: CPU: 2 PID: 5454 Comm: mem_use Kdump: loaded Tainted: G             L ------------ T 3.10.0-957.1.3.el7.x86_64 #1
            Sep 26 19:38:54 k8s-master01 kernel: Hardware name: Red Hat KVM, BIOS 1.10.2-3.el7_4.1 04/01/2014
            Sep 26 19:38:54 k8s-master01 kernel: Call Trace:
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa1761e41>] dump_stack+0x19/0x1b
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa175c86a>] dump_header+0x90/0x229
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa11ba036>] ? find_lock_task_mm+0x56/0xc0
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa11ba4e4>] oom_kill_process+0x254/0x3d0
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa1235186>] mem_cgroup_oom_synchronize+0x546/0x570
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa1234600>] ? mem_cgroup_charge_common+0xc0/0xc0
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa11bad74>] pagefault_out_of_memory+0x14/0x90
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa175ad72>] mm_fault_error+0x6a/0x157
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa176f7a8>] __do_page_fault+0x3c8/0x500
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa176f9c6>] trace_do_page_fault+0x56/0x150
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa176ef42>] do_async_page_fault+0x22/0xf0
            Sep 26 19:38:54 k8s-master01 kernel: [<ffffffffa176b788>] async_page_fault+0x28/0x30
            Sep 26 19:38:54 k8s-master01 kernel: Task in /my_mem killed as a result of limit of /my_mem
            Sep 26 19:38:54 k8s-master01 kernel: memory: usage 307200kB, limit 307200kB, failcnt 567
            Sep 26 19:38:54 k8s-master01 kernel: memory+swap: usage 307200kB, limit 9007199254740988kB, failcnt 0
            Sep 26 19:38:54 k8s-master01 kernel: kmem: usage 0kB, limit 9007199254740988kB, failcnt 0
            Sep 26 19:38:54 k8s-master01 kernel: Memory cgroup stats for /my_mem: cache:0KB rss:307200KB rss_huge:0KB mapped_file:0KB swap:0KB inactive_anon:0KB active_anon:307192KB inactive_file:0KB active_file:0KB unevictable:0KB
            Sep 26 19:38:54 k8s-master01 kernel: [ pid ]   uid  tgid total_vm      rss nr_ptes swapents oom_score_adj name
            Sep 26 19:38:54 k8s-master01 kernel: [ 5450]     0  5450   717445    76878     165        0             0 mem_use
            Sep 26 19:38:54 k8s-master01 kernel: Memory cgroup out of memory: Kill process 5456 (mem_use) score 973 or sacrifice child
            Sep 26 19:38:54 k8s-master01 kernel: Killed process 5450 (mem_use) total-vm:2869780kB, anon-rss:306836kB, file-rss:676kB, shmem-rss:0kB
            ```
