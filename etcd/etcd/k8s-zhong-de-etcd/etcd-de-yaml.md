# etcd的yaml

## yaml解析

```yaml
查看pod的基本信息查看
[root@kubernetes-master1 ~]# cat /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  ...
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --client-cert-auth=true # 启用客户端证书验证
    # 客户端服务器TLS证书文件的路径
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    # 客户端服务器TLS密钥文件的路径
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    # 客户端服务器的路径TLS可信CA证书文件
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    # 数据存储目录，在容器内部以ETCD_DATA_DIR变量存在
    # --wal-dir存放预写式日志,记录了数据变化的历程，不写与data-dir一致。
    - --data-dir=/var/lib/etcd

    # 节点名称,该值与集群初始化时候的--initial-cluster值一致
    - --name=kubernetes-master1    
    # 监听数据地址地址
    - --listen-client-urls=https://127.0.0.1:2379,https://10.0.0.12:2379
    # 要监听的其他URL列表将响应端点/metrics和/health端点
    - --listen-metrics-urls=http://127.0.0.1:2381
    # 与其他节点进行数据交换(选举，数据同步)的监听地址
    - --listen-peer-urls=https://10.0.0.12:2380
    # 用于通知其他ETCD节点，客户端接入本节点的监听地址
    - --advertise-client-urls=https://10.0.0.12:2379	
    # 通知其他节点与本节点进行数据交换（选举，同步）的地址
    # 属于listen-peer-urls属性值的子集 
    - --initial-advertise-peer-urls=https://10.0.0.12:2380
    # 初始化etcd集群时候，设定集群中所有节点的名称信息
    - --initial-cluster=kubernetes-master1=https://10.0.0.12:2380
    
    # etcd服务器集群间通信启用对等客户端证书验证
    - --peer-client-cert-auth=true
    # etcd服务器集群间通信用TLS证书文件的路径
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    # etcd服务器集群间通信用TLS密钥文件的路径
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    # etcd服务器集群间通信用TLS可信CA文件的路径
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    
    # 设定事务提交后，数据快照触发数量，容器内部以ETCD_SNAPSHOT_COUNT变量方式存在
    - --snapshot-count=10000
    # 数据传输时候，检查数据的有效性
    - --experimental-initial-corrupt-check=true
    ...
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
```
