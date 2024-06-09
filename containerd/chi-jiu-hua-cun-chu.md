# 持久化存储

```bash
# ctr container create docker.io/library/busybox:latest busybox3 --mount type=bind,src=/tmp,dst=/hostdir,options=rbind:rw
```

```
说明：
创建一个静态容器，实现宿主机目录与容器挂载
src=/tmp 为宿主机目录
dst=/hostdir 为容器中目录
```

```
运行用户进程
# ctr tasks start -d busybox3 bash
```

```
进入容器，查看是否挂载成功
# ctr tasks exec --exec-id $RANDOM -t busybox3 sh
​
/ # ls /hostdir
VMwareDnD
systemd-private-cf1fe70805214c80867e7eb62dff5be7-bolt.service-MWV1Ju
systemd-private-cf1fe70805214c80867e7eb62dff5be7-chronyd.service-6B6j8p
systemd-private-cf1fe70805214c80867e7eb62dff5be7-colord.service-6fI31A
systemd-private-cf1fe70805214c80867e7eb62dff5be7-cups.service-tuK4zI
systemd-private-cf1fe70805214c80867e7eb62dff5be7-rtkit-daemon.service-vhP67o
tracker-extract-files.0
vmware-root_703-3988031936
vmware-root_704-2990744159
vmware-root_713-4290166671
​
​
向容器中挂载目录中添加文件
/ # echo "hello world" > /hostdir/test.txt
​
退出容器
/ # exit
​
在宿主机上查看被容器挂载的目录中是否添加了新的文件，已添加表明被容器挂载成功，并可以读写此目录中内容。
[root@localhost ~]# cat /tmp/test.txt
hello world
```

\
