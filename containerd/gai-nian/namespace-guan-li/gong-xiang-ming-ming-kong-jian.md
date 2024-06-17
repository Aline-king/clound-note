# 共享命名空间

```bash
# ctr tasks ls
TASK        PID      STATUS
busybox3    13778    RUNNING
busybox     8377     RUNNING
busybox1    12469    RUNNING

# ctr container create --with-ns "pid:/proc/13778/ns/pid" docker.io/library/busybox:latest busybox4
[root@localhost ~]# ctr tasks start -d busybox4 bash
[root@localhost ~]# ctr tasks exec --exec-id $RANDOM -t busybox3 sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
   20 root      0:00 sh
   26 root      0:00 sh
   32 root      0:00 ps aux

```
