# 快照管理

RBD支持image快照技术，借助于快照可以保留image的状态历史。

Ceph还支持快照“分层”机制，从而可实现快速克隆VM映像。

## 常见命令

创建快照

rbd snap create \[--pool ] --image  --snap 或者 rbd snap create \[/]@
