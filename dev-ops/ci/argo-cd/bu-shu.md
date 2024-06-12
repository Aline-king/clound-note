# 部署

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
