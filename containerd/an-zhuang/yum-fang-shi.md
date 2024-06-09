# YUM方式

<details>

<summary>1：获取YUM源</summary>

获取阿里云YUM源

<pre class="language-bash"><code class="lang-bash"><strong>wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
</strong></code></pre>

查看YUM源中Containerd软件

```bash
yum list | grep containerd
containerd.io.x86_64                        1.4.12-3.1.el7             docker-ce-stable
```



</details>

<details>

<summary>2：使用yum命令安装</summary>

安装Containerd.io软件，即可安装Containerd

```bash
yum -y install containerd.io
```

</details>

<details>

<summary>3：验证安装及启动服务</summary>

使用rpm -qa命令查看是否安装

```bash
rpm -qa | grep containerd
# containerd.io-1.4.12-3.1.el7.x86_64
```

设置containerd服务启动及开机自启动

```bash
systemctl enable containerd
systemctl start containerd
```

查看containerd服务启动状态



</details>



```bash
systemctl status containerd
● containerd.service - containerd container runtime
   Loaded: loaded (/usr/lib/systemd/system/containerd.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2022-02-18 11:38:30 CST; 9s ago 此行第二列及第三列表示其正在运行状态
     Docs: https://containerd.io
  Process: 59437 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
 Main PID: 59439 (containerd)
    Tasks: 7
   Memory: 19.5M
   CGroup: /system.slice/containerd.service
           └─59439 /usr/bin/containerd
           ......
```

## 验证可用性

```
安装Containerd时ctr命令亦可使用，ctr命令主要用于管理容器及容器镜像等。
使用ctr命令查看Containerd客户端及服务端相关信息。
# ctr version
Client:
  Version:  1.4.12
  Revision: 7b11cfaabd73bb80907dd23182b9347b4245eb5d
  Go version: go1.16.10
​
Server:
  Version:  1.4.12
  Revision: 7b11cfaabd73bb80907dd23182b9347b4245eb5d
  UUID: 3c4b142d-d91d-44a5-aae2-9673785d4b2c
```

\
