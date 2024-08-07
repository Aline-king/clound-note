# 混沌工程

## 混沌工程 <a href="#id-82-hun-dun-gong-cheng-156" id="id-82-hun-dun-gong-cheng-156"></a>

> 故障注入是混沌工程的一部分，即为混沌工程的子集。

### 引入混沌工程的意义 <a href="#id-821-yin-ru-hun-dun-gong-cheng-de-yi-yi-158" id="id-821-yin-ru-hun-dun-gong-cheng-de-yi-yi-158"></a>

在复杂的分布式服务体系中，故障的随机性和不可预测性大大增加。随着服务化、微服务和持续集成的普及，快速迭代的门槛越来越低，但对复杂系统稳定性的考验却在成倍增长。分布式系统天生包含大量的交互和依赖点，导致故障点层出不穷：硬盘故障、网络故障、流量激增压垮某些组件、外部系统故障、不合理的降级方案等都会成为常见问题。

人力无法改变这种局面，更需要做的是在这些异常被触发之前，尽可能多地识别出系统的脆弱环节或组件，进而有针对性地进行改进，以避免故障发生，打造出更具弹性的系统。这正是混沌工程诞生的原因之一。

混沌工程是一种通过实证探究的方式来理解系统行为的方法，也是一套通过在系统基础设施上进行实验，主动找出系统中的脆弱环节的方法学。它在分布式系统上进行实验，旨在提升系统容错性，建立系统抵御生产环境中不可预知问题的信心。

混沌工程的意义在于能够让复杂系统中根深蒂固的混乱和不稳定性浮出水面，使工程师可以更全面地理解这些系统性固有现象，从而在分布式系统中实现更好的工程设计，不断提高系统弹性。

## 混沌工程发展简介 <a href="#id-822-hun-dun-gong-cheng-fa-zhan-jian-jie-163" id="id-822-hun-dun-gong-cheng-fa-zhan-jian-jie-163"></a>

<table><thead><tr><th width="125"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td><strong>2010年</strong></td><td>Netflix 内部开发了 AWS 云上随机终止 EC2 实例的混沌实验工具：Chaos Monkey。</td><td></td></tr><tr><td><strong>2011年</strong></td><td>Netflix 释出了其猴子军团工具集：Simian Army。</td><td></td></tr><tr><td><strong>2012年</strong></td><td>Netflix 向社区开源了由 Java 构建的 Simian Army，其中包括 Chaos Monkey V1 版本。</td><td></td></tr><tr><td><strong>2014年</strong></td><td><p>Netflix 开始正式公开招聘 Chaos Engineer（混沌工程师）。</p><p>Netflix 提出了故障注入测试（FIT），利用微服务架构的特性，控制混沌实验的爆炸半径。</p></td><td></td></tr><tr><td><strong>2015年</strong></td><td><p>Netflix 释出了 Chaos Kong，模拟 AWS 区域（Region）中断的场景。</p><p>Netflix 和社区正式提出了混沌工程的指导思想——Principles of Chaos Engineering。</p></td><td></td></tr><tr><td><strong>2016年</strong></td><td>Kolton Andrus（前 Netflix 和 Amazon 混沌工程师）创立了 Gremlin，正式将混沌实验工具商用化。</td><td></td></tr><tr><td><strong>2017年</strong></td><td><ul><li>Netflix 开源了由 Golang 重构的 Chaos Monkey V2 版本，必须集成 CD 工具 Spinnaker 来使用。</li><li>Netflix 释出了 ChAP（混沌实验自动平台），可视为应用故障注入测试（EIT）的加强版。</li><li>由 Netflix 前混沌工程师撰写的新书《混沌工程》在网上出版。</li><li>Russell Miles 创立了 ChaosIO 公司，并开源了 chaostoolkit 混沌实验框架。</li></ul></td><td></td></tr></tbody></table>

### 混沌工程的现实功用 <a href="#id-823-hun-dun-gong-cheng-de-xian-shi-gong-yong-178" id="id-823-hun-dun-gong-cheng-de-xian-shi-gong-yong-178"></a>

> 混沌工程对不同角色的意义

**对于架构师**

混沌工程能够验证系统架构的容错能力，尤其是验证面向失败设计的系统，从而确保系统在遇到故障时的稳定性和恢复能力。

**对于开发和运维**

混沌工程提高了故障应急处理的效率，使故障告警、定位和恢复变得更加有效和高效，从而缩短故障修复时间。

**对于测试**

混沌工程弥补了传统测试方法的不足，通过从系统的角度进行测试，降低故障复发率。这与传统测试方法从用户角度进行测试的方式有所不同。

**对于产品和设计**

通过混沌事件观察产品的表现，可以帮助提升客户的使用体验，确保产品在各种异常情况下仍能提供良好的用户体验。

### 混沌工程与故障注入测试的区别 <a href="#id-824-hun-dun-gong-cheng-yu-gu-zhang-zhu-ru-ce-shi-de-qu-bie-188" id="id-824-hun-dun-gong-cheng-yu-gu-zhang-zhu-ru-ce-shi-de-qu-bie-188"></a>

混沌工程、故障注入（EIT）和故障测试在侧重点和工具集上存在一些重叠。

**混沌工程与故障注入测试的区别**

* **混沌工程**：是一种通过实验发现新信息的过程，旨在识别系统中的脆弱环节和潜在问题。
* **故障注入测试**：是一种针对特定条件或变量的测试方法，通常用于验证系统在特定故障条件下的行为。

**测试与实验的区别**

* **测试**：通过产生二元结果（真或假）来判定测试是否通过。测试不能发现系统中未知或尚不明确的认知，它仅对已知的系统属性进行验证。
* **实验**：可以产生新的认知，通常能扩展对复杂系统的理解空间。

**混沌工程的广泛应用**

混沌工程并非仅仅是制造服务中断等故障。它不仅可以建设性、高效地发现问题，还可以应用于非故障类场景，例如：

* 流量激增
* 资源竞争条件
* 拜占庭故障
* 非计划中的或非正常组合的消息处理等

这些场景帮助揭示系统在各种极端和异常情况下的表现，提升系统的整体韧性和稳定性。

#### 8.2.5 混沌工程实验的输入样例 <a href="#id-825-hun-dun-gong-cheng-shi-yan-de-shu-ru-yang-li-199" id="id-825-hun-dun-gong-cheng-shi-yan-de-shu-ru-yang-li-199"></a>

**示例 1: 模拟云服务区域或数据中心故障**

**说明**: 模拟整个云服务区域或数据中心的故障，以测试系统在大规模基础设施故障下的恢复能力。\
**示例**: 假设一个电商平台在AWS云上运行，可以通过模拟一个AWS区域的完全中断，来观察平台是否能迅速切换到其他区域，并继续为用户提供服务。

**示例 2: 删除部分 Kafka Topic**

**说明**: 跨多个实例删除部分 Kafka topic，以重现生产环境中曾经发生的问题。\
**示例**: 某个实时数据处理系统依赖于Kafka传输数据，可以通过删除几个关键的Kafka topic，来测试系统是否能检测到数据丢失并进行相应的补救措施。

**示例 3: 服务间调用延时注入**

**说明**: 在特定时间段内，对部分流量的服务间调用注入特定的延时，以测试系统的响应能力和容错机制。\
**示例**: 在一个微服务架构的应用中，可以在高峰时段对订单服务和支付服务之间的调用注入延时，来测试系统是否能够处理延迟并避免用户体验的显著下降。

**示例 4: 方法级别的混乱（运行时注入）**

**说明**: 在运行时让方法随机抛出各种异常，以测试系统对异常的处理能力。\
**示例**: 在一个银行系统中，可以让转账方法随机抛出网络异常、数据库连接异常等，来观察系统是否能正确处理异常并保证资金安全。

**示例 5: 代码中插入故障注入指令**

**说明**: 在代码中插入一些指令，使得在这些指令之前可以运行故障注入。\
**示例**: 在一个库存管理系统的代码中插入指令，使得在特定的库存更新操作前可以模拟数据库连接中断，来测试系统能否及时重试或回滚操作。

**示例 6: 强制系统节点时间不同步**

**说明**: 强制系统中各个节点的时间不同步，以测试系统在时间不同步情况下的表现。\
**示例**: 在一个分布式数据库系统中，可以让各节点的时间不同步，来测试数据库的时间戳机制和一致性算法的有效性。

**示例 7: 模拟 I/O 错误**

**说明**: 在驱动程序中执行模拟 I/O 错误的程序，以测试系统对硬件故障的应对能力。\
**示例**: 在一个文件存储系统中，可以在驱动程序中模拟磁盘I/O错误，来观察系统是否能够及时发现错误并进行故障转移或数据恢复。

**示例 8: 让某个 Elasticsearch 集群 CPU 超负荷**

**说明**: 让某个 Elasticsearch 集群的 CPU 超负荷，以测试系统在资源紧张情况下的表现。\
**示例**: 在一个日志分析系统中，可以让 Elasticsearch 集群的CPU使用率达到100%，来测试系统是否能够在CPU超负荷的情况下继续处理查询请求并维持性能。
