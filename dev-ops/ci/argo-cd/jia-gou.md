# 架构

#### API server

gRPC/REST 服务器，服务于客户端的图形界面，命令行工具，还有ci/cd的逻辑。他对以下功能负责：

* 应用管理和状态报告
* 应用程序的操作。它对一些声明式 api 实现功能以外的内容负责，也就是一些过程式的操作。如高级部署、回滚，自定义的行为。
* 代码仓库和 k8s 集群的授权管理。这些存储在 k8s 的 secret 里面
* 对外部的资源进行验证和代理验证
* rbac 的实现
* 监听、转发代码仓库的事件

#### Repository Server

代码仓库服务器，为代码服务器（git）管理一个本地的缓存。实际上是把git仓库的文件存在本地，便于读取之后进行下一步的操作。他的行为是当输入以下内容时，他会返回真实的描述信息，这些信息可以被用来部署应用：

* 仓库地址
* 版本信息，如commitID、tag、branch
* 应用的路径
* 模版的设置，如：参数、helm 使用的 value.yaml 文件

#### Application Controller

实现 k8s api 的机制，将期望和现状进行reconcile，维护其生命周期。负责一些 hook 的触发执行，如 PreSync、Sync、PostSync。用户可以自定义这些 hook
