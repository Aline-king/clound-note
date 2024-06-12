# 网络模型

<table><thead><tr><th width="164">类型</th><th>说明</th></tr></thead><tbody><tr><td>None</td><td>不为容器配置任何网络功能，没有网络 --net=none</td></tr><tr><td>Container</td><td>与另一个运行中的容器共享Network Namespace，--net=container:containerID</td></tr><tr><td>Host</td><td>与主机共享Network Namespace，--net=host 使用宿主机ip+容器端口 网络性能高，业务比较固定，</td></tr><tr><td>Bridge</td><td>Docker设计的NAT网络模型（默认类型）Bridge默认docker网络隔离基于网络命名空间，在物理机上创建docker容器时会为每一个docker容器分配网络命名空间，并且把容器IP桥接到物理机的虚拟网桥上。</td></tr></tbody></table>
