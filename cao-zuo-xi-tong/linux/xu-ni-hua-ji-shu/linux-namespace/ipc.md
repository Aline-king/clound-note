# IPC

保障每个容器进程 IPC 通信隔离。

IPC Namespace 使用了的参数 CLONE\_NEWIPC，用来隔离 系统IPC 和 POSIX message队列

在相同 IPC 命名空间的容器进程之间才可以共享内存、信号量、消息队列通信。
