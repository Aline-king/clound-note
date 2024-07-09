# helm部署kubeapps

使用helm部署kubeapps

```bash
# helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

# helm repo list
NAME                    URL
micosoft                http://mirror.azure.cn/kubernetes/charts/
prometheus-community    https://prometheus-community.github.io/helm-charts
harborhelm              https://www.kubemsb.com/chartrepo/nginx
bitnami                 https://charts.bitnami.com/bitnami

# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harborhelm" chart repository
...Successfully got an update from the "prometheus-community" chart repository
...Successfully got an update from the "micosoft" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈

# helm search repo kubeapps
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/kubeapps        10.0.2          2.4.6           Kubeapps is a web-based UI for launching and ma...
```

```
# kubectl create ns kubeapps
namespace/kubeapps created

# helm install kubeapps bitnami/kubeapps --namespace kubeapps
```

{% code title="输出信息" %}
```bash
NAME: kubeapps
LAST DEPLOYED: Sun Jul 31 00:00:03 2022
NAMESPACE: kubeapps
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kubeapps
CHART VERSION: 10.0.2
APP VERSION: 2.4.6** Please be patient while the chart is being deployed **
​
Tip:
​
  Watch the deployment status using the command: kubectl get pods -w --namespace kubeapps
​
Kubeapps can be accessed via port 80 on the following DNS name from within your cluster:
​
   kubeapps.kubeapps.svc.cluster.local
​
To access Kubeapps from outside your K8s cluster, follow the steps below:
​
1. Get the Kubeapps URL by running these commands:
   echo "Kubeapps URL: http://127.0.0.1:8080"
   kubectl port-forward --namespace kubeapps service/kubeapps 8080:80
​
2. Open a browser and access Kubeapps using the obtained URL.
```
{% endcode %}

```bash
[root@k8s-master01 ~]# kubectl get pods -n kubeapps
NAME                                                         READY   STATUS    RESTARTS   AGE
apprepo-kubeapps-sync-bitnami-w5jlr-6nvsl                    1/1     Running   0          30s
kubeapps-994b988b5-dlp9r                                     1/1     Running   0          96s
kubeapps-994b988b5-ttxv8                                     1/1     Running   0          96s
kubeapps-internal-apprepository-controller-c78bf86bc-lcpz2   1/1     Running   0          96s
kubeapps-internal-dashboard-6445c69c6b-fgkrs                 1/1     Running   0          96s
kubeapps-internal-dashboard-6445c69c6b-phmxn                 1/1     Running   0          96s
kubeapps-internal-kubeappsapis-c5d8cbb7f-6rnnl               1/1     Running   0          96s
kubeapps-internal-kubeappsapis-c5d8cbb7f-9jthp               1/1     Running   0          96s
kubeapps-internal-kubeops-58794f58c8-bkjtc                   1/1     Running   0          96s
kubeapps-internal-kubeops-58794f58c8-qk655                   1/1     Running   0          96s
kubeapps-postgresql-0                                        1/1     Running   0          96s

[root@k8s-master01 ~]# kubectl get svc -n kubeapps
NAME                             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubeapps                         ClusterIP   10.96.2.39    <none>        80/TCP     2m18s
kubeapps-internal-dashboard      ClusterIP   10.96.1.130   <none>        8080/TCP   2m18s
kubeapps-internal-kubeappsapis   ClusterIP   10.96.2.40    <none>        8080/TCP   2m18s
kubeapps-internal-kubeops        ClusterIP   10.96.3.116   <none>        8080/TCP   2m18s
kubeapps-postgresql              ClusterIP   10.96.0.235   <none>        5432/TCP   2m18s
kubeapps-postgresql-hl           ClusterIP   None          <none>        5432/TCP   2m18s
```

\
\
