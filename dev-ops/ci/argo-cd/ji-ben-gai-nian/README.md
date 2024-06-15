# 基本概念

{% hint style="info" %}


* Application 应用：[Application应用](app://obsidian.md/Application%E5%BA%94%E7%94%A8)
* Application source type 应用源类型： [用于构建应用的工具类型](app://obsidian.md/%E7%94%A8%E4%BA%8E%E6%9E%84%E5%BB%BA%E5%BA%94%E7%94%A8%E7%9A%84%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%9E%8B)
* Target state 目标状况：执行部署应用时，期望的状态描述。这些描述在 git 中存储着
* Live state 目前的状况：目前应用在平台上运行的状况
* Sync state 同步状况：应用在git中描述的期望的状态与平台上运行的状态是否一致？
* Sync 同步：执行一些动作，让运行的状态和期望的状态相同，如执行apply一些yaml
* Sync operation status：同步的操作的状态，也就是同步的结果，是否成功>
* Refresh 刷新：比较期望的状态和运行的状态
* Health：应用的状态
* Tool 工具：根据文件创建描述的工具，如kustemize。他是应用源类型的一种实现
* Configuration management tool 管理配置工具：包含在工具中
* Configuration management plugin 管理配置插件：自定义的工具
{% endhint %}
