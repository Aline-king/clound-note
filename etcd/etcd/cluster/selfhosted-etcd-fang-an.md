# self-hosted etcd方案

特点：自由简单、但有风险 -- pod变化频率可知。

以容器应用方式部署到Kubernetes集群中，实现Kubernetes对自身依赖组件的管理。需要借助etcd-operator来维护etcd集群，最符合Kubernetes的使用习惯。
