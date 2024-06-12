# USER

USER指令设置启动容器的用户(像hadoop需要hadoop用户操作，oracle需要oracle用户操作),可以是用户名或UID

```
USER daemon
USER 1001
```

**注意**：如果设置了容器以daemon用户去运行，那么RUN,CMD和ENTRYPOINT都会以这个用户去运行 镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户

\
