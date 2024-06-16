# 生命周期与探测

## 探测

> \[参考资料] ([https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes))

对于k8s内部的pod环境来说，常见的这些API接口有：

* process health 状态健康检测接口&#x20;
* metrics 监控指标接口&#x20;
* liveness 容器存活状态的接口&#x20;
  * > 周期性检测，检测未通过时，kubelet会根据restartPolicy的定义来决定是否会重启该容器
    >
    > 未定义时，Kubelet认为只要容器未终止，即为健康；
    >
    > ```bash
    > kubectl explain pod.spec.containers.livenessProbe
    > ```
*   readiness 容器可读状态的接口&#x20;

    > 周期性检测，检测未通过时，与该Pod关联的Service，会将该Pod从Service的后端可用端点列表中删除；直接再次就绪，重新添加回来。
    >
    > 未定义时，只要容器未终止，即为就绪；
    >
    > 注意：ReadinessProbe 检测失败不会重启Pod
    >
    > ```bash
    > kubectl explain pod.spec.containers.ReadnessProbe
    > ```
* tracing 全链路监控的埋点(探针)接口
* logs 容器日志接口

StartupProbe：&#x20;

实现用户自定义的一些探测机制，效果等同于livenessProbe，区别在于应用不同的参数或阈值；&#x20;

```
kubectl explain pod.spec.containers.startupProbe
```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. init容器 初始化容器，独立于主容器之外，pod可以拥有任意数量的init容器，所有init顺序执行完成后，才启动主容器。它主要是为了为主容器准备应用的功能，比如向主容器的存储卷写入数据，然后将存储卷挂载到主容器上。
2. 生命周期钩子 启动后钩子 PostStart - 主程序启动后执行的程序 运行中钩子 Liveiness - 判断当前容器是否处于存活状态 Readiness - 判断当前容器是否可以正常的对外提供服务 停止前钩子 PreStop - 主程序关闭前执行的程序。
3. pod关闭 当API服务器接收到删除pod对象的命令后，移除资源对象。

## 探针类型

kubelet定期执行livenessProbe探针来诊断Pod的健康状况，Pod探针的实现方式有很多，常见的有如下三种：

ExecAction - 直接执行命令，命令成功返回表示探测成功；&#x20;

TCPSocketAction - 端口能正常打开，即成功&#x20;

HTTPGetAction - 向指定的path发HTTP请求，2xx, 3xx的响应码表示成功

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

这里面仅仅罗列的livenessProbe ，readnessProbe 的属性与livenessProbe一样

```yaml
spec:
  containers:
  - name: …
    image: …
    livenessProbe:
      exec <Object>     		# 命令式探针
      httpGet <Object>  		# http GET类型的探针
      tcpSocket <Object>  		# tcp Socket类型的探针
      initialDelaySeconds <integer>  	# 发起初次探测请求的延后时长
      periodSeconds <integer>         	# 请求周期
      timeoutSeconds <integer>        	# 超时时长，默认是1。
      successThreshold <integer>      	# 连续成功几次，才表示状态正常，默认值是1
      failureThreshold <integer>      	# 连续失败几次，才表示状态异常，默认值是3
```
