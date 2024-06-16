# secret

## 定义

secret volume为Pod提供加密的信息

* 相比于直接将敏感数据配置在Pod的定义或者镜像中，Secret提供了更加安全的机制，将共享的数据进行加密，防止数据泄露。&#x20;
* Secret的对象需要单独定义并创建，然后以数据卷的形式挂载到Pod中，Secret的数据将以文件的形式保存，容器通过读取文件可以获取需要的数据。&#x20;
  * secret volume是通过tmpfs（内存文件系统）实现的，所以这种类型的volume不是永久存储的。

## 方式

1. 手动创建：大部分情况用来存储用户私有的一些信息&#x20;
   1. &#x20;创建方式与CM的方式一致&#x20;
2. 自动创建：用来作为集群中各个组件之间通信的身份校验使用

## secret类型

1. generic&#x20;

通用类型，基于base64编码的，长用来存储密码，公钥之类的。常见的子类型有：kubernetes.io/basic-auth、kubernetes.io/rbd、kubernetes.io/ssh-auth(比较重要)

2. docker-registry&#x20;

专用于让kubelet启动Pod时从私有镜像仓库pull镜像时，首先认证到Registry时使用

3. tls&#x20;

专门用于保存tls/ssl用到证书和配对儿的私钥

