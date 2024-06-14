# 演示

```
准备专属目录
[root@kubernetes-master1 /data/kubernetes/storage]# mkdir sc-provider
[root@kubernetes-master1 /data/kubernetes/storage]# cd sc-provider/
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]#

准备镜像
docker pull registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner
docker tag registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner kubernetes-register.superopsmsb.com/google_containers/nfs-client-provisioner:latest
docker push kubernetes-register.superopsmsb.com/google_containers/nfs-client-provisioner:latest
docker rmi registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner
```

```
准备NFS的控制器
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]# cat 01_kubernetes_sc_provisioner.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  # 命名空间要与定制的rbac的一致
  namespace: superopsmsb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccount: nfs-provisioner
      containers:
      - name: nfs-client-provisioner
        image: kubernetes-register.superopsmsb.com/google_containers/nfs-client-provisioner:latest
        volumeMounts:
        - name: nfs-client-root
          mountPath: /persistentvolumes
        env:
        - name: PROVISIONER_NAME
             # 该变量的值，必须与nfs的storageclass的provisioner的值一致
          value: "nfsprovisioner"
        - name: NFS_SERVER
              # 设置NFS服务器的ip地址
          value: "10.0.0.18"
        - name: NFS_PATH
              # 设置NFS服务器分享的目录
          value: "/superopsmsb/nfs-data"
      volumes:
      - name: nfs-client-root
        nfs:
          # 直接使用nfs来挂载该目录，方便storageclass基于该pod对pv和pvc进行自动处理
          server: "10.0.0.18"
          path: "/superopsmsb/nfs-data"
```

```
准备NFS操作k8s资源的权限
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]# cat 02_kubernetes_sc_rbac.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nfsstorageclass
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
  namespace: superopsmsb
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
   name: nfs-provisioner
   namespace: superopsmsb
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services", "endpoints"]
    verbs: ["get","create","patch","list", "watch","update"]
  - apiGroups: ["extensions"]
    resources: ["podsecuritypolicies"]
    resourceNames: ["nfs-provisioner"]
    verbs: ["use"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner
  namespace: superopsmsb
subjects:
  - kind: ServiceAccount
    name: nfs-provisioner
    namespace: superopsmsb
roleRef:
  kind: ClusterRole
  name: nfs-provisioner
  apiGroup: rbac.authorization.k8s.io
```

StorageClass 的name不允许出现 大写字母

```
准备NFS操作的StorageClass资源
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]# cat 03_kubernetes_sc_pv.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storageclass
  namespace: superopsmsb
# 每个 StorageClass 都包含 provisioner、parameters 和 reclaimPolicy 字段
# provisioner用来决定使用哪个卷插件分配PV，必须与nfs-client的容器内部的 PROVISIONER_NAME 变量一致
# reclaimPolicy指定创建的Persistent Volume的回收策略
provisioner: "nfsprovisioner"
reclaimPolicy: Retain
parameters:
  # archiveOnDelete: "false"表示在删除时不会对数据进行打包，当设置为true时表示删除时会对数据进行打包
  archiveOnDelete: "false"
```

```
创建专属命名空间
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]# kubectl  create ns superopsmsb
namespace/superopsmsb created

创建资源对象
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]# kubectl  apply -f ./
deployment.apps/nfs-client-provisioner created
namespace/nfsstorageclass created
serviceaccount/nfs-provisioner created
clusterrole.rbac.authorization.k8s.io/nfs-provisioner created
clusterrolebinding.rbac.authorization.k8s.io/nfs-provisioner created
storageclass.storage.k8s.io/storageclass created

检查效果
[root@kubernetes-master1 /data/kubernetes/storage/sc-provider]# kubectl  get all -n superopsmsb
NAME                                          READY   STATUS    RESTARTS   AGE
pod/nfs-client-provisioner-797775d949-pf9x2   1/1     Running   0          38s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nfs-client-provisioner   1/1     1            1           38s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/nfs-client-provisioner-797775d949   1         1         1       38s
```
