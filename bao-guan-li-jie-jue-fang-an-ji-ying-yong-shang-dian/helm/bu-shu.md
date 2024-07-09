# 部署



```
[root@k8s-master01 ~]# wget https://get.helm.sh/helm-v3.9.2-linux-amd64.tar.gz
```

```
[root@k8s-master01 ~]#  ls
helm-v3.9.2-linux-amd64.tar.gz
```

```
[root@k8s-master01 ~]# tar xf helm-v3.9.2-linux-amd64.tar.gz
```

```
[root@k8s-master01 ~]# ls
linux-amd64
```

```
[root@k8s-master01 ~]# cd linux-amd64/
[root@k8s-master01 linux-amd64]# ls
helm  LICENSE  README.md
```

```
[root@k8s-master01 linux-amd64]# mv helm /usr/bin
```

```
[root@k8s-master01 linux-amd64]# helm version
version.BuildInfo{Version:"v3.9.2", GitCommit:"1addefbfe665c350f4daf868a9adc5600cc064fd", GitTreeState:"clean", GoVersion:"go1.17.12"}
```

\
