# username

用于隔离容器中 UID、GID 以及根目录等。

User Namespace 使用了 CLONE\_NEWUSER 的参数，可配置映射宿主机和容器中的 UID、GID。

UID 在当前的 Namespace 下，是具有容器 root 权限的，不是宿主机的。
