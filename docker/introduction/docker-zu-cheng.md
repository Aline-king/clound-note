# Docker组成

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* docker-cli：这是一个命令行工具，它是用来完成 `docker pull`, `build`, `run`, `exec` 等命令进行交互。
* containerd：这是一个管理和运行容器的守护进程。它推送和拉动镜像，管理存储和网络，并监督容器的运行。
* runc：这是低级别的容器运行时间（实际创建和运行容器的东西）。它包括 libcontainer，一个用于创建容器的基于 Go 的本地实现。
