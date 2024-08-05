# Cephx认证流程

## 身份验证流程

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

client和osd都有一个叫做monclient的模块负责认证和密钥交换。

monitor上有一个AuthMonitor的服务模块负责与monclient对话。

### 认证过程

每个Monitor都可以对客户端进行身份验正并分发密钥，不存在单点故障和性能瓶颈,

Monitor会返回用于身份验正的数据结构，其包含获取Ceph服务时用到的session key（通过客户端密钥进行加密）。

客户端使用session key 向 Monitor 请求所需的服务&#x20;

Monitor 向客户端提供一个 ticket，用于向实际处理数据的OSD等验正客户端身份&#x20;

Monitor和OSD共享同一个secret，因此OSD会信任由MON发放的ticket（存在有效期限）

{% hint style="warning" %}
CephX身份验正功能仅限制Ceph的各组件之间，它不能扩展到其它非Ceph组件；&#x20;

它并不解决数据传输加密的问题；
{% endhint %}

## MDS和OSD验证流程

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

1 当client与Monitor实现基本的认证后，monitor与后端的mds和osd会自动进行认证信息的同步&#x20;

2 当client与Monitor实现基本的通信认证后 可以独立与后端的MDS服务发送请求。 可以独立与后端的OSD服务发送请求
