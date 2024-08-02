# Envoy HTTP路由流量迁移

### 流量迁移实现方法 <a href="#id-51-liu-liang-qian-yi-shi-xian-fang-fa-47" id="id-51-liu-liang-qian-yi-shi-xian-fang-fa-47"></a>

如何在 Envoy 中通过高级路由和 `runtime_fraction` 对象实现流量迁移。通过动态调整流量分配比例，可以实现新旧版本之间的平滑过渡和版本升级。这是流量治理中的一个重要应用，有助于实现灰度发布、A/B测试等流量控制策略。

### **流量迁移概述**：

**目的**：通过动态调整流量分配，将流量逐渐从一个集群迁移到另一个集群，实现平滑的版本过渡。

### 使用方法

**使用 `runtime_fraction` 进行流量迁移**：

* **定义**：在路由配置中使用 `runtime_fraction` 对象，设置流量匹配的概率，从而控制流量逐渐从一个集群迁移到另一个集群。
* **关键参数**：
  * `numerator`：指定流量的分子值。
  * `denominator`：指定流量的分母值，可以是 HUNDRED（100）、TEN THOUSAND（10000）或 MILLION（1000000）。
  * `runtime_key`：指定用于动态调整分子值的运行时键。

**配置示例**：

```yaml
routes:
- match:                          #定义路由匹配参数;
  prefix|path|regex: ...          # 流量过滤条件，三者必须定义其中之一;
  runtime_fraction:               #额外匹配指定的运行时键值，每次评估匹配路径时，它必需低于此字段指示的匹配百分比;支持渐进式修改,这个是流量迁移的关键字
    default_value:                #运行时键值不可用时，则使用此默认值;
      numerator: 0                #指定分子，默认为0;如果此处为0就意味着这里的流量比例为0，将由第二个match匹配到，第二个match没有指定流量比例，只要第一个没有匹配到漏过来都将被第二个处理。
      denominator: HUNDRED        #指定分母，小于分子时，最终百分比为1; 分母可固定使用HUNDRED(默认，为100）、TEN THOUSAND（为10000，即万）和MILLION（为百万）:
    runtime_key: routing.traffic_shift.KEY     # 可以对分子的值随时进行动态调整，指定要使用的运行时键，其值需要用户自定义;
  route:
    cluster: app1_v1
- match:
    prefix|path|regex: ... #此处的匹配条件应该与前一个路由匹配条件相同，以确保能够分制流量;
  route:
    cluster: app1_v2           #此处的集群通常是前一个路由项目中的目标集群应用程序的不同版本;
```

**流量迁移过程**：

**步骤**：

1. 配置两个使用相同匹配条件的路由条目。
2. 在第一个路由条目中配置 `runtime_fraction` 对象，设定其接收的流量比例。
3. 通过 Envoy 的 admin 接口动态修改 `runtime_fraction` 对象的值，逐步调整流量分配，实现流量迁移。

**动态修改示例**：

* **使用 curl 命令动态修改流量分配**：

```
curl -XPOST http://envoy_ip:admin_port/runtime/modify?key1=val1&key2=val2流量迁移实现方法
```

### Envoy流量迁移配置示例 <a href="#id-52envoy-liu-liang-qian-yi-pei-zhi-shi-li-53" id="id-52envoy-liu-liang-qian-yi-pei-zhi-shi-li-53"></a>

以下内容介绍了流量迁移配置示例，特别是通过灰度发布的方式将流量逐步从一个微服务版本迁移到另一个版本。

1. **假设场景**：

* **描述**：假设存在一个微服务应用 `demoapp`，其 1.0 和 1.1 版本分别对应 `demoapp-v1.0` 和 `demoapp-v1.1` 两个集群。初始状态下，`demoapp-v1.0` 承载所有的请求流量。

2. **流量迁移**：
   * **描述**：通过不断调整运行时参数 `routing.traffic_shift.demoapp` 的值，将流量逐步从 `demoapp-v1.0` 迁移到 `demoapp-v1.1` 集群，最终实现完全迁移。
   *   **示例命令**：

       ```powershell
       curl -XPOST 'http://front_envop_ip:admin_port/runtime_modify?routing.traffic_shift.demoapp=90'
       ```
3.  **路由配置示例**：

    ```yaml
    route_config:
      name: local_route
      virtual_hosts:
      - name: demoapp
        domains: ["*"]
        routes:
        - match:
          prefix: "/"
          runtime_fraction:
            default_value:
              numerator: 100
              denominator: HUNDRED
            runtime_key: routing.traffic_shift.demoapp
          route:
            cluster: demoappv10
        - match:
          prefix: "/"
          route:
            cluster: demoappv11
    ```
4. **流量调度方式**：

> 逐步调度称为灰度发布，
>
> 一次性调度称为蓝绿部署。
>
> 这些策略是在路由配置中实现的。

* **注意**：`routing.traffic_shift.demoapp` 是自定义的键，只要不与其它键冲突即可。

5.  **集群配置示例**：

    ```yaml
    clusters:
      - name: demoappv10
        connect_timeout: 0.25s
        type: STRICT_DNS
        lb_policy: ROUND_ROBIN
        load_assignment:
          cluster_name: demoappv10
          endpoints:
          - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: demoappv10
                    port_value: 80
      - name: demoappv11
        connect_timeout: 0.25s
        type: STRICT_DNS
        lb_policy: ROUND_ROBIN
        load_assignment:
          cluster_name: demoappv11
          endpoints:
          - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: demoappv11
                    port_value: 80
    ```

以上内容详细介绍了如何通过 Envoy 实现流量灰度迁移。通过配置 `runtime_fraction` 和动态调整 `routing.traffic_shift` 参数，可以将流量逐步从旧版本迁移到新版本，从而实现平滑过渡。这种方法可以帮助在生产环境中逐步引入新版本，减少风险，并在必要时轻松回滚到旧版本。
