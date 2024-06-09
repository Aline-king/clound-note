---
description: containerd中namespace的作用为:隔离运行的容器，可以实现运行多个容器。
---

# namespace管理

1111

```
列出已有namespace
# ctr namespace ls
NAME    LABELS
default
k8s.io
```

```
创建namespace
# ctr namespace create kubemsb
​
[root@localhost ~]# ctr namespace ls
NAME    LABELS
default
k8s.io
kubemsb 此命名空间为新添加的
```

```
删除namespace
# ctr namespace rm kubemsb
kubemsb
​
再次查看是否删除
[root@localhost ~]# ctr namespace ls
NAME    LABELS
default
k8s.io
```

```
查看指定namespace中是否有用户进程在运行
# ctr -n kubemsb tasks ls
TASK    PID    STATUS
```

```
在指定namespace中下载容器镜像
# ctr -n kubemsb images pull docker.io/library/nginx:latest
```

```
在指定namespace中创建静态容器
# ctr -n kubemsb container create docker.io/library/nginx:latest nginxapp
```

```
查看在指定namespace中创建的容器
# ctr -n kubemsb container ls
CONTAINER    IMAGE                             RUNTIME
nginxapp     docker.io/library/nginx:latest    io.containerd.runc.v2
```

\
