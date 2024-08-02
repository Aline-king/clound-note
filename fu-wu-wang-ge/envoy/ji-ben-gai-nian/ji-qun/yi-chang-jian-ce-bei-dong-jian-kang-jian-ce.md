# 异常检测（被动健康检测）

## 一、 被动健康检测（异常点探测） <a href="#yi-bei-dong-jian-kang-jian-ce-yi-chang-dian-tan-ce-1" id="yi-bei-dong-jian-kang-jian-ce-yi-chang-dian-tan-ce-1"></a>

在Envoy中，被动健康检查（Passive Health Checking）是一种基于实时流量监控的机制，用于动态检测上游集群节点的健康状态。与主动健康检查不同，被动健康检查不需要定期发送健康检查请求，而是通过监控实际流量中的错误和超时来判断节点的健康状态。

### 1.1 被动健康检查机制 <a href="#id-11-bei-dong-jian-kang-jian-cha-ji-zhi-3" id="id-11-bei-dong-jian-kang-jian-cha-ji-zhi-3"></a>

被动健康检查主要依赖于异常检测（Outlier Detection）机制。当Envoy检测到某个上游节点的错误率或响应时间超过设定的阈值时，会将该节点标记为不健康，并在一段时间内停止向其发送流量。

### 1.2 关键配置项 <a href="#id-12-guan-jian-pei-zhi-xiang-5" id="id-12-guan-jian-pei-zhi-xiang-5"></a>

以下是被动健康检查的主要配置项：

* **consecutive\_5xx**: 连续出现指定次数的5xx错误后，将节点标记为不健康。
* **consecutive\_gateway\_failure**: 连续出现指定次数的网关错误（如502, 503, 504）后，将节点标记为不健康。
* **interval**: 检查节点健康状态的时间间隔。
* **base\_ejection\_time**: 节点被驱逐后的初始驱逐时间。
* **max\_ejection\_percent**: 在任何给定时间点，最多有百分之几的节点可以被驱逐。
* **enforcing\_consecutive\_5xx**: 启用或禁用连续5xx错误驱逐的权重（默认为100，表示完全启用）。
* **enforcing\_consecutive\_gateway\_failure**: 启用或禁用连续网关错误驱逐的权重（默认为0，表示禁用）。
* **enforcing\_success\_rate**: 启用或禁用基于成功率的驱逐权重（默认为0，表示禁用）。
* **success\_rate\_minimum\_hosts**: 计算成功率时所需的最小主机数。
* **success\_rate\_request\_volume**: 计算成功率时所需的最小请求数。
* **success\_rate\_stdev\_factor**: 节点的成功率与平均成功率的标准差因子，用于驱逐决定。

### 1.3 示例配置 <a href="#id-13-shi-li-pei-zhi-8" id="id-13-shi-li-pei-zhi-8"></a>

以下是一个包含被动健康检查配置的Envoy集群示例：

```yaml
static_resources:
  clusters:
  - name: service_with_passive_health_check
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_with_passive_health_check
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service.example.com
                port_value: 80
    outlier_detection:
      consecutive_5xx: 5
      interval: 10s
      base_ejection_time: 30s
      max_ejection_percent: 50
      enforcing_consecutive_5xx: 100
      enforcing_success_rate: 100
      success_rate_minimum_hosts: 3
      success_rate_request_volume: 100
      success_rate_stdev_factor: 1900
```

**配置项解释**

* **consecutive\_5xx: 5**: 如果一个节点连续出现5次5xx错误，将其标记为不健康。
* **interval: 10s**: 每10秒检查一次节点的健康状态。
* **base\_ejection\_time: 30s**: 节点被标记为不健康后，最少驱逐30秒。
* **max\_ejection\_percent: 50**: 最多可以驱逐50%的节点。
* **enforcing\_consecutive\_5xx: 100**: 完全启用连续5xx错误驱逐机制。
* **enforcing\_success\_rate: 100**: 完全启用基于成功率的驱逐机制。
* **success\_rate\_minimum\_hosts: 3**: 至少需要3个主机来计算成功率。
* **success\_rate\_request\_volume: 100**: 至少需要100个请求来计算成功率。
* **success\_rate\_stdev\_factor: 1900**: 使用成功率标准差因子1900来决定是否驱逐。

### 1.4 工作原理 <a href="#id-14-gong-zuo-yuan-li-13" id="id-14-gong-zuo-yuan-li-13"></a>

1. **监控流量**：Envoy实时监控每个上游节点的请求和响应。
2. **错误计数**：记录每个节点的错误响应（如5xx错误）。
3. **驱逐决策**：当节点的错误率或成功率超出设定的阈值时，Envoy将该节点标记为不健康并驱逐一段时间。
4. **恢复检查**：在驱逐期结束后，Envoy会重新评估该节点的健康状态，决定是否恢复流量。

### 1.5 监控和调试 <a href="#id-15-jian-kong-he-tiao-shi-15" id="id-15-jian-kong-he-tiao-shi-15"></a>

被动健康检查的结果可以通过Envoy的统计接口查看，例如：

* **`cluster.<cluster_name>.outlier_detection.ejections_active`**: 当前被驱逐的节点数。
* **`cluster.<cluster_name>.outlier_detection.ejections_consecutive_5xx`**: 因连续5xx错误被驱逐的节点数。
* **`cluster.<cluster_name>.outlier_detection.ejections_success_rate`**: 因成功率低被驱逐的节点数。

这些统计数据可以通过访问Envoy的admin接口获取，例如 `http://localhost:9901/stats`。

## 二、 关于成功率标准差因子的解释 <a href="#er-guan-yu-cheng-gonglbiao-zhun-cha-yin-zi-de-jie-shi-19" id="er-guan-yu-cheng-gonglbiao-zhun-cha-yin-zi-de-jie-shi-19"></a>

在Envoy的异常点探测（Outlier Detection）配置中，\*\*成功率标准差因子（success\_rate\_stdev\_factor）\*\*是一个用于决定节点是否应被驱逐的重要参数。理解这一点需要对标准差因子的计算和其在驱逐决策中的应用有一定的认识。

### 2.1 成功率标准差因子 <a href="#id-21-cheng-gonglbiao-zhun-cha-yin-zi-21" id="id-21-cheng-gonglbiao-zhun-cha-yin-zi-21"></a>

成功率标准差因子（success\_rate\_stdev\_factor）用于衡量某个节点的成功率与整个集群的平均成功率之间的偏差。如果某个节点的成功率与平均成功率之间的偏差超过设定的标准差因子，则该节点可能会被标记为异常并驱逐。

**配置示例**

```yaml
outlier_detection:
  consecutive_5xx: 5
  interval: 10s
  base_ejection_time: 30s
  max_ejection_percent: 50
  enforcing_consecutive_5xx: 100
  enforcing_success_rate: 100
  success_rate_minimum_hosts: 3
  success_rate_request_volume: 100
  success_rate_stdev_factor: 1900
```

**配置项解释**

* **success\_rate\_stdev\_factor: 1900**：表示使用1900作为标准差因子来决定节点是否应该被驱逐。

### 2.2 标准差因子的计算 <a href="#id-22-biao-zhun-cha-yin-zi-de-ji-suan-27" id="id-22-biao-zhun-cha-yin-zi-de-ji-suan-27"></a>

#### 2.2.1 计算思路 <a href="#id-221-ji-suan-si-lu-28" id="id-221-ji-suan-si-lu-28"></a>

1. **计算集群的平均成功率**：\
   集群中的每个节点都有一个成功率，这些成功率的平均值构成了集群的平均成功率。
2. **计算成功率的标准差**：\
   使用节点成功率计算标准差，表示节点成功率与集群平均成功率之间的波动范围。
3. **判断驱逐条件**：\
   如果某个节点的成功率与平均成功率的偏差超过了设定的标准差因子，则该节点会被标记为异常并驱逐。

#### 2.2.2 计算具体步骤 <a href="#id-222-ji-suan-ju-ti-bu-zhou-30" id="id-222-ji-suan-ju-ti-bu-zhou-30"></a>

1. **收集数据**：
   * 收集所有节点的成功率数据。
   * 计算集群的平均成功率。
2. **计算标准差**：
   * 计算每个节点成功率与平均成功率的差值。
   * 计算这些差值的平方和的平均值，然后取平方根，得到标准差。
3. **应用标准差因子**：
   * 设定的标准差因子如1900，表示需要将标准差乘以19（1900 / 100）。
   * 如果某个节点的成功率低于（平均成功率 - 19 \* 标准差），则该节点会被驱逐。

#### 2.2.3 示例计算 <a href="#id-223-shi-li-ji-suan-32" id="id-223-shi-li-ji-suan-32"></a>

假设集群中有以下节点成功率：

* 节点A：98%
* 节点B：95%
* 节点C：99%
* 节点D：60%

计算平均成功率：\
\[ \text{平均成功率} = \frac{98 + 95 + 99 + 60}{4} = 88% ]

计算标准差：\
\[ \text{标准差} = \sqrt{\frac{(98-88)^2 + (95-88)^2 + (99-88)^2 + (60-88)^2}{4\}} = \sqrt{\frac{100 + 49 + 121 + 784}{4\}} = \sqrt{263.5} \approx 16.23% ]

应用标准差因子1900（即19倍）：\
\[ \text{阈值} = 88 - 19 \times 16.23 = 88 - 308.37 \approx -220.37% ]

显然，这个阈值对于实际情况不适用。为了更实际的应用，通常success\_rate\_stdev\_factor设置为一个较低的值。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719841095052/99be5af34db84f8f88060c28dbd7d6e3.png)

在实际配置中，success\_rate\_stdev\_factor可能需要调试和调整，以适应不同的服务特性和需求。1900是一个较大的标准差因子，通常用于要求非常严格的驱逐策略。在生产环境中，可能需要根据实际观察到的成功率分布情况，选择一个合适的因子值，以确保合理的驱逐行为。

#### 2.2.4 实际应用 <a href="#id-224-shi-ji-ying-yong-41" id="id-224-shi-ji-ying-yong-41"></a>

假设我们将成功率标准差因子（success\_rate\_stdev\_factor）设置为200，这是一个较低的值。在这个例子中，我们将通过以下步骤进行计算。

**配置示例**

```yaml
outlier_detection:
  consecutive_5xx: 5
  interval: 10s
  base_ejection_time: 30s
  max_ejection_percent: 50
  enforcing_consecutive_5xx: 100
  enforcing_success_rate: 100
  success_rate_minimum_hosts: 3
  success_rate_request_volume: 100
  success_rate_stdev_factor: 200
```

**示例数据**

假设集群中有以下节点成功率：

* 节点A：98%
* 节点B：95%
* 节点C：99%
* 节点D：60%

**计算步骤**

1. **计算平均成功率**：\
   \[ \text{平均成功率} = \frac{98 + 95 + 99 + 60}{4} = 88% ]
2. **计算标准差**：\
   \[ \text{标准差} = \sqrt{\frac{(98-88)^2 + (95-88)^2 + (99-88)^2 + (60-88)^2}{4\}} = \sqrt{\frac{100 + 49 + 121 + 784}{4\}} = \sqrt{263.5} \approx 16.23% ]
3. **应用成功率标准差因子**：
   * 成功率标准差因子为200（即2倍）。
   * 计算驱逐阈值：\[ \text{驱逐阈值} = \text{平均成功率} - 2 \times \text{标准差} ]
   * \[ \text{驱逐阈值} = 88 - 2 \times 16.23 = 88 - 32.46 = 55.54% ]

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719841095052/98e6f2e011f64d31a0f8a7cd4a89f607.png)

**解释**

在这个设置下，如果某个节点的成功率低于55.54%，则该节点会被驱逐。我们可以看到，在此配置下：

* 节点A：98%（不驱逐）
* 节点B：95%（不驱逐）
* 节点C：99%（不驱逐）
* 节点D：60%（不驱逐）

根据计算结果，节点D的成功率虽然显著低于其他节点，但它仍高于驱逐阈值55.54%，所以不会被驱逐。

#### 2.2.5 示例配置的调整与应用 <a href="#id-225-shi-li-pei-zhi-de-tiao-zheng-yu-ying-yong-55" id="id-225-shi-li-pei-zhi-de-tiao-zheng-yu-ying-yong-55"></a>

根据上述计算，如果希望对异常节点采取更严格的措施，可以进一步调低成功率标准差因子，例如100（即1倍标准差）。这会将驱逐阈值设置得更高，从而更容易驱逐异常节点。

**调整后的配置示例**

```yaml
outlier_detection:
  consecutive_5xx: 5
  interval: 10s
  base_ejection_time: 30s
  max_ejection_percent: 50
  enforcing_consecutive_5xx: 100
  enforcing_success_rate: 100
  success_rate_minimum_hosts: 3
  success_rate_request_volume: 100
  success_rate_stdev_factor: 100
```

**调整后的计算**

1. **应用成功率标准差因子**：
   * 成功率标准差因子为100（即1倍）。
   * 计算驱逐阈值：\[ \text{驱逐阈值} = \text{平均成功率} - 1 \times \text{标准差} ]
   * \[ \text{驱逐阈值} = 88 - 16.23 = 71.77% ]

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719841095052/0f348a405aec4e68a35d2048cbf9eb19.png)

在此配置下：

* 节点A：98%（不驱逐）
* 节点B：95%（不驱逐）
* 节点C：99%（不驱逐）
* 节点D：60%（驱逐）

根据新的计算结果，节点D的成功率低于71.77%，因此会被驱逐。

使用成功率标准差因子来决定驱逐是一种精细化的异常检测策略，通过比较节点成功率与集群平均成功率的偏差，可以动态识别并隔离表现异常的节点，从而提高服务的可靠性和稳定性。

被动健康检查是一种有效的机制，能够基于实际流量中的错误和性能指标动态调整上游节点的健康状态。通过与主动健康检查相结合，Envoy可以确保服务的高可用性和稳定性，提供更好的负载均衡和故障处理能力。
