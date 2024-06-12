# Oracle

### 下载并运行oracle容器

```bash
# docker pull oracleinanutshell/oracle-xe-11g
```

{% hint style="info" %}
说明：&#x20;

49160 为ssh端口           49161 为sqlplus端口         49162 为oem端口

&#x20;                                     oracle数据库连接信息&#x20;

port:49161 sid:xe&#x20;

username:system&#x20;

password:oracle

SYS用户密码为:oracle
{% endhint %}

```bash
# docker run -h oracle --name oracle -d -p 49160:22 -p 49161:1521 -p 49162:8080 oracleinanutshell/oracle-xe-11g
237db949020abf2cee12e3193fa8a34d9dfadaafd9d5604564668d4472abe0b2
```

```
# docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED         STATUS         PORTS                                                                                                                               NAMES
237db949020a   oracleinanutshell/oracle-xe-11g   "/bin/sh -c '/usr/sb…"   7 seconds ago   Up 4 seconds   0.0.0.0:49160->22/tcp, :::49160->22/tcp, 0.0.0.0:49161->1521/tcp, :::49161->1521/tcp, 0.0.0.0:49162->8080/tcp, :::49162->8080/tcp   oracle
```

oracle数据库连接信息

```

sid:xe
username:system
password:oracle
​
SYS用户密码为:oracle

```
