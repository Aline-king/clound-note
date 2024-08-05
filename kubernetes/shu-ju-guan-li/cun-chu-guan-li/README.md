# 存储管理

## pv与pvc与sc的三者关系

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

## 原理解释

<figure><img src="../../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

1 用户创建了一个包含 PVC 的 Pod，该 PVC 要求使用动态存储卷；

&#x20;2 Scheduler 根据 Pod 配置、节点状态、PV 配置等信息，把 Pod 调度到一个合适的 Worker 节点上；

&#x20;3 PV 控制器 watch 到该 Pod 使用的 PVC 处于 Pending 状态，于是调用 Volume Plugin(in-tree)创建存储卷，并创建 PV 对象(out-of-tree 由 External Provisioner 来处理)；&#x20;

4 AD 控制器发现 Pod 和 PVC 处于待挂接状态，于是调用 Volume Plugin 挂接存储设备到目标 Worker 节点上

5 在 Worker 节点上，Kubelet 中的 Volume Manager 等待存储设备挂接完成，并通过 Volume Plugin 将设备挂载到全局目录： **/var/lib/kubelet/pods/\[pod uid]/volumes/kubernetes.io\~iscsi/\[PVname]**(以 iscsi 为例)；&#x20;

6 Kubelet 通过 Docker 启动 Pod 的 Containers，用 bind mount 方式将已挂载到本地全局目录的卷映射到容器中。



## 生命周期



<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Availabled 空闲状态，表示pv没有被其他对象使用

Bound 绑定状态，表示pv已经被其他对象使用

Released 未回收状态，表示pvc已经被删除了，但是资源没有被回收

Faild 资源回收失败
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

## AccessModes&#x20;

是用来对 PV 进行访问模式的设置，用于描述用户应用对存储资源的访问权限，访问权限包括

* ReadWriteOnce（RWO）单节点读写
* ReadOnlyMany（ROX）多节点只读
* ReadWriteMany（RWX）多节点读写
* ReadWriteOncePod(RWOP)单pod读写
