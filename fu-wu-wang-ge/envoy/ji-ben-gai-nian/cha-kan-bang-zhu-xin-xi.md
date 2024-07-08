# 查看帮助信息

这些内容描述的是一个网络服务（很可能是Envoy Proxy）的管理命令集，允许管理员通过HTTP接口对服务进行配置、监控和管理。下面是对每一行命令的简要解释：

```bash
# curl http://172.25.1.2:9901/help
admin commands are:
1. /: 访问管理员主页。
2. /certs: 打印服务器上的证书信息。
3. /clusters: 查看上游集群状态。
4. /config_dump: 导出当前Envoy配置（实验性功能），可选参数包括资源类型(resource)、筛选字段(mask)、名称正则匹配(name_regex)以及是否包含EDS配置(include_eds)。
5. /contention: 导出当前Envoy的锁竞争统计信息（如果启用）。
6. /cpuprofiler (POST): 启用或禁用CPU性能分析器，需指定是否启用(enable)。
7. /drain_listeners (POST): 平滑关闭监听器，可选项包括优雅关闭(graceful)、不退出(skip_exit)、仅关闭入站监听器(inboundonly)。
8. /healthcheck/fail (POST): 使服务器在健康检查中失败。
9. /healthcheck/ok (POST): 使服务器在健康检查中成功。
10. /heap_dump: 导出当前Envoy堆信息（如果支持）。
11. /heapprofiler (POST): 启用或禁用堆性能分析器，需指定是否启用(enable)。
12. /help: 打印所有管理命令列表。
13. /hot_restart_version: 打印热重启兼容版本。
14. /init_dump (experimental): 导出初始化管理器信息，可选参数mask用于指定组件。
15. /listeners: 打印监听器信息，可选参数format指定输出格式。
16. /logging (POST): 查询或更改日志级别，需指定路径(paths)和级别(level)。
17. /memory: 显示当前内存分配和使用情况。
18. /quitquitquit (POST): 关闭服务器。
19. /ready: 检查服务器状态，如果是活动的返回200，否则返回503。
20. /reopen_logs (POST): 重新打开访问日志。
21. /reset_counters (POST): 重置所有计数器为零。
22. /runtime: 打印运行时配置值。
23. /runtime_modify (POST): 添加或修改运行时配置，通过查询参数传递。
24. /server_info: 打印服务器版本和状态信息。
25. /stats: 打印服务器统计信息，支持多种参数如usedonly(只显示重启后更新的统计)、filter(过滤条件)、format(输出格式)、type(统计类型)、histogram_buckets(直方图展示模式)等。
26. /stats/prometheus: 以Prometheus格式打印服务器统计信息，支持类似/stats的参数。
27. /stats/recentlookups: 显示最近的统计名称查找记录。
28. /stats/recentlookups/clear (POST): 清除最近的统计名称查找记录及计数器。
29. /stats/recentlookups/disable (POST): 停止记录新的统计名称查找。
30. /stats/recentlookups/enable (POST): 开始记录新的统计名称查找。
```

这些命令为系统管理员提供了强大的工具来实时监控和调整Envoy实例的行为，从而确保其高效稳定地运行。
