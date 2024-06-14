# 运行状态

## pod状态

pod运行成功还是失败，总会是五个状态的一种。

> 官方资料：[https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
>
> [https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-phase)

查看生命周期

```bash
kubectl explain pods.status.phase
```

1. pending
2. running
3. succeeded
4. failed
5. Unknown

k8s官方提供三种策略进行管理

1. OnFailure，
2. Never
3. Always（默认）

## 容器状态

1. Waiting 容器处于 Running 和Terminated 状态之前的状态
2. &#x20;Running 容器能够正常的运行状态，容器正常了，pod提供的服务才可以对外开放&#x20;
3. Terminated 容器已经被成功的关闭了

## 镜像拉取状态

> [https://kubernetes.io/docs/concepts/containers/images#updating-images](https://kubernetes.io/docs/concepts/containers/images#updating-images)

1. &#x20;Always pod每次创建容器，都是从远程仓库拉取新镜像，这是默认值&#x20;
2. IfNotPresent 如果Node所在节点的本地仓库不存在的话，再拉取新镜像&#x20;
3. Never不获取新镜像,仅用本地镜像来运行容器

