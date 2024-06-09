# 容器管理

## 获取静态容器

> <mark style="color:yellow;">**说明：**</mark>
>
> 使用`ctr container create` 命令创建容器后，容器并没有处于运行状态，其只是一个静态的容器。这个 container 对象只是包含了运行一个容器所需的资源及配置的数据结构，
>
> 例如： namespaces、rootfs 和容器的配置都已经初始化成功了，只是用户进程(本案例为nginx)还没有启动。需要使用`ctr tasks`命令才能获取一个动态容器。

```bash
ctr container --help
```

## 获取动态容器

> <mark style="color:yellow;">**说明：**</mark>
>
> 使用`ctr run`命令可以创建一个静态容器并使其运行。一步到位运行容器。

```bash
ctr run --help
```



