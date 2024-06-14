# pv与pvc

之前我们提到的Volume可以提供多种类型的资源存储(可持久或不持久)，但是它定义在Pod上的，是属于"资源对象"的一部分。

工作中的存储资源一般都是独立的，这就是资源对象Persistent Volume(PV)，是由管理员设置的存储，它是群集的一部分，PV 是 Volume 之类的卷插件，但具有独立于使用 PV 的 Pod 的生命周期。。



Persistent Volume 跟Volume类似

PV就是Kubernetes集群中的网络存储，不属于Node、Pod等资源，但可以被他们访问。

PV是独立的网络存储资源对象，有自己的生命周期，支持很多种volume类型

Persistent Volume Claim(PVC) 是一个网络存储服务的请求。

PVC和PV与Pod之间的关系。

Pod只能通过PVC到PV上请求一块独立大小的网络存储空间，而PVC 可以动态的根据用户请求去申请PV资源，不仅仅涉及到存储空间，还有对应资源的访问模式。

StorageClass(SC)

管理员可以根据 Kubernetes 的 StorageClass 对象来动态的定义各种类型的资源。
