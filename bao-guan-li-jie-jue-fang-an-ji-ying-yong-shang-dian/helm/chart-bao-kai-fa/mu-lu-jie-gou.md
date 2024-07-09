# 目录结构

```
[root@k8s-master01 helmdir]# helm create foo
​
[root@k8s-master01 helmdir]# tree foo
foo
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

```
[root@master ~]# helm pull stable/mysql
​
[root@master ~]# tar xf mysql-1.6.8.tgz
​
[root@master ~]# ls mysql
Chart.yaml  README.md  templates  values.yaml
​
[root@master ~]# ls -l mysql/templates/ 
total 48
-rwxr-xr-x 1 root root  292 Jan  1  1970 configurationFiles-configmap.yaml
-rwxr-xr-x 1 root root 8930 Jan  1  1970 deployment.yaml
-rwxr-xr-x 1 root root 1290 Jan  1  1970 _helpers.tpl
-rwxr-xr-x 1 root root  295 Jan  1  1970 initializationFiles-configmap.yaml
-rwxr-xr-x 1 root root 2036 Jan  1  1970 NOTES.txt
-rwxr-xr-x 1 root root  868 Jan  1  1970 pvc.yaml
-rwxr-xr-x 1 root root 1475 Jan  1  1970 secrets.yaml
-rwxr-xr-x 1 root root  328 Jan  1  1970 serviceaccount.yaml
-rwxr-xr-x 1 root root  800 Jan  1  1970 servicemonitor.yaml
-rwxr-xr-x 1 root root 1231 Jan  1  1970 svc.yaml
drwxr-xr-x 2 root root   50 Nov 13 18:43 tests
```

<table data-header-hidden><thead><tr><th width="194">文件</th><th>说明</th></tr></thead><tbody><tr><td>Chart.yaml</td><td>用于描述Chart的基本信息; <code>helm show chart stable/mysql</code>命令查看的内容就是此文件内容</td></tr><tr><td>values.yaml</td><td>Chart的默认配置文件; <code>helm show values stable/mysql</code>命令查看的内容就是此文件内容</td></tr><tr><td>README.md</td><td>[可选] 当前Chart的介绍</td></tr><tr><td>LICENS</td><td>[可选] 协议</td></tr><tr><td>requirements.yaml</td><td>[可选] 用于存放当前Chart依赖的其它Chart的说明文件</td></tr><tr><td>charts/</td><td>[可选]: 该目录中放置当前Chart依赖的其它Chart</td></tr><tr><td>templates/</td><td>[可选]: 部署文件模版目录</td></tr></tbody></table>
