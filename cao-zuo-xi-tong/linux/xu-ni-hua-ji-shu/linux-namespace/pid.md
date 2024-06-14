# PID

* 保障进程隔离，每个容器都以 PID=1 的 init 进程来启动,其他进程的 PID 会依次递增。
* PID Namespace 使用了的参数 CLONE\_NEWPID。
* 父 NameSpace 可以拿到全部子 NameSpace 的状态，但每个子 NameSpace 之间是互相隔离的。
