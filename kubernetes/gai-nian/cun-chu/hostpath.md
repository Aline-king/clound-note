# Hostpath

<figure><img src="https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=NTlhOWEwZWUzNTU1N2I5OWY3ZTc4NDY0NGJiMTk0MWNfNFNid2FBaUtWV3dEWXVSWnFnY3BwaDlTalFjUG1ScmNfVG9rZW46Q0dRRWI0UmJZb2hwU0J4RlJWd2NzeGxJbkdJXzE3MTgyMTYxMDA6MTcxODIxOTcwMF9WNA" alt=""><figcaption></figcaption></figure>

## 配置格式

> 参考资料：https://kubernetes.io/docs/concepts/storage/volumes#hostpath

{% hint style="info" %}
配置属性解读&#x20;

kubectl explain pod.spec.volumes.hostPath&#x20;

path 指定宿主机的路径&#x20;

type 指定路径的类型，一共有7种，默认的类型是没有指定的话则自己创建&#x20;

1. DirectoryOrCreate 宿主机上不存在，创建此0755权限的空目录，属主属组均为kubelet
2. Directory 必须存在挂载已存在的目录，最常用&#x20;
3. FileOrCreate 宿主机上不存在挂载文件，就创建0644权限的空文件，属主和属组同为kubelet
4. File 必须存在文件&#x20;
5. Socket 事先必须存在的Socket文件路径&#x20;
6. CharDevice 事先必须存在的字符设备文件路径&#x20;
7. BlockDevice 事先必须存在的块设备文件路径
{% endhint %}

```yaml
  volumes:
  - name: volume_name
    hostPath:
     path: /path/to/host
```
