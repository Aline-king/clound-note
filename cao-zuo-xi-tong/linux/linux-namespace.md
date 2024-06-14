# Linux Namespace

* 很多编程语言都包含了命名空间的概念，我们可以认为命名空间是一种封装，封装本身实际上实现了代码的隔离
* 在操作系统中命名空间命名空间提供的是系统资源的隔离，其中系统资源包括了：进程、网络、文件系统......

linux系统实现命名空间主要目的之一就是为了实现轻量级虚拟化服务，达到隔离的目的

<table data-header-hidden><thead><tr><th width="167"></th><th width="172"></th><th width="224"></th><th></th></tr></thead><tbody><tr><td>命名空间</td><td>描述</td><td>作用</td><td>备注</td></tr><tr><td>进程命名空间</td><td>隔离进程ID</td><td>Linux通过命名空间管理进程号，同一个进程，在不同的命名空间进程号不同</td><td>进程命名空间是一个父子结构，子空间对于父空间可见</td></tr><tr><td>网络命名空间</td><td>隔离网络设备、协议栈、端口等</td><td>通过网络命名空间，实现网络隔离</td><td>docker采用虚拟网络设备，将不同命名空间的网络设备连接到一起</td></tr><tr><td>IPC命名空间</td><td>隔离进程间通信</td><td>进程间交互方法</td><td>PID命名空间和IPC命名空间可以组合起来用，同一个IPC名字空间内的进程可以彼此看见，允许进行交互，不同空间进程无法交互</td></tr><tr><td>挂载命名空间</td><td>隔离挂载点</td><td>隔离文件目录</td><td>进程运行时可以将挂载点与系统分离，使用这个功能时，我们可以达到 chroot 的功能，而在安全性方面比 chroot 更高</td></tr><tr><td>UTS命名空间</td><td>隔离Hostname和NIS域名</td><td>让容器拥有独立的主机名和域名，从而让容器看起来像个独立的主机</td><td>目的是独立出主机名和网络信息服务（NIS）</td></tr><tr><td>用户命名空间</td><td>隔离用户和group ID</td><td>每个容器内上的用户跟宿主主机上不在一个命名空间</td><td>同进程 ID 一样，用户 ID 和组 ID 在命名空间内外是不一样的，并且在不同命名空间内可以存在相同的 ID</td></tr></tbody></table>

## 系统调用

*   #### clone：

    创建新进程，根据传入上面的不同 NameSpace 类型，来创建不同的 NameSpace 进行隔离，

    对应的子进程也会被包含到这些 Namespace 中。
*   #### setns：

    将进程加入到 Namespace 中
*   #### unshare：

    将进程移出某个指定类型的 Namespace，并加入到新创建的 NameSpace 中

    容器中 NameSpace 也是通过 unshare 系统调用创建的。
