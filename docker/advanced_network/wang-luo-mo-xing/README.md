# 网络模型



<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="bridge 桥接网络" %}
\--network bridge

桥接容器，除了有一块本地回环接口(Loopback interface)外，还有一块私有接口(Private interface)通过容器虚拟接口(Container virtual interface)连接到桥接虚拟接口(Docker bridge virtual interface)，之后通过逻辑主机接口(Logical host interface)连接到主机物理网络(Physical network interface)。\
桥接网卡默认会分配到172.17.0.0/16的IP地址段。\
如果我们在创建容器时没有指定网络模型，默认就是(Nat)桥接网络，这也就是为什么我们在登录到一个容器后，发现IP地址段都在172.17.0.0/16网段的原因。
{% endtab %}

{% tab title="host 开放式容器" %}
\--network host

比联盟式网络更开放，我们知道联盟式网络是多个容器共享网络(Net),而开放式容器(Open contaner)就直接共享了宿主机的名称空间。因此物理网卡有多少个，那么该容器就能看到多少网卡信息。我们可以说Open container是联盟式容器的衍生。
{% endtab %}

{% tab title="none 封闭式网络" %}
\--network none

封闭式容器，只有本地回环接口(Loopback interface，和咱们服务器看到的lo接口类似)，无法与外界进行通信。不为容器配置任何网络功能，没有网络 --net=none
{% endtab %}

{% tab title="container 联盟式网络" %}
每个容器都各有一部分名称空间(Mount,PID,User)，另外一部分名称空间是共享的(UTS,Net,IPC)。\
由于它们的网络是共享的，因此各个容器可以通过本地回环接口(Loopback interface)进行通信。\
除了共享同一组本地回环接口(Loopback interface)外，还有一块一块私有接口(Private interface)通过联合容器虚拟接口(Joined container virtual interface)连接到桥接虚拟接口(Docker bridge virtual interface)，之后通过逻辑主机接口(Logical host interface)连接到主机物理网络(Physical network interface)。
{% endtab %}
{% endtabs %}
