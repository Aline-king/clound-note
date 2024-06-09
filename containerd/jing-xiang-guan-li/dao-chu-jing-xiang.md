# 导出镜像



> <mark style="color:yellow;">**说明**</mark> &#x20;
>
> \--all-platforms,导出所有平台镜像，本版本为1.6版本，1.4版本不需要添加此选项。

```
把容器镜像导出
# ctr i export --all-platforms nginx.img docker.io/library/nginx:alpine
```

```
查看已导出容器镜像
# ls
nginx.img

# ls -lh
总用量 196M

-rw-r--r--  1 root root  73M 2月  18 14:48 nginx.img
```
