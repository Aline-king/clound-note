# 分位值

### 定义

分位值即把所有的数值从小到大排序，取前N%位置的值，即为该分位的值。

### **分位值的意义是什么？**

一般用分位值来观察大部分用户数据，平均值会“削峰填谷”消减毛刺，同时高分位的稳定性可以忽略掉少量的长尾数据。

{% hint style="warning" %}
不适用

高分位数据不适用于全部的业务场景，例如金融支付行业，可能就会要求100%成功。
{% endhint %}

### 分位值如何计算

获取记录中95分位值

`histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))`

以95分位值为例： 将采集到的100个数据，从小到大排列，95分位值就是取出第95个用户的数据做统计。

