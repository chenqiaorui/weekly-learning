### DDOS介绍
分布式拒绝攻击，大量请求攻击服务器，使服务器资源耗尽。

类型：

* DNS攻击，构造大量域名攻击服务器，使dns服务器迭代查询，刷新缓存，响应慢。

* 构造大量SYN请求，消耗tcp连接。单ip的半开连接容量数查询：`sysctl net.ipv4.tcp_max_syn_backlog`，修改`sysctl -w net.ipv4.tcp_max_syn_backlog=1024`。半开连接失败内核会重试，次数为5，修改次数：`sysctl -w net.ipv4.tcp_synack_retries=1`。

* SYN攻击可以开启SYN Cookies防御：`sysctl -w net.ipv4.tcp_syncookies=1`，持久化保存至：`/etc/sysctl.conf`