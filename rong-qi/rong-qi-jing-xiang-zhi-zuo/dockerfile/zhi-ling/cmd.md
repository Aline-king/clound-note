# CMD

{% hint style="info" %}
**CMD与RUN的不同**

CMD用于指定在<mark style="color:yellow;">容器启动时</mark>所要执行的命令,

RUN用于指定<mark style="color:green;">镜像构建时</mark>所要执行的命令。
{% endhint %}

{% code title="格式有三种" %}
```bash
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
```
{% endcode %}

每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉CMD指定的命令。

```bash
什么是启动容器时指定运行的命令?
# docker run -d -p 80:80 镜像名 运行的命令
```

\
