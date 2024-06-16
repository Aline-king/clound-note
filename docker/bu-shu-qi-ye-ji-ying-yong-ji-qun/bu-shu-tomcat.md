# 部署tomcat

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="不暴露端口运行" %}
```
# docker run -d --rm tomcat:9.0
```

```
# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED             STATUS             PORTS                                                  NAMES
c20a0e781246   tomcat:9.0   "catalina.sh run"        27 seconds ago      Up 25 seconds      8080/tcp                                               heuristic_cori
```

\

{% endtab %}

{% tab title="暴露端口运行" %}
```
# docker run -d -p 8080:8080 --rm tomcat:9.0
2fcf5762314373c824928490b871138a01a94abedd7e6814ad5f361d09fbe1de
```

```
# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED             STATUS             PORTS                                                  NAMES
2fcf57623143   tomcat:9.0   "catalina.sh run"        3 seconds ago       Up 1 second        0.0.0.0:8080->8080/tcp, :::8080->8080/tcp              eloquent_chatelet
```

**在宿主机访问**

```
# docker exec 2fc ls /usr/local/tomcat/webapps
里面为空，所以可以添加网站文件。
```

\

{% endtab %}

{% tab title="暴露端口及添加网站文件" %}
```
# docker run -d -p 8081:8080 -v /opt/tomcat-server:/usr/local/tomcat/webapps/ROOT tomcat:9.0
f456e705d48fc603b7243a435f0edd6284558c194e105d87befff2dccddc0b63
```

```
# docker ps
CONTAINER ID   IMAGE        COMMAND             CREATED         STATUS         PORTS                                       NAMES
f456e705d48f   tomcat:9.0   "catalina.sh run"   3 seconds ago   Up 2 seconds   0.0.0.0:8081->8080/tcp, :::8081->8080/tcp   cool_germain
```

```
# echo "tomcat running" > /opt/tomcat-server/index.html
```

**在宿主机访问**
{% endtab %}
{% endtabs %}

