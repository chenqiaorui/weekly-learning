##### linux内核防火墙工具 
centos7默认是firewalld，但iptables更为好用。iptables 默认所有服务是开启的，firewalld则相反。

###### 安装iptables
yum install -y iptables-services

###### 服务管理
systemctl status iptables

##### 命令
基本选项说明：

参数	作用
-P	设置默认策略:iptables -P INPUT (DROP
-F	清空规则链
-L	查看规则链
-A	在规则链的末尾加入新规则
-I	num 在规则链的头部加入新规则
-D	num 删除某一条规则
-s	匹配来源地址 IP/MASK，加叹号"!"表示除这个 IP 外。
-d	匹配目标地址
-i	网卡名称 匹配从这块网卡流入的数据
-o	网卡名称 匹配从这块网卡流出的数据
-p	匹配协议,如 tcp,udp,icmp
--dport num	匹配目标端口号
--sport num	匹配来源端口号

##### 示例
iptables -F  # 清空预设表所有规则链的规则

iptables -L -n  # 查看所有规则
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy DROP)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```
注解：INPUT代表进来的规则；OUTPUT代表出去的规则；FORWARD代表转发规则；policy ACCEPT代表默认允许；DROP代表默认拒绝。

##### 允许22端口连接
```
iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT
# 22为你的ssh端口， -s 192.168.1.0/24表示允许这个网段的机器来连接，其它网段的ip地址是登陆不了你的机器的。 -j ACCEPT表示接受这样的请求
# 规则一经设置都是立即生效的；但是重启电脑后规则将不会保存，所以需要service iptables save。
```

more see： https://github1s.com/jaywcjlove/reference/blob/HEAD/docs/iptables.md