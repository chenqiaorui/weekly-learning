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
#### diff 
示例
新建file1文件：

```
Hi,
Hello,
How are you?
I am fine,
Thank you.
```

新建file2文件：
```
Hello,
Hi,
How are you?
I am fine.
```

比较：
`diff file1 file2`  # diff 旧文件 新文件，比如过程是逐行比较，最终变动结果是旧文件内容与新文件内容一致。
```
1d0
< Hi, 
2a2 
> Hi,
4,5c4
< I am fine,
< Thank you.
--- 
> I am fine.

```
说明：
- `1d0`: 1表旧文件第一行，d表删除。<表要删除的行，如果有>，表要增加的行。整句表删除旧文件第一行。
- `2a2`: 左边的2表旧文件的第二行，a表增加。整句表在旧文件第2行后添加Hi。
- `4,5c4`: 4到5行现在已被改变并且需要用新文件中的第4行代替。

#### jq简介
jq，用来处理json数据的工具。

#### jq安装使用
centos安装
```
yum install -y jq
```

使用
```
# 获取一个键的值
echo '{"name":"ricky", "age":18}' |jq '.name'

# 获取数组数据
echo '[{"name": "flolunsa", "age": 12}, {"name": "ricky", "age": 27}]' | jq .[0]
echo '[{"name":"JSON", "good":true}, {"name":"XML", "good":false1}]' | jq '.[1]' # false不能写成false1

# 同时获取多个key的值
echo '{"name":"ricky", "age":18}' |jq '.name, .age'
```

#### 定义一个json
```
# 错误例子: 使用单引号
{
 "url": 'https://www.examples.com'
}

# 错误使用非十进制数据，json只能使用十进制 
{
  "foo": 0x123
}

# 正确例子
# 定义对象
{  
  "bar": "nisha",
  composition: {
    "a": 1,
    "name": "ricky"
  }  
}

```
#### 压缩工具
看一个例子: tar
```
wget http://www.apuebook.com/src.3e.tar.gz
# 解压tar.gz，v显示操作过程，z通过gzip指令解压缩文件，x从归档文件中提取文件，f指定备份文件
tar -zxvf src.3e.tar.gz

tar -xjvf src.gz2 # 解压gz2包，不常用
```

看一个例子: zip 和 unzip
```
zip -q -r ansible.zip ansible # ansible目录压缩成ansible.zip，-q变不显示压缩信息，-r 表递归压缩下面层级的目录或文件

unzip ansible.zip # 解压
```

看一个例子: gzip 和 gunzip
```
gzip access.log # 压缩，原来文件消失
gunzip access.log.gz # 解压，原来gz文件消失
```
注意：gzip常用来压缩日志文件而非目录，当然也可以压缩目录

#### 文本处理工具
例子：sed
```
sed -i s/ricky/chen/g  mcw.log # 把log文件所有ricky替换成chen
```
例子：tee
```
ifconfig | tee test.log # 输出到控制台并存到test.log
echo "hello" | tee -a test.log # 追加到test.log
```
例子：grep
```
grep "ricky" test.log   # 只取选中的字符所在行

grep -v "ricky" test.log  # 取不选中的行

grep -r "ricky" ./*   # 递归查找当前目录及子目录下的ricky

grep -e "rick*" test.log  # 开启正则匹配模式

grep -w 'ricky' txt                          # 精确匹配字符串，-w只会匹配单独存在的ricky，如文本里存在arickya不会被匹配到，但a ricky a 会

1. 筛选ip
grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'

2. 筛选指定内容行的前后3行 
grep -C 3 'test'

3. 筛选指定字符开头
grep -o 'S.*'

4. 筛选两个字符串之间的内容(不包含w1,w2)
grep -o -P '(?<=w1).*(?=w2)'

5. 只取第一个匹配到的
grep -m 1 bob 

6. 忽略大小写
grep -i "bob"
```


#### 远程连接工具
```
ssh root@192.168.1.2 -p 22
ssh-keygen # 生成`id_rsa.pub`和 `id_rsa.private`
/etc/ssh/sshd_config # 配置文件入口，可以修改端口等
systemctl status sshd # 服务管理
```
#### time
```
time sh test.sh # 查看执行脚本的时间 
```
说明：
- `real time` 脚本真正执行全时长，可能包含脚本的sleep时间加user加sys时间。
- `user`和`sys`表示用户空间和内核空间使用cpu的时间。

#### vim
```
:set ff # 查看文本格式，如果从windows环境编辑器编写xx.sh文件，传输到linux机器，文本格式可能是dos。在linux下编写xx.sh，生成的是unix格式。
:set binary # 改为unix格式，改完之后需要:wq保存后退出才生效。
ctrl+r # 恢复撤销
:set list # 可以显示一些符号，$出现在行结尾，代表换行；^I 代表tab键；空格依旧是空格
:%s/admin/ricky/g # 替换所有admin
```

#### watch
```
watch -t -d uptime  # 动态显示程序运行结果， -t隐藏标题栏，-d 高亮变更处
```