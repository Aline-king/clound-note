# net

保障每个容器有独立的网络栈、socket 和网卡设备。

NET Namespace 使用了参数 CLONE\_NEWNET，隔离了和网络有关的资源，如网络设备、IP 地址端口等。
