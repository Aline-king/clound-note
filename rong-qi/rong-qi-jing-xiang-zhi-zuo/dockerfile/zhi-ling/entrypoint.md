# ENTRYPOINT

ENTRYPOINT与CMD非常类似

相同点： 一个Dockerfile只写一条，如果写了多条，那么只有最后一条生效 都是容器启动时才运行

不同点： 如果用户启动容器时候指定了运行的命令，ENTRYPOINT不会被运行的命令覆盖，而CMD则会被覆盖

```
格式有两种:
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
```

\
