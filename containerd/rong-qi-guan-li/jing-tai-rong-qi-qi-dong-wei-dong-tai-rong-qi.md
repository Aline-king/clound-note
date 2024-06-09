# 静态容器启动为动态容器

```
复制containerd连接runC垫片工具至系统
# ls usr/local/bin/
containerd  containerd-shim  containerd-shim-runc-v1  containerd-shim-runc-v2  containerd-stress  crictl  critest  ctd-decoder  ctr
[root@localhost ~]# cp usr/local/bin/containerd-shim-runc-v2 /usr/bin/
```

> <mark style="color:yellow;">**说明：**</mark>
>
> &#x20;\-d 表示daemon或者后台的意思，否则会卡住终端

{% code title="启动task，即表时在容器中运行了进程，即为动态容器。" fullWidth="false" %}
```bash

# ctr task start -d nginx1
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
```
{% endcode %}

```
查看容器所在宿主机进程，是以宿主机进程的方式存在的。
# ctr task ls
TASK      PID     STATUS
nginx1    3395    RUNNING
```

```
查看容器的进程(都是物理机的进程)
# ctr task ps nginx1
PID     INFO
3395    -
3434    -
```

```
物理机查看到相应的进程
# ps -ef | grep 3395
root       3395   3375  0 19:16 ?        00:00:00 nginx: master process nginx -g daemon off;
101        3434   3395  0 19:16 ?        00:00:00 nginx: worker process
```
