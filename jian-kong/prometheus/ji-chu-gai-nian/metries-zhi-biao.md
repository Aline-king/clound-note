# Metries 指标

## **指标是什么？**

通俗来说，指标是**数值型的测量结果**。

时间序列意味着随着时间的推移记录变化。不同的应用程序需要测量的内容各不相同。对于一个网页服务器来说，可能是请求的响应时间；对于一个数据库来说，可能是活动连接数或活动查询数等。

指标在理解应用程序为什么以某种方式工作中起着重要作用。假设你在运行一个网络应用程序，并发现应用程序很慢。你需要一些信息来找出应用程序出了什么问题。例如，当请求的数量很大时，应用程序可能变慢。如果你有请求计数的指标，你可以找出原因，并增加服务器数量来处理负载。



{% hint style="info" %}
**四大指标**Google的Google SRE Books一书中提出了系统监控的四个黄金指标

* Latency：延时
* Utilization：使用率
* Saturation：饱和度
* Errors：错误数或错误率
{% endhint %}

**为什么是这四个？**

这四个黄金指标在在任何系统中都是很好的性能状态指标他们之所以被称为”黄金“指标，很大一个因素是因为他们反映了终端用户的感知。因此任何监控系统都会提供被监控对象的这些指标或其变形，并在此基础上辅助

### **两种系统分类**

1. 资源提供系统 ： 对外提供简单的资源，比如CPU（计算资源），存储，网络带宽
   1. Utilization ：往往体现为资源使用的百分比
   2.  Saturation ： 资源使用的饱和度或过载程度，过载的系统往往意味着系统需要辅助的排队系统完成相关任务

       > 以CPU为例，Utilization往往是CPU的使用百分比，Saturation则是当前等待调度CPU的线程或进程队列长度



       Errors : 这个可能是使用资源的出错率或出错数量，比如网络的丢包率或误码率等等
2. 服务提供系统 ： 对外提供更高层次与业务相关的任务处理能力，比如订票，购物等等
   1. Rate ： 单位时间内完成服务请求的能力
   2. Errors ： 错误率或错误数量：单位时间内服务出错的比列或数量
   3. Duration ： 平均单次服务的持续时长（或用户得到服务响应的时延）



{% tabs %}
{% tab title="延时" %}
`histogram_quantile(0.99, sum(rate(apiserver_request_duration_seconds_bucket{job="kubernetes-apiservers"}[5m])) by (verb, le))`

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption><p>可以得到各个http的请求方法的99分位延迟值。 如果99分位延迟值很高，可能是apiserver处理能力达到上限，可以考虑扩容一下。</p></figcaption></figure>


{% endtab %}

{% tab title="饱和度" %}
<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

对于饱和度可以查看apiserver请求队列的情况，如`apiserver_current_inqueue_requests`很大的话，说明排队严重。
{% endtab %}
{% endtabs %}
