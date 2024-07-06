# 配置概念

Bootstrap配置中几个重要的基础概念

```
{
"node": "{...}",              // 节点标识，以呈现给管理服务器并且例如用于标识目的
"static_resources": "{...}",  // 静态配置的资源，用于配置静态的listener、cluster和secret
"dynamic_resources": "{...}", // 动态配置的资源，用于基于xDS API获取listener、cluster和secret配置的lds_config、cds_config和ads_config
"cluster_manager": "{...}",
"hds_config": "{...}",
"flags_path": "...",
"stats_sinks": [],           // 统计信息接收器
"stats_config": "{...}",
"stats_flush_interval": "{...}",
"stats_flush_on_admin": "...",
"watchdog": "{...}",
"watchdogs": "{...}",
"tracing": "{...}",              // 分布式跟踪
"layered_runtime": "{...}",    // 层级化的运行时，支持使用RTDS从管理服务器动态加载
"admin": "{...}",              // Envoy内置的管理接口
"overload_manager": "{...}", // 过载管理器
"enable_dispatcher_stats": "...",
"header_prefix": "...",       // 使用HDS从管理服务器加载上游主机健康状态检测相关的配置
"stats_server_version_override": "{...}",
"use_tcp_for_dns_lookups": "...",
"bootstrap_extensions": [],
"fatal_actions": [],
"default_socket_interface": "..."
}
```
