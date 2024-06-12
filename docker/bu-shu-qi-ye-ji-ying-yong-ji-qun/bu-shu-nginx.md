# 部署Nginx

拉取镜像

docker pull nginx

{% tabs %}
{% tab title="不在docker host暴露端口" %}
```bash
# docker run -d --name nginx-server -v /opt/nginx-server:/usr/share/nginx/html:ro nginx
```

```bash
# docker ps
```

```
# docker inspect 664 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
```

```bash
# curl http://172.17.0.3
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.6</center>
</body>
</html>
```

```
# ls /opt
nginx-server
# echo "nginx is working" > /opt/nginx-server/index.html
```

```
# curl http://172.17.0.3
nginx is working
```
{% endtab %}

{% tab title="在docker host暴露80端口" %}
```
# docker run -d -p 80:80 --name nginx-server-port -v /opt/nginx-server-port:/usr/share/nginx/html:ro nginx
```

```
# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED             STATUS             PORTS                                                  NAMES
74dddf51983d   nginx       "/docker-entrypoint.…"   3 seconds ago       Up 2 seconds       0.0.0.0:80->80/tcp, :::80->80/tcp                      nginx-server-port
```

```
# ls /opt
nginx-server  nginx-server-port
```

```
# echo "nginx is running" > /opt/nginx-server-port/index.html
```

**在宿主机上访问**

```bash
# docker top nginx-server-port
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                22195               22163               0                   15:08               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 22387               22195               0                   15:08               ?                   00:00:00            nginx: worker process

```
{% endtab %}

{% tab title="挂载配置文件使用nginx" %}
配置文件复制出来修改后使用。

```
# docker cp nginxwebcontainername:/etc/nginx/nginx.conf /opt/nginxcon/
修改后即可使用
```

```
# ls /opt/nginxcon/nginx.conf
/opt/nginxcon/nginx.conf
```

```
# docker run -d \
-p 82:80 --name nginx-server-conf \
-v /opt/nginx-server-conf:/usr/share/nginx/html:ro \
-v /opt/nginxcon/nginx.conf:/etc/nginx/nginx.conf:ro \
nginx
76251ec44e5049445399303944fc96eb8161ccb49e27b673b99cb2492009523c
```

```
# docker top nginx-server-conf
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                25005               24972               0                   15:38               ?                   00:00:00            nginx: master process nginx -g daemon off;
101                 25178               25005               0                   15:38               ?                   00:00:00            nginx: worker process
101                 25179               25005               0                   15:38               ?                   00:00:00            nginx: worker process
```

\

{% endtab %}
{% endtabs %}
