# Google Dapper

{% hint style="info" %}
### 分布式跟踪系统的启发来源 <a href="#id-41-fen-bu-shi-gen-zong-xi-tong-de-qi-fa-lai-yuan-80" id="id-41-fen-bu-shi-gen-zong-xi-tong-de-qi-fa-lai-yuan-80"></a>

多数现代分布式跟踪系统都受到Google的Dapper的启发。

* Google在2010年发布了一篇关于Dapper的论文。
* Dapper是Google开发的一种内部工具，用于解决分布式系统中的跟踪问题。
{% endhint %}

**微服务架构中的调用流程**

在微服务架构中，各个模块通过REST、RPC等协议进行调用。一个客户端在前端发起的操作可能需要多个微服务协同工作才能完成。

如图所示，一次请求需要经过多个系统处理才能完成，整个过程形成了一个树形结构的调用链。

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

**这张图片展示了一个用户请求在分布式系统中的完整调用路径，具体过程如下：**

1. **用户请求发起**：用户向系统发出一个请求（RequestX）。
2. **前端处理**：请求首先到达前端服务A（Frontend），A处理完一部分后需要调用其他服务继续处理。
3. **中间层处理**：前端服务A调用中间层的服务B和C（Middle Tier）。
   * **服务A调用服务B（rpc1）。**
   * **服务A调用服务C（rpc2）。**
4. **后端处理**：中间层的服务C进一步调用后端服务D和E（Backend）。
   * **服务C调用服务D（rpc3）。**
   * **服务C调用服务E（rpc4）。**
5. **响应返回**：所有服务处理完毕后，结果返回给前端服务A，最后由A返回给用户（ReplyX）。



## 什么是分布式跟踪系统？

***

想象一下你在点餐，点餐过程包括你下单、厨房准备、打包、送餐等多个步骤。如果想知道你点的餐在每个步骤花了多长时间，这就是分布式跟踪系统的作用。它记录了每个步骤的开始和结束时间，帮你了解整个流程。

Dapper是Google开发的一个内部工具，用来解决这个问题。它可以记录并分析一个请求在多个服务之间的流动路径，帮助快速找出哪里出现了问题。

在现代的微服务架构中，一个请求通常需要多个服务的协作才能完成。比如你在电商网站上提交一个订单，前端服务会处理你的请求，然后调用库存服务确认商品是否有库存，再调用支付服务处理支付，最后调用物流服务安排发货。

树形结构的调用链

分布式跟踪系统使用树形结构来记录每个请求的详细路径。这棵“树”展示了请求在系统中的所有步骤，包括每个服务的调用顺序和父子关系。这种结构有助于我们理解请求在系统中的流动过程，快速找到并解决问题。

由此我们可以看到分布式跟踪系统如何帮助我们详细记录和分析一个请求在多个服务之间的流动路径。Dapper作为一种开创性的工具，为现代的分布式跟踪系统提供了重要的理论基础和实践经验。

## **Dapper跟踪模型中的树形结构、Span和Annotation**

**1. 树形结构和节点**

* **树节点是架构的基本单元**：在Dapper的跟踪模型中，树形结构的每个节点都是系统架构的基本单元。这些节点代表了请求在系统中的不同阶段或步骤。
* **每个节点是对Span的引用**：每个树节点实际上是对一个Span的引用。Span是Dapper用来记录请求操作的基本单位。

**2. Span和它的关系**

* **Span和父Span的关系**：节点之间的连线表示Span和它的父Span之间的关系。这些连线展示了请求在不同服务之间的流动路径和调用关系。
* **Span的定义**：每个Span记录了请求在某个服务中的处理信息，包括操作的开始和结束时间、操作名称等。

**3. Dapper记录的Span信息**

* **Span名称**：Dapper记录每个Span的名称，用于描述这个Span所代表的操作或服务。
* **Span ID和父ID**：每个Span都有一个唯一的ID和一个父ID。父ID表示该Span的上一级操作，从而可以重建不同Span之间的关系。

**4. 跟踪ID**

* **所有Span共享一个跟踪ID**：在一次追踪过程中，所有的Span都挂在一个特定的跟踪上，共享一个跟踪ID。这个跟踪ID用于标识整个请求的生命周期。

Dapper使用树形结构和Span来详细记录请求在系统中的每一步。通过记录每个Span的名称、ID和父ID，以及所有Span共享的跟踪ID，Dapper可以重建请求的完整路径，帮助我们理解和分析系统中的每个操作。这种方式让我们能够清楚地看到请求从开始到结束的每一步，找到并解决可能出现的问题。

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

这张图片展示了一个分布式系统中请求的调用链，以树形结构表示。图片中，每个矩形框代表一个Span，显示了请求在不同服务之间的调用关系。具体内容如下：

1. **Trace ID和Span ID**：
   * **Trace ID**：`d3adb33f`，这是整个请求的唯一标识符。
   * **Span ID**：每个Span都有一个唯一的ID，用于标识该Span。

### Trace的介绍 <a href="#id-42trace-de-jie-shao-122" id="id-42trace-de-jie-shao-122"></a>

***

**什么是Trace？**

* **Trace**：是指某个事务或请求的完整流程，从开始到结束。Trace 记录了这个请求在系统中经过的每一步操作。每个操作在 Trace 中表现为一个 Span。

**Span的定义**

* **Span**：基本工作单元，表示请求中的一次调用。例如：
  * 当 Service A 调用 Service B 时，网络报文来回传输的时长可以用一个 Span 表示。
  * Service B 自身处理请求的时长也可以用一个 Span 表示。这两个 Span 都属于同一个 Trace，因为它们是同一个请求的不同部分。

**Trace的主要元素**

1. **Span**：
   * Span 是 Trace 的基本工作单元。每个 Span 代表一次调用或操作，例如一个 REST 调用或数据库操作。
   * Span 包含开始时间和持续时间等信息。
2. **Trace树**：
   * 一系列相关联的 Span 组成一个树状结构，这个结构被称为 Trace 树。
   * Trace 树展示了请求在系统中流经的所有路径和各个操作之间的关系。
3. **Annotation（标注）**：
   * Annotation 用来及时记录一个事件的存在。
   * 核心的 Annotation 通常用来定义一个请求的开始和结束，比如记录请求到达的时间和处理完成的时间。

**通俗易懂的解释**

**Trace**

* 想象你点了一份外卖，Trace 就是从你下单到收到外卖的全过程。它记录了外卖订单在系统中经历的每一步操作。

**Span**

* **Span** 是 Trace 的基本单元，相当于你订单处理过程中的一个环节。例如：
  * 从你下单到餐厅确认接单，这段时间是一个 Span。
  * 餐厅开始准备食物到准备完毕，这段时间是另一个 Span。
  * 送餐员取餐并送到你家的这段时间又是一个 Span。这些 Span 共同构成了一个完整的 Trace。

### Trace的组成部分 <a href="#id-43trace-de-zu-cheng-bu-fen-134" id="id-43trace-de-zu-cheng-bu-fen-134"></a>

1. **Span**：
   * 每个 Span 记录了一个环节的具体操作和时长。比如，从你下单到餐厅确认接单的时间。
2. **Trace树**：
   * 所有的 Span 组合在一起形成了一个 Trace 树，展示了你订单从开始到结束的整个流程，以及每个环节的先后顺序和关系。
3. **Annotation（标注）**：
   * Annotation 是用来记录关键事件的，比如下单时间、餐厅接单时间、食物准备完毕时间等。它们帮助我们更清晰地了解整个订单的处理过程。

通过 Trace、Span 和 Annotation，我们可以详细地记录和分析一个请求在系统中的每一步操作。这有助于我们了解系统的性能，快速定位并解决问题，确保请求能够顺利完成。

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



{% tabs %}
{% tab title="左侧：Trace 树形结构" %}
* **根节点 A**：
  * 代表一个起始服务（Edge service），例如前端服务或入口服务。每个 Trace 都从一个根节点开始。
  * A 节点有一个唯一的 ID，表示整个 Trace 的上下文。
* **子节点 B 和 E**：
  * 根节点 A 调用了两个子服务 B 和 E。每个子服务也是一个 Span，有自己独立的上下文。
  * 节点 A 传递给 B 和 E 的上下文包含父节点的信息。
* **进一步的子节点 C 和 D**：
  * 服务 B 调用了两个子服务 C 和 D，每个子服务也有自己的上下文。
  * 节点 B 传递给 C 和 D 的上下文包含父节点 B 的信息。
{% endtab %}

{% tab title="右侧：Span 的时间轴" %}
* **Span A**：
  * 横跨时间轴的最长 Span，表示根节点 A 的操作持续时间。
* **Span B**：
  * 从 Span A 的中间开始并结束在 A 之前，表示 B 服务的操作是在 A 服务操作的一部分时间内完成的。
* **Span C 和 D**：
  * 都是 B 服务操作的一部分，表示 C 和 D 的操作是在 B 服务操作的一部分时间内完成的。
* **Span E**：
  * 从 Span A 的中间开始并在 A 之后结束，表示 E 服务的操作也是 A 服务操作的一部分，但其操作结束时间晚于 B。
{% endtab %}
{% endtabs %}

**如何理解这幅图片**

* **Trace 树形结构**：展示了请求在系统中经过的服务节点。每个节点代表一个服务调用，节点之间的箭头表示调用关系和上下文的传递。
* **Span 的时间轴**：展示了每个服务调用在时间上的分布。横向的条表示每个 Span 的持续时间，条的长度表示操作所花的时间。

**通俗易懂的解释**

1. **Trace 和 Span 的概念**：
   * **Trace**：可以想象成一次旅行的路线图，从起点到终点，包括中间经过的每个站点。
   * **Span**：可以想象成旅行中的每一段路程，从一个站点到另一个站点的具体路径。
2. **左侧的 Trace 树**：
   * A 是旅行的起点。
   * 从 A 出发，旅行分成了两条主要路径，分别到达 B 和 E。
   * 从 B 出发，又分成了两条路径，到达 C 和 D。
   * 这就像是一次旅行的详细路线图，展示了每一步是如何进行的。
3. **右侧的 Span 时间轴**：
   * A 是整个旅行的总时间，从出发到结束。
   * B 是旅行的一个部分，在 A 的时间段内进行。
   * C 和 D 是 B 的子部分，在 B 的时间段内进行。
   * E 是另一个与 B 平行的部分，但它比 A 持续时间更长。

### 分布式追踪系统中 Span Context（Span 上下文信息）的传递过程 <a href="#id-44-fen-bu-shi-zhui-zong-xi-tong-zhong-spancontextspan-shang-xia-wen-xin-xi-de-chuan-di-guo-cheng-148" id="id-44-fen-bu-shi-zhui-zong-xi-tong-zhong-spancontextspan-shang-xia-wen-xin-xi-de-chuan-di-guo-cheng-148"></a>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

这张图片展示了分布式追踪系统中 Span Context（Span 上下文信息）的传递过程。

**Span Context**

* **Span Context**：Span 的上下文信息，包括 Trace ID、Span ID 以及需要传递到下游服务的其他内容。
  * **Trace ID**：标识整个请求的唯一标识符。
  * **Span ID**：标识当前操作的唯一标识符。

**Span Context 的传递**

* **传递方式**：
  * 具体实现需要通过某种序列化机制（Wire Protocol）在进程（或服务）边界上传递 Span Context，以将不同进程中的 Span 关联到同一个 Trace 上。这也被称为 Context Propagation。
  * 这些 Wire Protocol 可以基于文本（例如 HTTP header），也可以是二进制协议。

1. **Service A**：
   * **Parent Span**：表示 Service A 中的一个操作或请求。
   * **Child Span**：表示在 Service A 内部的子操作，每个子操作都记录在 Parent Span 内。
   * **Log**：记录操作的具体日志信息。
2. **Network Call**：
   * **Span Context 传播**：Service A 在网络请求中注入跟踪相关的 Header，这些 Header 包含 Trace ID、Span ID 等信息。
   * 这些 Header 被传递到下游服务 Service B。
3. **Service B**：
   * **Child Span**：Service B 读取这些 Header，生成自己的 Span，并将其作为子 Span 记录下来。
   * **Log**：Service B 记录自己的操作日志。

**通俗解释**

**什么是 Span Context？**

* **Span Context**：可以把它看作是一张旅行记录单，包含了旅行的所有信息（Trace ID）和当前旅程的阶段信息（Span ID）。

**Span Context 的传递方式**

* 当你在旅途中换乘时（例如从火车换到公交车），你需要把旅行记录单传递给下一个交通工具的工作人员，这个过程就叫做 Context Propagation。
* 这种传递可以通过文本（例如 HTTP header）或二进制协议来完成。

**图片中的流程**

1. **Service A 的操作**：
   * **Parent Span**：假设你在火车上，火车行驶的过程就是 Parent Span。
   * **Child Span**：你在火车上进行的每个操作（如阅读、休息）就是 Child Span。
   * **Log**：记录你在火车上的每个操作细节。
2. **网络请求**：
   * 当你从火车换乘到公交车时，你把旅行记录单（Span Context）传递给公交车的工作人员。
3. **Service B 的操作**：
   * **Child Span**：公交车上的操作记录，这些操作是火车行程的继续。
   * **Log**：记录你在公交车上的每个操作细节。

通过 Span Context 的传递，我们可以将不同服务中的操作关联起来，构成一个完整的 Trace。这有助于我们在分布式系统中追踪请求的完整路径，分析和解决系统中的问题。
