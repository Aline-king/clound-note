# helm作用及核心概念

Helm基于go模板语言，用户只要提供规定的目录结构和模板文件。

在真正部署时Helm模板引擎便可以将其渲染成真正的K8s资源配置文件，并按照正确的顺序将它们部署到节点上。

Helm 定义了一套 Chart 格式来描述一个应用。



{% hint style="info" %}
打个比方，一个安卓程序打包成 APK 格式，就可以安装到任意一台运行安卓系统的手机上，如果我们把 kubernetes 集群比做安卓系统，kubernetes 集群内应用比做安卓程序，那么 Chart 就可以比做 APK。这就意味着，kubernetes 集群应用只要打包成 Chart，就可以通过 Helm 部署到任意一个 kubernetes 集群中。
{% endhint %}

Helm中有三个重要概念，分别为<mark style="color:blue;">**Chart**</mark>、<mark style="color:orange;">**Repository**</mark>和<mark style="color:yellow;">**Release**</mark>。

* Chart代表中Helm包。它包含在K8s集群内部运行应用程序，工具或服务所需的所有资源定义，为所有项目资源清单yaml文件的集合，采用TAR格式，可以类比成yum中的RPM。
* Repository就是用来存放和共享Chart的地方，可以类比成YUM仓库。
* Release是运行在K8s集群中的Chart的实例，一个Chart可以在同一个集群中安装多次。Chart就像流水线中初始化好的模板，Release就是这个“模板”所生产出来的各个产品。

Helm作为K8s的包管理软件，每次安装Charts 到K8s集群时，都会创建一个新的 release。

你可以在Helm 的Repository中寻找需要的Chart。Helm对于部署过程的优化的点在于简化了原先完成配置文件编写后还需使用一串kubectl命令进行的操作、统一管理了部署时的可配置项以及方便了部署完成后的升级和维护。
