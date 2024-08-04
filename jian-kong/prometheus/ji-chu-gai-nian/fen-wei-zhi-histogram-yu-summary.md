# 分位值（histogram与summary）

{% embed url="https://prometheus.io/docs/practices/histograms/#histograms-and-summaries" %}
官方对于histogram与summary的介绍
{% endembed %}

## histogram与summary的对比

<table data-header-hidden><thead><tr><th width="158">对比点</th><th>histogram</th><th>summary</th></tr></thead><tbody><tr><td>查询表达式对比</td><td><code>histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))</code></td><td><code>http_request_duration_seconds_summary{quantile="0.95"}</code></td></tr><tr><td>所需配置</td><td>选择合适的buckets</td><td>选择所需的φ分位数和滑动窗口。其他φ分位数和滑动窗口以后无法计算。</td></tr><tr><td>客户端性能开销</td><td>开销低，因为它们只需要增加计数器</td><td>开销高，由于流式分位数计算</td></tr><tr><td>服务端性能开销</td><td>开销高，因为需要在服务端实时计算(而且bucket值指标基数高)</td><td>开销低，可以看做是gauge指标上传，仅查询即可</td></tr><tr><td>分位值误差</td><td>随bucket精度变大而变大(线性插值法计算问题)</td><td>误差在φ维度上受可配置值限制</td></tr><tr><td>是否支持聚合</td><td>支持</td><td>不支持(配置sum avg等意义不大)</td></tr><tr><td>是否提供全局分位值</td><td>支持(根据promql匹配维度决定)</td><td>不支持(因为数据在每个实例/pod/agent侧已经算好，无法聚合)</td></tr></tbody></table>



{% tabs %}
{% tab title="histogram" %}
## **histogram数据说明**

```
HELP prometheus_tsdb_compaction_duration_seconds Duration of compaction runs
TYPE prometheus_tsdb_compaction_duration_seconds histogram
prometheus_tsdb_compaction_duration_seconds_bucket{le="1"} 222
prometheus_tsdb_compaction_duration_seconds_bucket{le="2"} 223
prometheus_tsdb_compaction_duration_seconds_bucket{le="4"} 226
prometheus_tsdb_compaction_duration_seconds_bucket{le="8"} 230
prometheus_tsdb_compaction_duration_seconds_bucket{le="16"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="32"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="64"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="128"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="256"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="512"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="1024"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="2048"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="4096"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="8192"} 231
prometheus_tsdb_compaction_duration_seconds_bucket{le="+Inf"} 231
prometheus_tsdb_compaction_duration_seconds_sum 78.46930486000002
prometheus_tsdb_compaction_duration_seconds_count 231
```

<table><thead><tr><th width="161"></th><th>意思</th><th data-hidden></th></tr></thead><tbody><tr><td>xxx_sum </td><td>记录的和，比如这个指标就是tsdb_compaction延迟秒数的和 78秒</td><td></td></tr><tr><td>xxx_count</td><td>代表记录的数量和，就是 一共231次上报</td><td></td></tr><tr><td>xxx_bucket </td><td><p>代表延迟描述小于这个le的记录数为多少个</p><ul><li><code>prometheus_tsdb_compaction_duration_seconds_bucket{le="4"} 226</code> 的意思就是 小于4秒的一共226个（其中包括1和2）</li></ul><p>bucket的最后一定是个+inf的记录，因为算分位值的时候要用到+inf</p><p>一个新的数据上报时，会把大于这个value的 bucket全部+1</p></td><td></td></tr><tr><td></td><td></td><td></td></tr></tbody></table>
{% endtab %}

{% tab title="summary" %}
## **summary数据说明**

```Go
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.000734711
go_gc_duration_seconds{quantile="0.25"} 0.0010731
go_gc_duration_seconds{quantile="0.5"} 0.001139736
go_gc_duration_seconds{quantile="0.75"} 0.00123169
go_gc_duration_seconds{quantile="1"} 0.006106601
go_gc_duration_seconds_sum 16.28009843
go_gc_duration_seconds_count 13959
```
{% endtab %}
{% endtabs %}



<table><thead><tr><th width="144"></th><th></th><th data-hidden></th></tr></thead><tbody><tr><td>xxx_sum </td><td>代表记录的和，比如这个指标就是go_gc消耗秒数的和 为16秒</td><td></td></tr><tr><td>xxx_count </td><td>代表记录的数量和，就是 一共13959次上报</td><td></td></tr><tr><td>xxx{quantile} </td><td><p>代表分位值=quantile的值</p><ul><li><code>go_gc_duration_seconds{quantile="0.75"} 0.00123169</code> 代表就是75分位值为0.00123169秒</li></ul></td><td></td></tr></tbody></table>
