---
description: 使用config配置文件
---

# 如何配置

## 目的

**使用不同的SA/UA账户并搭配不同的config文件来进行管理集群**

> SA：service account 服务账户
>
> UA：username account 用户账户
>
> 都需要认证和授权，而且都是以插件模式，并且按照短路模型来运行

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

## 集群命令解读

```
集群操作
	get-clusters    显示 kubeconfig 文件中定义的集群
	delete-cluster  删除 kubeconfig 文件中指定的集群
	set-cluster     Set a cluster entry in kubeconfig
	
用户操作 
	delete-user     Delete the specified user from the kubeconfig
	get-users       Display users defined in the kubeconfig
	set-credentials Set a user entry in kubeconfig
	
上下文操作 让用户和集群绑定
	current-context 
	delete-context  删除 kubeconfig 文件中指定的 context
	get-contexts    描述一个或多个 contexts
	rename-context  Rename a context from the kubeconfig file
	set-context     Set a context entry in kubeconfig
	use-context     Set the current-context in a kubeconfig file
	
其他操作 
    set             Set an individual value in a kubeconfig file
    unset           Unset an individual value in a kubeconfig file	  
    view            显示合并的 kubeconfig 配置或一个指定的 kubeconfig 文件
```
