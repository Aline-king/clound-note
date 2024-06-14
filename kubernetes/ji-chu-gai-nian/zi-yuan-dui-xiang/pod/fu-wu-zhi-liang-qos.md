# 服务质量- Qos

QoS是作用在 Pod 上的一个配置，当 Kubernetes 创建一个 Pod 时，它就会给这个 Pod 分配一个 QoS 等级，可以是以下等级之一：

1. Guaranteed等级：&#x20;
   * Pod内的每个容器同时设置了CPU和内存的requests和limits 而且值必须相等。 这样的话pod在运行时候有了稳定的资源保障措施。&#x20;
2. Burstable等级：&#x20;
   * pod至少有一个容器设置了cpu或内存的requests和limits，不满足 Guarantee 等级的要求。 这样的话pod在运行时候虽然有了资源保障措施，但是可能出现意外资源不足的情况。&#x20;
3. BestEffort等级：&#x20;
   * 没有任何一个容器设置了requests或limits的属性。 那就采用 默认的资源配置 - 后期运行可能因为资源限制导致无法运行 不做资源配置措施 - 后期因为资源抢占导致无法运行

依次递增：BestEffort -> Burstable -> Guaranteed

### **不同 Qos 的本质区别**

* 在调度时调度器只会根据 request 值进行调度；
* 二是当系统 OOM上时对于处理不同 OOMScore 的进程表现不同，也就是说当系统 OOM 时，首先会 kill 掉 BestEffort pod 的进程，若系统依然处于 OOM 状态，然后才会 kill 掉 Burstable pod，最后是 Guaranteed pod；

## 例子

配置代表cpu 申请100m，限制1000m。内存申请100Mi ，限制2500Mi

```Go
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 1000m
            memory: 2500Mi
```

`kube-state-metrics` 提供的4个相关指标。

| 指标名称                                                    | 含义                 | 单位说明                    |
| ------------------------------------------------------- | ------------------ | ----------------------- |
| kube\_pod\_container\_resource\_requests\_cpu\_cores    | 容器设置的cpu requests值 | request=100m 代表使用0.1个核心 |
| kube\_pod\_container\_resource\_requests\_memory\_bytes | 容器设置的mem requests值 | 单位：字节                   |
| kube\_pod\_container\_resource\_limits\_cpu\_cores      | 容器设置的cpu limits值   | request=100m 代表使用0.1个核心 |
| kube\_pod\_container\_resource\_limits\_memory\_bytes   | 容器设置的mem limits值   | 单位：字节                   |

{% hint style="warning" %}
### k8s中的**cpu属于可压缩资源**

pod中服务使用CPU超过设置的limits，pod不会被kill掉但会被限制，所以我们应该通过观察容器cpu被限制的情况来考虑是否将cpu的limit调大。
{% endhint %}

#### **cpu限制率和利用率**

| <ol start="1"><li>限制率</li></ol><p><code>container_cpu_cfs_periods_total</code>代表 container生命周期中度过的cpu周期总数<code>container_cpu_cfs_throttled_periods_total</code>代表container生命周期中度过的受限的cpu周期总数所以使用下面的表达式来查出最近5分钟，超过25%的CPU执行周期受到限制的container有哪些。</p><pre class="language-Go"><code class="lang-Go"> 100 * sum by(container_name, pod_name, namespace)(increase(container_cpu_cfs_throttled_periods_total{container_name!=""}[5m])) / sum by(container_name, pod_name, namespace) (increase(container_cpu_cfs_periods_total[5m]))  > 25
</code></pre> |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <ol start="2"><li>利用率</li></ol><p>用下面的计算方式表示容器cpu使用率<code>container_cpu_usage_seconds_total</code> 代表cpu的计数器<code>container_spec_cpu_quota</code>是容器的CPU配额，它的值是容器指定的CPU个数*100000。</p><pre class="language-Go"><code class="lang-Go">sum(rate(container_cpu_usage_seconds_total{image!=""}[1m])) by (container, pod)  / (sum(container_spec_cpu_quota{image!=""}/100000) by (container, pod)  )* 100
</code></pre>                                                                                                                                  |

{% hint style="warning" %}
### **在k8s中mem属于不可压缩资源**

pod之间是无法共享的，完全独占的。所以一旦容器内存使用超过limits，会导致oom，然后重新调度。
{% endhint %}

#### **mem oom 判定依据**

`container_memory_working_set_bytes`是容器真实使用的内存量kubelet通过比较 `container_memory_working_set_bytes`和 `container_spec_memory_limit_bytes` 来决定oom container。同时还有 `container_memory_usage_bytes`用来表示容器使用内存，其中包含了很久没用的缓存，该值比 `container_memory_working_set_bytes`要大所以cpu使用率可以使用下面的公式计算

```Go
(container_memory_working_set_bytes/container_spec_memory_limit_bytes )*100
```

\
