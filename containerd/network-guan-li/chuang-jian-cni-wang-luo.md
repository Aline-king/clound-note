# 创建CNI网络

> <mark style="color:yellow;">**说明：**</mark>
>
>

| [_containernetworking_/_cni_](https://github.com/containernetworking/cni)         | [CNI v1.0.1](https://github.com/containernetworking/cni/releases/tag/v1.0.1)             |
| --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| [_containernetworking_/_plugins_](https://github.com/containernetworking/plugins) | [CNI Plugins v1.0.1](https://github.com/containernetworking/plugins/releases/tag/v1.0.1) |

#### 7.1.1 获取CNI工具源码

![image-20220219095355845](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219095355845.png?lastModify=1717923303)

![image-20220219095427153](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219095427153.png?lastModify=1717923303)

![image-20220219095515772](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219095515772.png?lastModify=1717923303)

![image-20220219095615236](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219095615236.png?lastModify=1717923303)

```
使用wget下载cni工具源码包
# wget https://github.com/containernetworking/cni/archive/refs/tags/v1.0.1.tar.gz
```

```
查看已下载cni工具源码包
# ls
v1.0.1.tar.gz
​
解压已下载cni工具源码包
# tar xf v1.0.1.tar.gz
​
查看解压后已下载cni工具源码包
# ls
cni-1.0.1
​
重命名已下载cni工具源码包目录
# mv cni-1.0.1 cni
​
查看重新命名后目录
# ls
cni
​
查看cni工具目录中包含的文件
# ls cni
cnitool             CONTRIBUTING.md  DCO            go.mod  GOVERNANCE.md  LICENSE   MAINTAINERS  plugins    RELEASING.md  scripts  test.sh
CODE-OF-CONDUCT.md  CONVENTIONS.md   Documentation  go.sum  libcni         logo.png  pkg          README.md  ROADMAP.md    SPEC.md
```

#### 7.1.2 获取CNI Plugins（CNI插件）

![image-20220219095946940](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219095946940.png?lastModify=1717923303)

![image-20220219100008810](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219100008810.png?lastModify=1717923303)

![image-20220219100056059](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219100056059.png?lastModify=1717923303)

![image-20220219100303944](file:///G:/game/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7Containerd/05\_%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd/01\_%E7%AC%94%E8%AE%B0/%E8%BD%BB%E9%87%8F%E7%BA%A7%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%20Containerd.assets/image-20220219100303944.png?lastModify=1717923303)

```
使用wget下载cni插件工具源码包
# wget https://github.com/containernetworking/plugins/releases/download/v1.0.1/cni-plugins-linux-amd64-v1.0.1.tgz
```

```
查看已下载cni插件工具源码包
# ls
cni-plugins-linux-amd64-v1.0.1.tgz
cni
​
创建cni插件工具解压目录
# mkdir /home/cni-plugins
​
解压cni插件工具至上述创建的目录中
# tar xf cni-plugins-linux-amd64-v1.0.1.tgz -C /home/cni-plugins
​
查看解压后目录
# ls cni-plugins
bandwidth  bridge  dhcp  firewall  host-device  host-local  ipvlan  loopback  macvlan  portmap  ptp  sbr  static  tuning  vlan  vrf
```

#### 7.1.3 准备CNI网络配置文件

> 准备容器网络配置文件，用于为容器提供网关、IP地址等。

```
创建名为mynet的网络，其中包含名为cni0的网桥
# vim /etc/cni/net.d/10-mynet.conf
# cat /etc/cni/net.d/10-mynet.conf
{
  "cniVersion": "1.0.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.66.0.0/16",
    "routes": [
      { "dst": "0.0.0.0/0" }
   ]
  }
}
```

```
# vim /etc/cni/net.d/99-loopback.conf
# cat /etc/cni/net.d/99-loopback.conf
{
  "cniVerion": "1.0.0",
  "name": "lo",
  "type": "loopback"
}
```

#### 7.1.4 生成CNI网络

```
获取epel源
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
​
安装jq
# yum -y install jq
```

```
进入cni工具目录
# cd cni
[root@localhost cni]# ls
cnitool             CONTRIBUTING.md  DCO            go.mod  GOVERNANCE.md  LICENSE   MAINTAINERS  plugins    RELEASING.md  scripts  test.sh
CODE-OF-CONDUCT.md  CONVENTIONS.md   Documentation  go.sum  libcni         logo.png  pkg          README.md  ROADMAP.md    SPEC.md
​
​
必须在scripts目录中执行，需要依赖exec-plugins.sh文件，再次进入scripts目录
[root@localhost cni]# cd scripts/ 
​
查看执行脚本文件
[root@localhost scripts]# ls
docker-run.sh  exec-plugins.sh  priv-net-run.sh  release.sh
​
执行脚本文件，基于/etc/cni/net.d/目录中的*.conf配置文件生成容器网络
[root@localhost scripts]# CNI_PATH=/home/cni-plugins ./priv-net-run.sh echo "Hello World"
Hello World
```

```
在宿主机上查看是否生成容器网络名为cni0的网桥
# ip a s
......
5: cni0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 36:af:7a:4a:d6:12 brd ff:ff:ff:ff:ff:ff
    inet 10.66.0.1/16 brd 10.66.255.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::34af:7aff:fe4a:d612/64 scope link
       valid_lft forever preferred_lft forever
```

```
在宿主机上查看其路由表情况
# ip route
default via 192.168.10.2 dev ens33 proto dhcp metric 100
10.66.0.0/16 dev cni0 proto kernel scope link src 10.66.0.1
192.168.10.0/24 dev ens33 proto kernel scope link src 192.168.10.164 metric 100
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1
```

\
