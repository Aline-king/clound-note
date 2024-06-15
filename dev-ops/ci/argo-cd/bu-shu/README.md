# 部署

分为“多租户”和“只核心功能”

{% tabs %}
{% tab title="多租户" %}
多租户安装是安装 Argo CD 的最常见方式。

&#x20;这种类型的安装通常用于为组织中的多个应用程序开发团队提供服务，并由平台团队维护。&#x20;

终端用户可以使用 Web UI 或 argocd CLI 通过 API 服务器访问 Argo CD。&#x20;

> 访问方式：argocd login \<server-host> 命令配置 argocd CLI

它有两种安装方式：

1. ~~**没有高可用（不推荐生产部署**~~**）**

这种类型的安装通常在评估期间用于演示和测试。&#x20;

这里面还有两种安装级别：

* 集群管理员可执行的安装[install.yaml](https://github.com/argoproj/argo-cd/blob/master/manifests/install.yaml) 标准安装，建议要部署应用的集群和 argocd 是同一个集群，即 kubernetes.svc.default 的时候使用此安装方案
*   命名空间管理员可执行的安装 \[namespace-install.yaml] ([argo-cd/namespace-install.yaml at master · argoproj/argo-cd · GitHub](https://github.com/argoproj/argo-cd/blob/master/manifests/namespace-install.yaml)) 仅在 namespace 级别使用，如果部署的应用所在集群和 argocd 不在同一个集群，可以使用此安装方案。这里可以指定集群作为参数来部署。值得注意的是，这里没有安装对应的crd。

    需要集群管理员来安装 `kubectl apply -k https://github.com/argoproj/argo-cd/manifests/crds\?ref\=stable`



2. **有高可用**

与没有高可用的安装方式和类别相同，只是增加了高可用的配置。

> [ha/install.yaml](https://github.com/argoproj/argo-cd/blob/master/manifests/ha/install.yaml)
>
> [ha/namespace-install.yaml](https://github.com/argoproj/argo-cd/blob/master/manifests/ha/namespace-install.yaml)
{% endtab %}

{% tab title="核心功能" %}
[core-install.yaml](https://github.com/argoproj/argo-cd/blob/master/manifests/core-install.yaml)

此种安装最适合独立使用 Argo CD 且不需要多租户特性的集群管理员。&#x20;

此安装包含更少的组件并且更易于设置。 不包含 API 服务器或 UI，并安装每个组件的轻量级（非 HA）版本。 最终用户需要 Kubernetes 访问权限来管理 Argo CD。&#x20;

argocd CLI 必须设置默认 namespace 为 argocd

> 设置方式：`kubectl config set-context --current --namespace=argocd`
{% endtab %}
{% endtabs %}



```bash

curl -LO https://github.com/argoproj/argo-cd/releases/download/v2.5.0/argocd-linux-amd64 
```

```
sudo mv argocd-linux-amd64 /usr/local/bin/argocd sudo chmod +x /usr/local/bin/argocd 
```

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo 
```

```bash
kubectl port-forward svc/argocd-server -n 
```

```
argocd 8080:443 argocd login localhost:8080 
```

```
argocd account update-password
```
