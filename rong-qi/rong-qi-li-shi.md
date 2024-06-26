# 容器历史

<img src="../.gitbook/assets/file.excalidraw.svg" alt="" class="gitbook-drawing">

{% tabs %}
{% tab title="1979年" %}
chroot

* 容器技术的概念可以追溯到1979年的UNIX chroot。
* 它是一套“UNIX操作系统”系统，旨在将其root目录及其它子目录变更至文件系统内的新位置，且只接受特定进程的访问。
* 这项功能的设计目的在于为每个进程提供一套隔离化磁盘空间。
* 1982年其被添加至BSD当中。
{% endtab %}

{% tab title="2000年 " %}
FreeBSD Jails

* FreeBSD Jails是由Derrick T. Woolworth于2000年在FreeBSD研发协会中构建而成的早期容器技术之一。
* 这是一套“操作系统”系统，与chroot的定位类似，不过其中包含有其它进程沙箱机制以对文件系统、用户及网络等资源进行隔离。
* 通过这种方式，它能够为每个Jail、定制化软件安装包乃至配置方案等提供一个对应的IP地址。
{% endtab %}

{% tab title="2001年" %}
Linux VServer

* Linux VServer属于另一种jail机制，其能够被用于保护计算机系统之上各分区资源的安全(包括文件系统、CPU时间、网络地址以及内存等)。
* 每个分区被称为一套安全背景(security context)，而其中的虚拟化系统则被称为一套虚拟私有服务器。
{% endtab %}

{% tab title="2004年" %}
Solaris容器

* Solaris容器诞生之时面向x86与SPARC系统架构，其最初亮相于2004年2月的Solaris 10 Build 51 beta当中，随后于2005年正式登陆Solaris 10的完整版本。
* Solaris容器相当于将系统资源控制与由分区提供的边界加以结合。各分区立足于单一操作系统实例之内以完全隔离的虚拟服务器形式运行。
{% endtab %}

{% tab title="2005年" %}
### OpenVZ

* OpenVZ与Solaris容器非常相似，且使用安装有补丁的Linux内核以实现虚拟化、隔离能力、资源管理以及检查点交付。
* 每套OpenVZ容器拥有一套隔离化文件系统、用户与用户群组、一套进程树、网络、设备以及IPC对象。
{% endtab %}

{% tab title="2006年" %}
### Process容器

* Process容器于2006年由谷歌公司推出，旨在对一整套进程集合中的资源使用量(包括CPU、内存、磁盘I/O以及网络等等)加以限制、分配与隔离。
* 此后其被更名为Control Groups(即控制组)，从而避免其中的“容器”字眼与Linux内核2.6.24中的另一术语出现冲突。这表明了谷歌公司率先重视容器技术的敏锐眼光以及为其做出的突出贡献。
{% endtab %}

{% tab title="Untitled" %}

{% endtab %}
{% endtabs %}

### 1.7 2007年 — Control Groups

Control Groups也就是谷歌实现的cgroups，其于2007年被添加至Linux内核当中。

### 1.8 2008年 — LXC

* LXC指代的是Linux Containers
* 是第一套完整的Linux容器管理实现方案。
* 其功能通过cgroups以及Linux namespaces实现。
* LXC通过liblxc库进行交付，并提供可与Python3、Python2、Lua、Go、Ruby以及Haskell等语言相对接的API。
* 相较于其它容器技术，LXC能够在无需任何额外补丁的前提下运行在原版Linux内核之上。

### 1.9 2011年 — Warden

* Warden由CloudFoundry公司于2011年所建立，其利用LXC作为初始阶段，随后又将其替换为自家实现方案。
* 与LXC不同，Warden并不会与Linux紧密耦合。相反，其能够运行在任意能够提供多种隔离环境方式的操作系统之上。Warden以后台进程方式运行并提供API以实现容器管理。

### 1.10 2013年 — LMCTFY

* Lmctfy代表的是“Let Me Contain That For You(帮你实现容器化)”。它其实属于谷歌容器技术堆栈的开源版本，负责提供Linux应用程序容器。谷歌公司在该项目的起步阶段宣称其能够提供值得信赖的性能表现、高资源利用率、共享资源机制、充裕的发展空间以及趋近于零的额外资源消耗。
* 2013年10月lmctfy的首个版本正式推出，谷歌公司在2015年决定将lmctfy的核心概念与抽象机制转化为libcontainer。在失去了主干之后，如今lmctfy已经失去一切积极的发展势头。

　　Libcontainer项目最初由Docker公司建立，如今已经被归入开放容器基金会的管辖范畴。

### 1.11 2013年-Docker

* 在2013年Docker刚发布的时候，它是一款基于LXC的开源容器管理引擎。
* 把LXC复杂的容器创建与使用方式简化为Docker自己的一套命令体系。
* 随着Docker的不断发展，它开始有了更为远大的目标，那就是反向定义容器的实现标准，将底层实现都抽象化到Libcontainer的接口。这就意味着，底层容器的实现方式变成了一种可变的方案，无论是使用namespace、cgroups技术抑或是使用systemd等其他方案，只要实现了Libcontainer定义的一组接口，Docker都可以运行。这也为Docker实现全面的跨平台带来了可能。

/\
