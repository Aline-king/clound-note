# 删除容器

```bash
# ctr tasks delete nginx2
必须先停止tasks或先删除task，再删除容器

查看静态容器，确认其还存在于系统中
# ctr container ls
CONTAINER    IMAGE                             RUNTIME
nginx2       docker.io/library/nginx:alpine    io.containerd.runc.v2

删除容器
# ctr container delete nginx2
```
