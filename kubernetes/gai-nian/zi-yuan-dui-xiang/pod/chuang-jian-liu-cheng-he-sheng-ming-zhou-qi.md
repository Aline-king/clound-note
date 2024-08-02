# 创建流程和生命周期

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1 用户通过多种方式向master结点上的API-Server发起创建一个Pod的请求&#x20;

2 apiserver 将资源对象操作信息写入 etcd&#x20;

3 scheduluer 检测到api-Server上有建立Pod请求，开始调度该Pod到指定的Node 同时借助于apiserver更新信息到etcd&#x20;

4 kubelet 检测到有新的 Pod 调度过来，通过容器引擎(不限于Docker)创建并运行该 Pod对象&#x20;

5 kubelet 通过 container runtime 取到 Pod 状态，并同步信息到 apiserver，由它更新信息到etcd

##
