# 暂停容器

```
查看容器状态
# ctr tasks ls
TASK      PID     STATUS
nginx2    4061    RUNNING

暂停容器
# ctr tasks pause nginx2

再次查看容器状态，看到其状态为PAUSED，表示停止。
# ctr tasks ls
TASK      PID     STATUS
nginx2    4061    PAUSED

[root@localhost ~]# curl http://192.168.10.164
在宿主机访问，发现不可以访问到网站
```
