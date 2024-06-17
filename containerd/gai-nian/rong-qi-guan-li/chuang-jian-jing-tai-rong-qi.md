# 创建静态容器

```
ctr c create docker.io/library/nginx:alpine nginx1
```

```
# ctr container ls
CONTAINER    IMAGE                             RUNTIME
nginx1       docker.io/library/nginx:alpine    io.containerd.runc.v2
```



```bash
查看容器详细信息
# ctr container info nginx1
```
