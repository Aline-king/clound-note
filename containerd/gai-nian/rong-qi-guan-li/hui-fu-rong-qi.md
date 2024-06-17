# 恢复容器

```
使用resume命令恢复容器
# ctr tasks resume nginx2

查看恢复后状态
# ctr tasks ls
TASK      PID     STATUS
nginx2    4061    RUNNING

在宿主机上访问容器中提供的服务
# curl http://192.168.10.164
nginx2
```
