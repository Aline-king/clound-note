# 安全框架

k8s应用访问流程有两种

1. 无需apiserver认证 &#x20;
   * 用户 -- ingress|service -- pod （也可以在网络策略上设置是否认证）
2. 需要认证
   * 管理k8s平台上各种应用对象

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## 安全框架运行流程

对于apiserver访问，需要进过三步，认证 授权 准入控制，前两个可以使用插件来实现，可以最大化的用户自定义效果。

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<table data-card-size="large" data-column-title-hidden data-view="cards" data-full-width="false"><thead><tr><th></th><th data-hidden></th><th data-hidden></th></tr></thead><tbody><tr><td><p>图中左边：</p><p>对于k8s来说，它主要面对两种用户： </p><ul><li><p> User Account </p><ul><li>普通人类用户(交互模式)</li></ul></li><li><p>Service Account</p><ul><li>集群内部的pod用户(服务进程)</li></ul></li></ul></td><td></td><td></td></tr><tr><td><p>图中间</p><p>每个部分都是以插件形式整合到 k8s 环境中： </p><ul><li><mark style="color:blue;"><strong>认证</strong></mark>和<mark style="color:purple;"><strong>授权</strong></mark>是按照 插件的顺序进行依次性的检测，并且遵循 "<a data-footnote-ref href="#user-content-fn-1">短路模型</a>" 的认证方式</li><li><p>准入控制 由于仅用户写操作场景，所以它不遵循 "短路模型"，而是会全部检测 </p><ul><li>原因在于，它要记录为什么不执行，原因是什么，方便后续审计等动作。</li></ul></li></ul></td><td></td><td></td></tr></tbody></table>

<table data-card-size="large" data-view="cards"><thead><tr><th></th><th></th><th data-hidden></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><mark style="color:purple;"><strong>认证</strong></mark> </td><td>验证你是谁，确认“你是不是你"，只允许被当前系统许可的人进入集群内部</td><td></td><td><a href="ren-zheng/">ren-zheng</a></td><td><a href="../../../.gitbook/assets/8f7c93de9ade4f62a143a262d58fbc0a.png">8f7c93de9ade4f62a143a262d58fbc0a.png</a></td></tr><tr><td><mark style="color:purple;"><strong>授权</strong></mark></td><td>确认“你是不是有权利做这件事”，</td><td></td><td><a href="shou-quan/">shou-quan</a></td><td><a href="../../../.gitbook/assets/fmmsq383f7jm2h5axarr.png">fmmsq383f7jm2h5axarr.png</a></td></tr><tr><td></td><td>类似于审计，主要侧重于 操作动作的校验、语法规范的矫正等写操作场景。</td><td></td><td><a href="zhun-ru-kong-zhi.md">zhun-ru-kong-zhi.md</a></td><td></td></tr></tbody></table>

[^1]: 失败的时候，到此结束,后续的规则不会进行
