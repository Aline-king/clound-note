# mount

保障每个容器都有独立的目录挂载路径。

Mount Namespace 使用了的参数 CLONE\_NEWNS，用来隔离各个进程看到的挂载点视图。
