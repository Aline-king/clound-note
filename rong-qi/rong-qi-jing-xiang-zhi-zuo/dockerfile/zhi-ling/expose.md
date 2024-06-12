# EXPOSE

EXPOSE指令

用于指定容器在运行时监听的端口

```
格式:EXPOSE <port> [<port>...]
例:EXPOSE 80 3306 8080
```

上述运行的端口还需要使用docker run运行容器时通过-p参数映射到宿主机的端口.

\
