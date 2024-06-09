# 停止容器

```
# ctr tasks --help

使用kill命令停止容器中运行的进程，既为停止容器
# ctr tasks kill nginx2

查看容器停止后状态，STATUS为STOPPED
# ctr tasks ls
TASK      PID     STATUS
nginx1    3395    RUNNING
nginx2    4061    STOPPED
```
