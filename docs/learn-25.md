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

### 使得普通用户ricky能使用root权限？
1.切换至root账户：su root
2.编辑/etc/sudoers文件，如文件只读，需设置```chmod u+w /etc/sudoers```。
新增内容:

```
#ricky   ALL=(ALL:ALL)NOPASSWD: ALL  # 使用sudo时，不需要ricky密码
ricky   ALL=(ALL:ALL) ALL            # 使用sudo时，需要ricky密码
```

:wq保存文件即时生效。```chmod u-w /etc/sudoers``` 去除写。

#### 操作系统概貌

应用程序 Application
 |
 库函数 Libraries
 |
 系统调用 System call
 |
 内核 Linux Kernel
 |
 硬件 Device

 #### cpu
 平均负载：平均活跃进程数

 mpstat -P ALL 5 # 动态监控每个cpu使用情况，-P ALL 表所有cpu，每5s输出一次

 pidstat -u 5  # 隔5s输出一组数组，-u 展示cpu效用。此命令能看pid和cpu占用情况。%wait表进程等待的cpu使用率


 ####  PC组成剖析
 ##### 1.BIOS(基本输入输出系统)，存在于主板ROM中。作用：系统自建，建立中断向量表等。一般BIOS系统装载在ROM（只读存储器）中。
 电源按钮开机，cpu把代码段寄存器CS设置为0XF000（指向BIOS），执行BIOS（做一些硬件检测和初始化工作）。复制BIOS数据到内存指定位置后，让CPU进入实地址模式工作。BIOS从硬盘等块设备把操作系统引导程序加载到内存0x7c00z，系统启动。

 ##### 2.CMOS存储器
 存储系统硬件配置信息和实时时钟信息。

 ##### 3.DMA控制器
 实现外设与内存数据交互（可不受cpu控制）

 ##### 4.键盘控制器
 记录接通码+断开码发送至操作系统的键盘数据队列。

 #### 用C实现列出目录的例子
```
# 安装apue.h
cd /home/
mkdir learnApue
cd learnApue
wget http://www.apuebook.com/src.3e.tar.gz
tar -zxvf src.tar.gz

cd ./src.3e/apue.3e
cp ./include/apue.h /usr/include/
cp ./lib/error.c /usr/include/

vim /usr/include/apue.h
# 最后一行下加：#include "error.c"

cd /home/learnApue/src.3e/apue.3e/
make # 编译

# if error, maybe you should do the follow thing and make again
yum install libbsd libbsd-devel
```
```
### 代码
vi ls.c

#include "apue.h"
#include "dirent.h"

int main(int argc, char *argv[]) {
    DIR *dp;
    struct dirent *dirp;

    if (argc != 2)
        printf("usage: ls direcory_name");
    
    if ((dp = opendir(argv[1])) == NULL)
        printf("can't open %s", argv[1]);

    while ((dirp = readdir(dp)) != NULL)
        printf("%s\n", dirp->d_name);
    closedir(dp);
    exit(0);	 

}

gcc ls.c
./a.out /opt 
```
#### 性能工具
perf top -g -p pid # 动态追踪指定进程调用函数情况
pidstat -p pid # 查看单进程cpu使用、磁盘读写情况 
sar -n DEV 1 # 查看网络收发包情况

### 案例
利用perf生成flame graph
1. perf record -F 99 -P PID -g -- sleep 60  # -F 99 表每秒采样99次，-g 表生成调用栈
perf运行原理：给定采样频率，根据采样频率每隔一段时间对cpu进行中断并记录程序符号(函数)。抽样频率越高代表函数占用cpu时间越长。


#### 动态追踪
systemstap一系列学习内容：https://github.com/lichuang/awesome-systemtap-cn#%E6%95%99%E7%A8%8B%E7%B1%BB
systemstap begin guide: https://sourceware.org/systemtap/SystemTap_Beginners_Guide/using-systemtap.html#installproper

##### 1. systemtap 安装
1.1 前提 
基于centos

1.2
yum install systemtap systemtap-runtime  # 安装systemtap

stap-prep # 检查并自动安装相关内核包：-devel and -debuginfo

stap -v -e 'probe vfs.read { printf( "read performed\n"); exit() }'   # 测试stap是否能正常工作：设置探针，当检测到vfs有read performed后打印。

##### 2. systemstap 工作原理
2.1 将脚本编程解析树
2.2 分析脚本内核符号信息
2.3 转化成c源码
2.4 构建内核模块
2.5 内核模块嵌入内核并将输出发送到stdout。ctrl+c退出进程并卸载模块

##### 3. stap命令行参数
`-x pid` # 传递参数pid，通过target()获得
```
// $ sudo stap x-param.stp -x 10
// 输出：pid:10
probe begin
{
  printf("pid:%d\n", target())
}
```

`-T 3` # 设置3s后退出脚本
```
// $ sudo stap T-params.stp -T 3
// 输出：time:2
global count

probe timer.s(1) {
  count += 1
}

probe end {
  printf("time:%d\n", count)
}
```

`-L` 或 `-l`: 打印二进制文件位置和函数行数

##### 4. systemtap 语法
一般是设置探测点去检测
`probe [event] { statement }` , 其中 event可分为同步事件和异步事件

- 同步事件: 示例1：在指定行设置探测点，打印局部变量值
```
vi cc_stap_test.c

#include <stdio.h>

int main(int argc, char *argv[])
{
	int a;

	a = 1;
	printf("a:%d\n", a);
	a = 2;
	printf("a:%d\n", a);
	return 0;
}


probe process("./a.out").statement("main@./cc_stap_test.c:8") {
  printf("systemtap probe line 8 a:%d\n", $a)
}

```

- 异步事件：如 probe start ; probe end; 

##### openresty-systemtap-toolkit 示例

```
# 跟踪pid为22222的进程，采样时间20秒，只采用用户层信息，输出到tmp.bt文件中
./sample-bt -p 22222 -t 20 -u > tmp.bt

# 调用stackcollapse-stap.pl文件，将第一步采样的数据生成cbt格式文件
./stackcollapse-stap.pl flame.bt > flame.cbt

# 生成火焰图的svg文件
./flamegraph.pl flame.cbt > flame.svg
```

#### 压测工具
ab -c 10 -n 100 http://192.168.0.10:10000/  #　并发10个请求，共100个


#### 性能监控
sar -u 1 2 # 查看cpu使用率，每秒采样1次，共2次。
vmstat 1 2 # 可以 动态 监控cpu和内存

#### 什么是ACL？
ACL(Access Control List) 访问控制列表。区别于普通权限(所属用户、所属组)，增加更细的权限分配。

#### 应用场景
```
# 列出一个文件的权限详情
root@easub-Inspiron-3470:/opt# ll a.log
-rw-r--r-- 1 root root 0 10月  9 14:27 a.log
```
描述：a.log 所属用户、所属组皆为root。
    rw- 表 root用户有读写权限
    r-- 表 root组有读权限。
    r-- 表 其他用户有读权限。

#### 文件描述符、文件句柄和索引节点Inode

`文件描述符`(fd) 是 一个非负整数。进程包含记录项，记录项包含文件描述符fd和文件指针，文件指针指向一个文件表，文件表称为`文件句柄`。文件表包含了文件状态(文件大小、文件类型等)、当前文件偏移量等。Inode存储着文件状态，Inode占用磁盘空间。

- df -i # 查看已使用的Inode数目。inode标识一个文件，文件名并不能代表文件。修改文件名并不改变inode。


#### 磁盘结构
文件存储在磁盘。磁盘->最小存储单位扇区（sector）512字节。

读磁盘一次 -> 读几个扇区(等于块block，块是文件存储的最小单位，常见为4KB，即8个扇区组成一个块)

#### swap
swap(交换空间)：将内存页写到磁盘预配置空间，释放内存页。
cat /proc/sys/vm/swappiness    # swappiness 取值范围0~100（默认为0），数值越大，使用swap概率越高。
free -h 和 swapon -s 皆可以查看swap使用情况。

#### xargs
```ps aux|grep code|awk '{ print $2 }'|xargs -I {} kill {}```
xargs -I 指定替换字符串为{}，{}代表着管道传递的参数，后面再使用{}会替换成相应参数值。