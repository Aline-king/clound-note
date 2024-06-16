# 认证

### 用户组&#x20;

单个控制用户的权限太繁琐，可以借助于用户组的机制实现批量用户的自动化管理，主要有以下几种

<table data-header-hidden><thead><tr><th width="222"></th><th></th></tr></thead><tbody><tr><td>system:unauthenticated </td><td>未能通过任何一个授权插件检验的账号的所有未通过认证测试的用户统一隶属的用户组； </td></tr><tr><td>system:authenticated </td><td>认证成功后的用户自动加入的一个专用组，用于快捷引用所有正常通过认证的用户账号； </td></tr><tr><td>system:serviceaccounts </td><td>所有名称空间中的所有ServiceAccount对象 </td></tr><tr><td>system:serviceaccounts: </td><td>特定名称空间内所有的ServiceAccount对象</td></tr></tbody></table>

### 认证方式

&#x20;证书认证 - 本质上就是TLS双向认证&#x20;

令牌认证 - 大量手动配置TLS认证比较麻烦，可以将证书生成token进行间接使用。











