#### 命令集合
```
# 历史命令
history 
// HISTSIZE 缓冲区记录条数，默认：1000
// HISTFILE 命令存放文件
// HISTFILESIZE 历史命令文件能存的条数：1000

# 别名
alias key='value' // 临时生效
unalias key

# 通配符
a*b // 多个
a?b // 单个
[0-9] // 范围内单个，不区分大小写
[^0-9] // 匹配范围以外单个

# 重定向
&> 标准输出和错误合并
&>> 合并且追加
> file 2>&1 

< 输入重定向
cat << EOF
hello,
world
> EOF

# 管道：成为下一命令的参数
cat /etc/passwd | grep "test"

# 系统信息
cat /proc/cpuinfo // cpu信息

```
