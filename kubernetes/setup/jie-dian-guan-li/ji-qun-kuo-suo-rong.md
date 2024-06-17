# 集群扩缩容

<details>

<summary>集群缩容</summary>

1. 基本环境确认

```
准备应用环境
[root@kubernetes-master1 ~]# kubectl  create deployment nginx-web --image=kubernetes-register.superopsmsb.com/superopsmsb/nginx_web:v0.1 --replicas=3
deployment.apps/nginx-web created
​
查看效果
[root@kubernetes-master1 ~]# kubectl get pod -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE               NOMINATED NODE   READINESS GATES
nginx-web-6c84585c7b-b6xc9   1/1     Running   0          6s    10.244.2.41   kubernetes-node2   <none>           <none>
nginx-web-6c84585c7b-fsxdk   1/1     Running   0          6s    10.244.1.38   kubernetes-node1   <none>           <none>
nginx-web-6c84585c7b-kxhc4   1/1     Running   0          6s    10.244.3.2    kubernetes-node3   <none>           <none>
```

2. 冻结待删除节点

```
使用cordon冻结节点
[root@kubernetes-master1 ~]# kubectl cordon kubernetes-node3
node/kubernetes-node3 cordoned
​
查看效果
[root@kubernetes-master1 ~]# kubectl get node kubernetes-node3
NAME               STATUS                     ROLES    AGE    VERSION
kubernetes-node3   Ready,SchedulingDisabled   <none>   5m3s   v1.23.8
```

3. 驱离待删除节点资源

因为当前节点还需要被管理，所以需要留一个集群组件服务

{% code title="使用drain命令清理一般资源对象" %}
```bash
[root@kubernetes-master1 ~]# kubectl  drain kubernetes-node3 --delete-emptydir-data  --force
```
{% endcode %}

```
强制将ds等相关资源驱离
[root@kubernetes-master1 ~]# kubectl taint node kubernetes-node3 diskfull=true:NoExecute
[root@kubernetes-master1 ~]# kubectl get pod -o wide -n kube-flannel  | grep node3
[root@kubernetes-master1 ~]# kubectl get pod -o wide -n kube-system  | grep node3
kube-proxy-69sgn                             1/1     Running   0                11m     10.0.0.17     kubernetes-node3     <none>           <none>
```

```
这种方式对于ds类型的资源无效
[root@kubernetes-master1 ~]# kubectl get pod -o wide -n kube-flannel  | grep node3
kube-flannel-ds-5w746   1/1     Running   0               10m    10.0.0.17   kubernetes-node3     <none>           <none>
```

```
node3节点确认效果
[root@kubernetes-node3 ~]# docker ps  | grep -v NAME | wc -l
2
```

4. 执行节点删除命令

```
使用delete移除节点
[root@kubernetes-master1 ~]# kubectl  delete nodes kubernetes-node3
node "kubernetes-node3" deleted
​
master节点确认效果
[root@kubernetes-master1 ~]# kubectl  get nodes kubernetes-node3
Error from server (NotFound): nodes "kubernetes-node3" not found
```

5. 被删除节点清理集群环境

```
集群环境还原
[root@kubernetes-node3 ~]# kubeadm reset
​
清理网络配置
[root@kubernetes-node3 ~]# rm -f /etc/cni/net.d/*
```

7. 重启被删除节点主机
   * 这步的目的是将当期节点上遗留的网络相关信息记录全部清理

```
重启主机
[root@kubernetes-node3 ~]# reboot
```

</details>



<details>

<summary>集群扩容</summary>

1. 待添加节点准备集群环境

```
设定主机名
[root@localhost ~]# hostnamectl set-hostname kubernetes-node4
[root@localhost ~]# exec /bin/bash
[root@kubernetes-node4 ~]#
​
获取文件
[root@kubernetes-node4 ~]# mkdir /data/scripts -p ; cd /data/scripts
[root@kubernetes-node4 /data/scripts]# scp root@10.0.0.12:/data/scripts/* ./
[root@kubernetes-node4 /data/scripts]# scp root@10.0.0.12:/etc/yum.repos.d/kubernetes.repo /etc/yum.repos.d/kubernetes.repo
[root@kubernetes-node4 /data/scripts]# scp root@10.0.0.12:/etc/hosts /etc/hosts
```

```
主机内核调整
[root@kubernetes-node4 /data/scripts]# /bin/bash 02_kubernetes_kernel_conf.sh
vm.swappiness = 0
vm.swappiness = 0
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

```
集群环境软件环境配置
[root@kubernetes-node4 /data/scripts]# /bin/bash 03_kubernetes_docker_install.sh
​
准备docker环境
[root@kubernetes-node4 /data/scripts]# scp root@10.0.0.12:/etc/docker/daemon.json /etc/docker/daemon.json
​
重启docker服务
[root@kubernetes-node4 /data/scripts]# systemctl restart docker
[root@kubernetes-node4 /data/scripts]# systemctl enable docker
​
确认效果
[root@kubernetes-node4 /data/scripts]# docker info | egrep 'systemd|superopsmsb'
 Cgroup Driver: systemd
  kubernetes-register.superopsmsb.com
```

```
安装基础软件
[root@kubernetes-node4 ~]# yum install -y kubelet-1.23.8-0 kubeadm-1.23.8-0
```

2. 添加节点的到集群

```
master节点定制节点添加命令
[root@kubernetes-master1 ~]# kubeadm token create --print-join-command
kubeadm join 10.0.0.200:6443 --token 0om798.7e1x521h7fog3o50 --discovery-token-ca-cert-hash sha256:d8a77a69fb0b54cd72a692be83fdcd2c39f203bdc6e729b9d7d63cca3030cfcc
```

```
新节点执行添加命令
[root@kubernetes-node4 ~]# kubeadm join 10.0.0.200:6443 --token 0om798.7e1x521h7fog3o50 --discovery-token-ca-cert-hash sha256:d8a77a69fb0b54cd72a692be83fdcd2c39f203bdc6e729b9d7d63cca3030cfcc
```

\
因为是新的节点，所以依赖的镜像需要准备一段时间，等待一段时间后，就可以转变为正常状态了

```
master节点确认效果
[root@kubernetes-master1 ~]# kubectl get nodes kubernetes-node4
NAME               STATUS     ROLES    AGE    VERSION
kubernetes-node4   NotReady   <none>   118s   v1.23.8
   
[root@kubernetes-master1 ~]# kubectl get nodes kubernetes-node4
NAME               STATUS   ROLES    AGE     VERSION
kubernetes-node4   Ready    <none>   4m59s   v1.23.8
```

</details>

