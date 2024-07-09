# 部署



{% tabs %}
{% tab title="准备" %}
查看准备的磁盘

* ```bash
  [root@nfsserver ~]# lsblk
  NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
  sda               8:0    0  100G  0 disk
  ├─sda1            8:1    0    1G  0 part /boot
  └─sda2            8:2    0   99G  0 part
    ├─centos-root 253:0    0   50G  0 lvm  /
    ├─centos-swap 253:1    0    2G  0 lvm  [SWAP]
    └─centos-home 253:2    0   47G  0 lvm  /home
  sdb               8:16   0  100G  0 disk
  ```

&#x20;nfs配置

* ```bash
  # yum -y install nfs-utils

  创建挂载点
  # mkdir /netshare

  格式化硬盘
  # mkfs.xfs /dev/sdb

  编辑文件系统配置文件
  # vim /etc/fstab
  在文件最后添加此行内容
  /dev/sdb                /netshare               xfs     defaults        0 0

  手动挂载全部分区
  # mount -a

  在本地查看文件系统挂载情况
  # df -h
  文件系统                 容量  已用  可用 已用% 挂载点

  /dev/sdb                 100G   33M  100G    1% /netshare

  添加共享目录到配置文件
  # vim /etc/exports
  # cat /etc/exports
  /netshare       *(rw,sync,no_root_squash)

  启动服务及设置开机自启动
  # systemctl enable nfs-server
  # systemctl start nfs-server

  本地验证目录是否共享
  # showmount -e
  Export list for nfsserver:
  /netshare *

  在k8s master节点验证目录是否共享
  # showmount -e 192.168.10.143
  Export list for 192.168.10.143:
  /netshare *

  在k8s worker01节点验证目录是否共享
  # showmount -e 192.168.10.143
  Export list for 192.168.10.143:
  /netshare *
  ```

部署存储动态供给

* ```Go
  在k8s master节点获取NFS后端存储动态供给配置资源清单文件
  # for file in class.yaml deployment.yaml rbac.yaml  ; do wget https://raw.githubusercontent.com/kubernetes-incubator/external-storage/master/nfs-client/deploy/$file ; done

  查看是否下载
  # ls
  class.yaml  deployment.yaml  rbac.yaml
  ```
* ```Go
  应用rbac资源清单文件
  # kubectl apply -f rbac.yaml

  修改存储类名称
  # vim class.yaml
  # cat class.yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: nfs-client
  provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
  parameters:
    archiveOnDelete: "false"
    
  应用class（存储类）资源清单文件
  # kubectl apply -f class.yaml
  storageclass.storage.k8s.io/nfs-client created

  应用deployment资源清单文件之前修改其配置，主要配置NFS服务器及其共享的目录
  # vim deployment.yaml

  注意修改处内容

  # vim deployment.yaml
  # cat deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nfs-client-provisioner
    labels:
      app: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
  spec:
    replicas: 1
    strategy:
      type: Recreate
    selector:
      matchLabels:
        app: nfs-client-provisioner
    template:
      metadata:
        labels:
          app: nfs-client-provisioner
      spec:
        serviceAccountName: nfs-client-provisioner
        containers:
          - name: nfs-client-provisioner
            image: registry.cn-beijing.aliyuncs.com/pylixm/nfs-subdir-external-provisioner:v4.0.0
            volumeMounts:
              - name: nfs-client-root
                mountPath: /persistentvolumes
            env:
              - name: PROVISIONER_NAME
                value: fuseim.pri/ifs
              - name: NFS_SERVER
                value: 192.168.10.143
              - name: NFS_PATH
                value: /netshare
        volumes:
          - name: nfs-client-root
            nfs:
              server: 192.168.10.143
              path: /netshare
  ```
* ```Go
  应用资源清单文件
  # kubectl apply -f deployment.yaml

  查看pod运行情况

  # kubectl get pods
  出现以下表示成功运行
  NAME                                     READY   STATUS    RESTARTS   AGE
  nfs-client-provisioner-8bcf6c987-7cb8p   1/1     Running   0          74s

  设置默认存储类
  # kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

  # kubectl get sc
  NAME                   PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
  nfs-client (default)   fuseim.pri/ifs   Delete          Immediate           false                  18m
  ```
* ```Go
  测试用例：
  # vim nginx.yaml
  # cat nginx.yaml
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ---
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    selector:
      matchLabels:
        app: nginx
    serviceName: "nginx"
    replicas: 2
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:latest
          ports:
          - containerPort: 80
            name: web
          volumeMounts:
          - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "nfs-client"
        resources:
          requests:
            storage: 1Gi
  ```
* ```Go
  # kubectl apply -f nginx.yaml
  service/nginx created
  statefulset.apps/web created

  # kubectl get pvc
  NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  www-web-0   Bound    pvc-57bee742-326b-4d41-b241-7f2b5dd22596   1Gi        RWO            nfs-client     3m19s
  ```
* **metallb部署**
  1. ![](https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTY1ZDAxNTI4Y2ZhYjk3ZmVlNTc1ZWM2YzdiYmMwYTRfdEVmV24yRmdJWXJpaWh6M3YwU0RKcFc3bU05T1JIMFZfVG9rZW46RDRKMGJjUGJQb0NYZUZ4N0Y2emNIV0hGbkJiXzE3MTgxODg1Mjk6MTcxODE5MjEyOV9WNA)  **修改kube-proxy配置**
  2. ```Go
     [root@k8s-master01 ~]# kubectl get configmap -n kube-system
     NAME                                 DATA   AGE

     kube-proxy                           2      18d
     ```
  3. ```Go
     [root@k8s-master01 ~]# kubectl edit configmap kube-proxy -n kube-system
     ......
      ipvs:
           excludeCIDRs: null
           minSyncPeriod: 0s
           scheduler: ""
           strictARP: true 由false修改为true
           syncPeriod: 0s
           tcpFinTimeout: 0s
           tcpTimeout: 0s
           udpTimeout: 0s
         kind: KubeProxyConfiguration
         metricsBindAddress: ""
         mode: ipvs 由空修改为ipvs
         nodePortAddresses: null
     ```
  4. ```Go
     [root@k8s-master01 ~]# kubectl rollout restart daemonset kube-proxy -n kube-system
     ```
  5. &#x20; 安装
  6. ```Go
     [root@k8s-master01 ~]# kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
     ```
  7. ```Go
     [root@k8s-master01 ~]# kubectl get ns
     NAME              STATUS   AGE

     ......
     metallb-system    Active   1m

     [root@k8s-master01 ~]# kubectl get pods -n metallb-system
     NAME                          READY   STATUS    RESTARTS   AGE
     controller-6f5c46d94b-rwpch   1/1     Running   0          2m
     speaker-c4fbz                 1/1     Running   0          2m
     speaker-z9jqx                 1/1     Running   0          2m
     speaker-zzpkk                 1/1     Running   0          2m
     ```
  8. &#x20; **添加IP地址池及二层通告**
  9. ```Go
     [root@k8s-master01 ~]# cat ippool.yaml
     apiVersion: metallb.io/v1beta1
     kind: IPAddressPool
     metadata:
       name: first-pool
       namespace: metallb-system
     spec:
       addresses:
       - 192.168.10.200-192.168.10.210
     ```
  10. ```Go
      [root@k8s-master01 ~]# kubectl apply -f ippool.yaml
      ```
  11. ```Go
      [root@k8s-master01 ~]# kubectl get ipaddresspool -n metallb-system
      NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
      first-pool   true          false             ["192.168.10.200-192.168.10.210"]
      ```
  12. ```Go
      [root@k8s-master01 ~]# cat l2.yaml
      apiVersion: metallb.io/v1beta1
      kind: L2Advertisement
      metadata:
        name: example
        namespace: metallb-system
      ```
  13. ```Go
      [root@k8s-master01 ~]# kubectl apply -f l2.yaml
      ```

es准备

* ```Go
  [root@k8s-master01 ~]# mkdir k8s-es-kibana
  [root@k8s-master01 ~]# cd k8s-es-kibana/
  [root@k8s-master01 k8s-es-kibana]# ls
  elastic.yml  kibana.yml
  ```
* ```Go
  [root@k8s-master01 k8s-es-kibana]# vim elastic.yml
  [root@k8s-master01 k8s-es-kibana]# cat elastic.yml
  apiVersion: v1
  kind: Service
  metadata:
    name: elasticsearch
    namespace: default
    labels:
      app: elasticsearch
  spec:
    selector:
      k8s-app: elasticsearch
    clusterIP: None
    ports:
      - port: 9200
        name: db
      - port: 9300
        name: inter

  ---
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: elasticsearch
    namespace: default
    labels:
      k8s-app: elasticsearch
  spec:
    serviceName: elasticsearch
    selector:
      matchLabels:
        k8s-app: elasticsearch
    template:
      metadata:
        labels:
          k8s-app: elasticsearch
      spec:
        containers:
        - image: docker.elastic.co/elasticsearch/elasticsearch:7.17.2
          name: elasticsearch
          resources:
            limits:
              cpu: 1
              memory: 2Gi
            requests:
              cpu: 0.5
              memory: 500Mi
          env:
            - name: "discovery.type"
              value: "single-node"
            - name: ES_JAVA_OPTS
              value: "-Xms512m -Xmx512m"
          ports:
          - containerPort: 9200
            name: db
            protocol: TCP
          - name: inter
            containerPort: 9300
          volumeMounts:
          - name: elasticsearch-data
            mountPath: /usr/share/elasticsearch/data
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        storageClassName: "nfs-client"
        accessModes: [ "ReadWriteMany" ]
        resources:
          requests:
            storage: 5Gi
  ```
* ```Go
  [root@k8s-master01 k8s-es-kibana]# kubectl apply -f elastic.yml
  [root@k8s-master01 k8s-es-kibana]# vim kibana.yml
  [root@k8s-master01 k8s-es-kibana]# cat kibana.yml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: kibana
    namespace: default
    labels:
      k8s-app: kibana
  spec:
    replicas: 1
    selector:
      matchLabels:
        k8s-app: kibana
    template:
      metadata:
        labels:
          k8s-app: kibana
      spec:
        containers:
        - name: kibana
          image: docker.elastic.co/kibana/kibana:7.17.2
          resources:
            limits:
              cpu: 1
              memory: 1G
            requests:
              cpu: 0.5
              memory: 500Mi
          env:
            - name: ELASTICSEARCH_HOSTS
              value: http://elasticsearch.default.svc.cluster.local.:9200
          ports:
          - containerPort: 5601
            protocol: TCP

  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: kibana
    namespace: default
  spec:
    ports:
    - port: 5601
      protocol: TCP
      targetPort: 5601
    type: LoadBalancer
    selector:
      k8s-app: kibana
  ```
* ```Go
  [root@k8s-master01 k8s-es-kibana]# kubectl apply -f kibana.yml
  [root@k8s-master01 k8s-es-kibana]# kubectl get pods
  NAME                                      READY   STATUS    RESTARTS      AGE
  elasticsearch-0                           1/1     Running   0             6m15s
  kibana-5f59d74cc5-cfbgv                   1/1     Running   0             4m43s
  nfs-client-provisioner-69cbc84694-zcsg4   1/1     Running   4 (20m ago)   30m
  [root@k8s-master01 k8s-es-kibana]# kubectl get svc
  NAME            TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)             AGE
  elasticsearch   ClusterIP      None           <none>           9200/TCP,9300/TCP   6m19s
  kibana          LoadBalancer   10.233.47.58   192.168.10.200   5601:31985/TCP      4m46s
  kubernetes      ClusterIP      10.233.0.1     <none>           443/TCP             8d
  ```
* ![](https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=MzFhZDA3MmI4ZGEwYWIxOTgzY2Y5NjVlMTZlMDM1MDhfaldHT29FRTg5MEJUWjZlSEVvUlhqbGlXY1RUUUdyWG5fVG9rZW46U3lHZGI2M0hvb0U1WXd4NUdlN2Npa29EbnZnXzE3MTgxODg1Mjk6MTcxODE5MjEyOV9WNA)


{% endtab %}

{% tab title="安装skywalking" %}
* ```Go
  [root@k8s-master01 ~]# mkdir k8s-skywalking
  [root@k8s-master01 ~]# cd k8s-skywalking/
  [root@k8s-master01 k8s-skywalking]# vim skywalking-oap-server.yaml
  [root@k8s-master01 k8s-skywalking]# cat skywalking-oap-server.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: oap
    namespace: skywalking
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: oap
        release: skywalking
    template:
      metadata:
        labels:
          app: oap
          release: skywalking
      spec:
        containers:
          - name: oap
            image: apache/skywalking-oap-server:9.5.0
            imagePullPolicy: IfNotPresent
            ports:
              - containerPort: 11800
                name: grpc
              - containerPort: 12800
                name: rest
            env:
              - name: SW_STORAGE
                value: elasticsearch
              - name: SW_STORAGE_ES_CLUSTER_NODES
                value: elasticsearch.default.svc.cluster.local.:9200
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: oap
    namespace: skywalking
    labels:
      service: oap
  spec:
    ports:
      - port: 12800
        name: rest
      - port: 11800
        name: grpc
    type: ClusterIP
    selector:
      app: oap
  ```
* ```Go
  [root@k8s-master01 k8s-skywalking]# kubectl create ns skywalking
  [root@k8s-master01 k8s-skywalking]# kubectl apply -f skywalking-oap-server.yaml
  [root@k8s-master01 k8s-skywalking]# kubectl get pods -n skywalking
  NAME                   READY   STATUS    RESTARTS   AGE
  oap-78c764b8bd-9pkt2   1/1     Running   0          85s
  ```

#### &#x20;**skywalking ui**

* ```Go
  [root@k8s-master01 k8s-skywalking]# vim skywalking-ui.yaml
  [root@k8s-master01 k8s-skywalking]# cat skywalking-ui.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ui-deployment
    namespace: skywalking
    labels:
      app: ui
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: ui
    template:
      metadata:
        labels:
          app: ui
      spec:
        containers:
          - name: ui
            image: apache/skywalking-ui:9.5.0
            ports:
              - containerPort: 8080
                name: page
            env:
              - name: SW_OAP_ADDRESS
                value: http://oap.skywalking.svc.cluster.local.:12800
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: ui
    namespace: skywalking
    labels:
      service: ui
  spec:
    ports:
      - port: 8080
        name: page
    type: LoadBalancer
    selector:
      app: ui
  ```
* ```Go
  [root@k8s-master01 k8s-skywalking]# kubectl apply -f skywalking-ui.yaml
  [root@k8s-master01 k8s-skywalking]# kubectl get pods -n skywalking
  NAME                             READY   STATUS    RESTARTS   AGE
  oap-78c764b8bd-9pkt2             1/1     Running   0          4m36s
  ui-deployment-7c5df89f6c-fqpqz   1/1     Running   0          2m23s
  [root@k8s-master01 k8s-skywalking]# kubectl get svc -n skywalking
  NAME   TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)               AGE
  oap    ClusterIP      10.233.59.68   <none>           12800/TCP,11800/TCP   5m10s
  ui     LoadBalancer   10.233.13.64   192.168.10.201   8080:30214/TCP        2m57s
  ```
*   ![](https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=M2NhZWNiZjUxOTFlZWI0NWM4YjFjYTBmMzQ1Y2E2NjBfRWMxMUIzaWhSZXVnWlRGbGRBZ0JSclg3YmZNMlFIc1NfVG9rZW46Q0hXZmIxT3JIb0VvZ2J4STRKT2N2ak9EbjZjXzE3MTgxODg1Mjk6MTcxODE5MjEyOV9WNA)

    <figure><img src="https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTc3ZTc5NDZmNzY3ZGZhMzYzZjVkZDY3NzY0ZTc0MTNfNjdwczY3NHdjNG9nQWNoMXh6bE56ODV6WWZCc3BuaE1fVG9rZW46TXJsUWI5V0hRb0F6TEp4bTJSM2NkN1lrbjNlXzE3MTgxODg1Mjk6MTcxODE5MjEyOV9WNA" alt=""><figcaption></figcaption></figure>


{% endtab %}
{% endtabs %}

1.

\
