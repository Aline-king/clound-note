# 访问kubeapp

```bash
# vim kubeapps-ingress.yaml

# cat kubeapps-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-kubeapps                    #自定义ingress名称
  namespace: kubeapps
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: kubeapps.kubemsb.com                   # 自定义域名
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: kubeapps     # 对应上面创建的service名称
            port:
              number: 80
```

```
# kubectl apply -f kubeapps-ingress.yaml
ingress.networking.k8s.io/ingress-kubeapps created
```

<figure><img src="../../.gitbook/assets/image-20220731003937933.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://kubeapps.dev/docs/latest/tutorials/getting-started/" %}

## 额外设置

{% tabs %}
{% tab title="创建用户" %}
**`kubectl create --namespace default serviceaccount kubeapps-operator`**
{% endtab %}

{% tab title="绑定集群管理员角色" %}
`kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator`
{% endtab %}

{% tab title="获取token" %}
```bash
# cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kubeapps-operator-token
  namespace: default
  annotations:
    kubernetes.io/service-account.name: kubeapps-operator
  type: kubernetes.io/service-account-token
EOF


输出：
secret/kubeapps-operator-token created
```

**生成token**

`kubectl get --namespace default secret kubeapps-operator-token -o jsonpath='{.data.token}' -o go-template='{{.data.token | base64decode}}' && echo`
{% endtab %}
{% endtabs %}

<figure><img src="../../.gitbook/assets/image-20220731005426854.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image-20220731005818965.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image-20220731005916797.png" alt=""><figcaption></figcaption></figure>
