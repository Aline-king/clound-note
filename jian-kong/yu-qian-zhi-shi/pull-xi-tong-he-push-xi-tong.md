# pull系统和push系统

<table data-header-hidden><thead><tr><th width="151"></th><th></th><th></th></tr></thead><tbody><tr><td><strong>采集模型</strong></td><td><strong>原理简介</strong></td><td><strong>代表</strong></td></tr><tr><td>push</td><td>agent定时<strong>推</strong>送数据到server</td><td>夜莺&#x26;open-falcon</td></tr><tr><td>pull</td><td>server定时去agent<strong>拉</strong>数据</td><td><a href="https://y04h4pmzvxx.feishu.cn/wiki/Abj4wq102i3zL8knen5cPMXfnHh">Prometheus</a></td></tr></tbody></table>

### 优缺点

1.  **就采集器是否丰富来说**

    我们对比的是这个系统是否有很好的插件扩展机制，因为这直接决定了开源社区对该系统采集器贡献的活跃度  prometheus采集的pull模型，使用者可以用自定义exporter的模式灵活的接入。
2. **push型的致命缺点 agent和服务端强耦合**
   1. 那就是agent需要配置服务端地址，带来了一定的耦合性，不适合云原生场景。
   2. 如果采用push型，试想一下你的应用部署在k8s中，在启动的时候需要指定监控上报的服务地址，那是不能接受的。
3. **如果push端的服务地址变化了怎么办**
   1. 典型的场景就是在k8s中，pod的扩缩十分频繁，服务端的地址也不固定。

### **pull型的处理方法**

应用pull模型采集的prometheus，可以对接多种服务发现源，特别适合k8s环境。举个例子，应用的pod一旦发生变化，prometheus就可以通过配置好k8s的服务发现模式监听到资源变化，进行采集的增删，agent侧只需要暴露自己的指标，完全不关心是哪一个server过来获取数据。
