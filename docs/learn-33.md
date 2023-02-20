### python介绍
python，即python解释器，用C语言实现，能调用C的库文件。Python2.0在2000年发布，Python3在08年发布。Python3不完全兼容版本2。

以centos7为例，系统默认安装了Python2.7。

#### 第一行代码
```
# 执行python进入命令行
python

# 打印helloworld
print "helloworld"

# 将 print "helloworld" 存放到hello.py文件，执行
python hello.py
```

### ipython简介


- ipython是一个python的交互式shell，比默认的python shell好用得多，支持变量自动补全，自动缩进，支持bash shell命令，内置了许多很有用的功能和函数。
- 学习ipython将会让我们以一种更高的效率来使用python。
- 同时它也是利用Python进行科学计算和交互可视化的一个最佳的平台。

#### 安装ipython
```
# 可能需要安装pip
yum -y install epel-release #有时候系统rpm库有些包找不到，这时就可以epel-release第三方库来拓展了。
yum install python-pip
pip --version # 查看版本

# 配置阿里云pip源
cd ~
mkdir .pip
cd .pip
touch pip.conf
vi pip.conf

[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com

# 保存后，升级pip，不指定版本的话，版本跨度太大容易有兼容问题
pip install --upgrade pip==19.3.1

# 安装ipython
pip install ipython

```
#### Windows使用ipython
之所以考虑在windows下使用ipython，是因为要用到画图的matplotlib，画图需要用到界面GUI相关的东西。

```
# 输入 cmd 开启一个docs窗口，输入 python -m pip install -U pip 安装pip，安装完会显示python的安装目录路径
# 添加环境变量，在Path下编辑加入： C:\Users\admin\AppData\Local\Programs\Python\Python310\Scripts，接着保存退出。
# 重新开启一个docs窗口，输入pip即可验证
# 安装ipython，pip install ipython
```
#### 使用ipython
```
# 进入命令行，执行
ipython

In [1]:

# 打印，回车执行
print "hello"

# 定义变量
a = 1
# 输入变量 a，输出变量 1
In [3]: a
Out[3]: 1

# 打印所有变量
whos

# 强制重置所有变量
reset -f

# 当前路径
pwd

# 创建目录
mkdir test

# 更多命令
`cd` `ls` `run hello.py` # 更多参考：https://ailearning.apachecn.org/#/docs/da/003 
```
#### 安装Anaconda
介绍： Anaconda是一个好用的python开发IDE，集成了很多科学计算用到的第三方库。

下载Anaconda：https://www.anaconda.com/products/distribution
### Python语法
```
# 计算
In [1]:
2 + 2

Out[1]:
4

# 赋值
a = 0.2

# 字符串，单双引号等价
a = "test"

# 定义多行字符并打印
s = """hello
world"""

print s

# 字符串相加
s = "hello" + " world"

# 定义列表和打印第一项
s = [1, "hello"]

print s[0]

# {}定义集合set，会去重
s = {2, 3, 4, 2}

# {key:value}定义字典并获取值
s = {'dog': 5, 'cat': 4}
s['dog']

# 数组 Numpy Arrays
# 需要先导入需要的包`pip install numpy`，Numpy数组可以进行很多列表不能进行的运算。
In [35]:
from numpy import array
a = array([1, 2, 3, 4])
a

# 写文件
f = open('data.txt', 'w')
f.write('1 2 3 4\n')
f.write('2 3 4 5\n')
f.close()

# 删除文件
import os
os.remove('data.txt')

# 函数
def pil(x, y):
  z = x + y
  return z
print(pil(1, 2))

# numpy数组计算
from numpy import array
x = array([1, 2])
pil(x,2)

# 定义函数并指定默认参数
def pil(x, y=2):
  z = x + y
  return z
pil(3)

# 导入模块
import os
os.getpid()

# 类：class
class Person(object):
  def __init__(self, first, last, age):
    self.first = first
    self.last = last
    self.age = age

  def full_name(self):
    return self.first + ' ' + self.last
person = Person('Mertle', 'Sedgewick', 52)
person.first
person.full_name()

# 网络数据
import urllib2

url = 'http://ichart.finance.yahoo.com/table.csv?s=GE&d=10&e=5&f=2013&g=d&a=0&b=2&c=1962&ignore=.csv'
ge_csv = urllib2.urlopen(url)
data = []
for line in ge_csv:
    data.append(line.split(','))
data[:4]
```
#### Python数据类型
数据类型

类型 例子

整型 -100
浮点数 3.13
字符串 '23hell'
列表 [1, 'hell']
numpy数组 array([1, 2, 3])
字典 {'dog': 3, 'cat': 4}
布尔型 True,Flase

#### 数字
```
# 相加
1 + 2 # 3

# 相减
3 - 4 # -1

# 相乘
3 * 4 # 12

# 相除
12 / 5 # 2

# 幂指数
2 ** 5 # 32

# 取余
32 % 5 # 2

## 浮点数计算
12.0 / 5.0 # 2.4

# 3.4 - 3.2 的结果并不是我们预期的0.2，这是因为浮点数本身储存方式引起的，浮点数本身会存在一点误差
# 我们使用print显示时，Python会自动校正这个结果

# 数学函数
abs(-12.4) # 绝对值
round(21.6) # 取整数，22.0

print min(2, 3, 4, 5) # 最小值，2
print max(2, 4, 3) # 最大值，4

# 类型转换
int(2.456) # 2
```
#### 字符串
```
# 定义
s = 'hello, world'
s = 'hello' + 'world'

## 函数
# 切割split
s = "1 2 3"
a = s.split() # 按照空格切割，返回一个字符列表list

# 连接join
t = ','
lianjie = t.jon(a) # 以","为连接符把list串联成字符串

# 替换replace
s = "hello, world"
b = s.replace('world', 'python')

# 大小写 upper/lower
s= "Hello, World"
s.upper()
s.lower()

## 修剪strip
# 去除两端空格
s = "    hello, world   "
b = s.strip()

# 去除左端空格
s.lstrip()

# 去除右端空格
s.rstrip()

### 更多函数查看
dir(s)
```
#### 索引和分片
```
s = 'hello'
s[0] // 索引，h
s[1:3] // 分片，el
```
#### 列表
有序，[]
```
l = [1, "hello", 2] # 定义列表
l = [] # 空列表
len(l) # 长度
a[0] # 索引
del a[0] # 删除元素
l.append(11) # 添加到最后
l.insert(3,'a') # 在索引3插入a
a.remove(11) # 移除第一个11
a.pop(2) # 删除索引2处的值并返回
a.sort() # 排序
a.reverse() # 元素反向排序
```
#### 可变和不可变类型
```
# 列表是可变的（mutable）
a[0] = 100 # 修改第一项

# 字符串是不可变的
s[0] = 'x' # 报错
print s.replace('world', 'Mars') # 只是返回新的字符串，s不变
```
#### 元组
和列表一样是有序，但元组Tuple不可变
```
t = (1, 2, 3)
t[0] # 索引，10
t[1,3] # 分片，(2, 3)
t[0] = 1 # 不可变，报错
```
#### 字典
{} 或 dict{} 定义字典
```
a = {}
type(a) 
a["one"] = "this is number 1" # 插入
a['one'] # 查看键值，'this is number 1'

## 方法
# get
a = {"dog":24, "cat":23}
a.get("dog") # 24
a.get("pig") # 返回None
a.get("pig","undefined") # 定义默认值'undefined'

# 删除pop
a.pop('dog')

# 更新update
newd = {'pig': 22}
a.update(newd)

# 返回所有key
```
#### 集合
集合set，无序号但不重复，元素以{}包裹
```
a = set() // 定义
a = {1, 2, 3, 1}
```
#### 判断语句
```
# python 通过缩进区分不同代码块
a = 5
if a > 3:
  print "hello" 

# if elif else
if con1:
  <statement1>
elif con2:
  <statement2>
else:
  <statement3>

# 判断是否为闰年：能被400整除；能被4整除且不能被100整除
year = 1900

if year % 400 == 0:
  print "this is leap year"
elif year % 4 == 0 and year % 100 != 0:
  print "this is leap year"
else:
  print "this is not leap year"
```
#### 循环语句
```
for var in sequence:
  <block code>
```
#### 函数
```
def add(x, y):
 """add two nums"""
 a = x + y
 return a
```
说明：
- """add two nums""" 注释，说明函数作用
- return 返回特定的值，如果省略，返回 None 。

#### 模块和包
Python会将所有 .py 结尾的文件认定为Python代码文件
```
# 创建文件ex1.py(可以把ex1.py当作模块)
%%writefile ex1.py

PI = 3.1416

def sum(lst):
    tot = lst[0]
    for value in lst[1:]:
        tot = tot + value
    return tot

w = [0, 1, 2, 3]
print sum(w), PI

# 执行文件
%run ex1.py

# 导入模块
import ex1 # 导入文件会把文件内容执行一遍

# ex1内所有变量将被加载，可以这么使用
ex1.变量名
ex1.函数名

# 删除文件
import os
os.remove('ex1.py')

# __name__ 属性
# 有时候我们想将一个 .py 文件既当作脚本，又能当作模块用，这个时候可以使用 __name__ 这个属性。
# 只有当文件被当作脚本执行的时候， __name__的值才会是 '__main__'，所以我们可以：

%%writefile ex2.py

PI = 3.1416

def sum(lst):
    """ Sum the values in a list
    """
    tot = 0
    for value in lst:
        tot = tot + value
    return tot

def add(x, y):
    " Add two values."
    a = x + y
    return a

def test():
    w = [0,1,2,3]
    assert(sum(w) == 6)
    print 'test passed.'

if __name__ == '__main__':
    test()

# 执行文件
%run ex2.py

# 当作模块导入， test() 不会执行：
import ex2

# 但是可以使用其中的变量：
ex2.PI

# 使用别名
import ex2 as e2
e2.PI

# 导入变量
from ex2 import add, PI,直接调用add(), PI
add(3, 4.5)

# 导入所有，这种导入方法不是很提倡，因为如果你不确定导入的都有哪些，可能覆盖一些已有的函数
from ex2 import *
add(3, 4.5)

# 包
# 假如有一个文件夹
# foo/
- __init__.py
- bar.py
这意味着 foo 是一个包，我们可以这样导入其中的内容:

from foo.bar import func

导入包要求：
- 文件夹 foo 在Python的搜索路径中
- __init__.py 表示 foo 是一个包，它可以是个空文件
```
#### 常用的标准库
- re 正则表达式
- copy 复制
- math cmath 数学
- gzip,bz2 压缩工具
- os 文件系统 
- csv
- xml
- cmd 命令行
#### 异常
下列代码，计算输入数字的对数，输入q退出。假如输入-1，程序会报错异常退出。
```
import math

while True:
  text = raw_input('> ')
  if text[0] == 'q':
    break

  x = float(text)
  y = math.log10(x)
  print "log10{0} = {1}".format(x, y)
  
```
如果不希望程序停止执行，那么我们可以添加一对 try & except：
```
import math

while True:
  try:
    text = raw_input('> ')
    if text[0] == 'q':
        break
    
    x = float(text)
    y = math.log10(x)
    print "log10{0} = {1}".format(x, y)
  except ValueError:
    print "the value must be greater than 0"

```
说明：
- 其他发生其他异常，ValueError没有匹配到，还是会抛异常，可以将except 的值改成 Exception 类，来捕获所有的异常。
  即 `except Exception:` 或者 写多个具体的异常 `except (ValueError, ZeroDivisionError):` 或者分开写，指示不同错误类型:
```
except ValueError:
  print "the value must be greater than 0"
except ZeroDivisionError:
  print "the value must not be 1"
except Exception:
  print "unexpected error"
```
补充一下finally，不管异常与否，finally里面的语句最终都会执行:
```
try:
    print 1 / 0
except ZeroDivisionError:
    print 'divide by 0.'
finally:
    print 'finally was called.'
```
#### 文件读写
```
# 读文件
f = open('test.txt')
text = f.read()
print text
f.close()

# 删除文件
import os
os.remove('test.txt')

# 写文件
f = open('myfile.txt', 'w') # 使用 w 模式时，如果文件不存在会被创建；如果文件已经存在， w 模式会覆盖之前写的所有内容，a 模式代表追加
f.write('hello world!')
f.close() # 关闭文件可以保证内容已经被写入文件，而不关闭可能会出现意想不到的结果
```