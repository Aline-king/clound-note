# NodePort

在ClusterIP的基础上，以:方式对外提供服务，默认端口范围沿用Docker初期的随机端口范围 30000\~32767，但是NodePort设定的时候，会在集群所有节点上实现相同的端口。