# emptyDir

一个emptyDir volume在pod被调度到某个Node时候自动创建的，无需指定宿主机上对应的目录。

特点如下：

&#x20;1、跟随Pod初始化而来，开始是空数据卷&#x20;

2、Pod移除，emptyDir中数据随之永久消除&#x20;

3、emptyDir一般做本地临时存储空间使用&#x20;

4、emptyDir数据卷介质种类跟当前主机的磁盘一样。

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
