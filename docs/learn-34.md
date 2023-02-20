#### ECMAScript和JavaScript的关系
ECMAScript规定了浏览器脚本语言的标准，JavaScript是对ECMAScript规范的实现。

#### JavaScript基本语法
```
# 变量
var a = 4; 或 var a, b;

# 标识符
标识符identifier用来识别各种值的合法名称。如变量名和函数名。JS变量名大小写敏感。

标识符命名规则：

第一个字母可以是Unicode(即英文字母或其他语言字母)，或_或$。

不合法的标识符：

1a
*
a+b
-d

# 注释
// 或 /* */

# if语句
if (m === 3) {
  m = m + 1;
} else if () {
  // ..
} else {
  // ..
}

# 三元运算符
var a = (n == 0)? true : false; #  如果n等于0，a为true 

# for循环
var x = 3;
for (var i = 0; i < x; i++) {
    console.log(i)
}

# 数据类型
共6种。

数值number: 整数和小数(1和3.14)
字符串string：如 hello world
布尔值boolean: 表示真伪，true和false
underfined: 表示未定义
null: 表示空值
对象object: 各种值组成的集合，狭义的object又可分为函数function，数组array

# typeof
返回一个值的类型，如typeof 123

# null和undefined
两者都表示"无"，zhuan
var a = undefined;
var b = null;

5 + undefined // Nan，undefined被转换成数值Nan，Not a Number非数字，一些数学运算会出现Nan，如0 / 0 返回NaN
5 + null  // 5，null会被自动转换成数值0

# 字符串
var a = "" 或 var a = ''

# 转义
\ 被称为转义符，如想使用'，就必须加上反斜杠，用来转义。

# 对象
var a = {foo: 'hello', bar: 'world'} 或 var a = {'foo': 'hello', 'bar': 'world'}

a.foo 或 a['foo'] // 获取值
a.foo = "h" // 赋值
Object.keys(a); // ['foo', 'bar']，键查看
delte a.foo // 删除foo
bar in a // true，查看a是否存在键bar

# 遍历object
for (var i in a) {
    // 键：i
    // 值：a[i]
}

# 值拷贝
var x = 1;
var y = x;

x = 2;
y // 1

# 函数
function c(s) {
    console.log(s);
}

或

var c = function(s) {
    return c;
}

或

var c = new Function(
    'return "gell";'
);

# 调用函数
c()

# 闭包
有一种场景：函数外部想调用函数内部定义的变量，怎么办？可以在函数f1里面再定义一个函数f2，返回f2

function f1() {
    function f2() {
      var a = 2;
      return a;
    }
    return f2;
}

f1()() 或者 var result = f1(); result(); // 2，调用

也可以写成:

function f1(){
    return function f2(){var a = 2; return a; }
}

# 数组 []
var a = ['1', '2'] // 定义
a[0]  // 1
a.length // 长度

for (var i in a) {
    // a[i]
}

# 严格相等运算符 === 和 相等运算符区别
=== 先比较类型再比较值
== 转换成同样类型再比较值

# 其他运算符
&& 且
|| 或
void(x = 5) // 执行表达式，然后返回undefined，一般用于书签防止跳转

# 异常处理
function c() {
    try {
        new Array(-1)
    }
    catch (e) {
        console.log("error")
    }
}

# Object对象 
var obj = new Object();
obj.valueOf() === obj // true，返回对象本身

# Array对象
var arr = new Array(2);
arr.length // 2
```
#### console对象和控制台
开发者工具面板
- Elements：查看网页的 HTML 源码和 CSS 代码。
- Resources：查看网页加载的各种资源文件（比如代码文件、字体文件 CSS 文件等），以及在硬盘上创建的各种内容（比如本地缓存、Cookie、Local Storage等）。
- Network：查看网页的 HTTP 通信情况。
- Sources：查看网页加载的脚本源码。
- Timeline：查看各种网页行为随时间变化的情况。
- Performance：查看网页的性能情况，比如 CPU 和内存消耗。
- Console：用来运行 JavaScript 命令。

console.log使用占位符
- %d 整数
- %s 字符串
- %f 浮点数

如：
var a = 2;
console.log("%d value", a)

#### DOM
DOM是js操纵网页的接口。

```
// HTML 代码如下
// <div id="d1">hello world</div>
var div = document.getElementById('d1');
div.nodeName

document.baseURI // 'file:///C:/Users/admin/Downloads/div.html'

var children = document.childNodes;
for (var i = 0; i < children.length; i++) {
  console.log(children[i].nodeType);
}

var foo = document.getElementById('foo');
if (foo.hasChildNodes()) {
  foo.removeChild(foo.childNodes[0]);
}

# 操作css
div.setAttribute(
  'style',
  'background-color:red;' + 'border:1px solid black;'
);

# 事件
function hello() {
  console.log('Hello world');
}

var button = document.getElementById('btn');
button.addEventListener('click', hello, false);

# 浏览器模型
window.localStorage.setItem('foo', 'a');
window.localStorage.setItem('bar', 'b');
window.localStorage.setItem('baz', 'c');

window.localStorage.length // 3
window.localStorage.getItem('foo') // 'a'
```

##### 一. HTML入门
HTML简介

是超文本标记语言，用来表示页面

html 元素

`<html>`表示根元素

head 元素

`<head>`表示包含文档元数据

body 元素

`<head>`表示包含文档可见部分

HTML属性
- class
- id
- style
- title

HTML标签释义
```
<p>段落</>
<br/><!-- 这是换行 -->
<b>粗体文本</b>
<a href="http://www.example.com/">This is a Link</a>
<!-- 无序列表 -->
<ul>
  <li>First item</li>
  <li>Next item</li>
</ul>

<!-- 有序列表 -->
<ol>
  <li>First item</li>
  <li>Next item</li>
</ol>
```

表单
```
<form action="http://www.example.com/test.asp" method="post/get">
  <input type="text" name="lastname" value="Niox" size="30" maxlength="50" />
</form>
```

#### package.json属性介绍
```
# name 
{
  "name": "my-awesome-package"
}
- 不能以.和_开头
- 不能包含大写字母

# version
{
  "version": "1.0.0"
}
- 包当前版本

# description
{
  "description": "我的包的概要简短描述"
}
- 帮助使用者了解包的功能的字符串，包管理器也会把这个字符串作为搜索关键词

# license
{
  "license": "MIT",
  "license": "(MIT or GPL-3.0)",
  "license": "SEE LICENSE IN LICENSE_FILENAME.txt",
  "license": "UNLICENSED"
}
- 所有包都应该指定许可证，以便让用户了解他们是在什么授权下使用此包，以及此包还有哪些附加限制。

# homepage
{
  "homepage": "https://your-package.org"
}
- 是包的项目主页或者文档首页。

# repository
{
  "repository": {
    "type": "git", "url": "https://github.com/user/repo.git"
  },
  "repository": "github:user/repo",
  "repository": "gitlab:user/repo",
  "repository": "bitbucket:user/repo",
  "repository": "gist:a1b2c3d4e5f"
}
- 包的实际代码所在的位置。

# author
{
  "author": {
    "name": "Your Name",
    "email": "you@xxx.com",
    "url": "http://your-x.com"
  },
  "author": "Your Name <you@xxx.com> (http://your-x.com)"
}
- 项目的维护者。

# contributors
{
  "contributors": [
    { "name": "Your Friend", "email": "friend@xxx.com", "url": "http://friends-xx.com" }
    { "name": "Other Friend", "email": "other@xxx.com", "url": "http://other-xx.com" }
  ],
  "contributors": [
    "Your Friend <friend@xxx.com> (http://friends-xx.com)",
    "Other Friend <other@xxx.com> (http://other-xx.com)"
  ]
}

# main 
{
  "main": "filename.js"
}
- 项目入口文件

# scripts
"scripts": {
    "dev": "vuepress dev docs",
    "build": "vuepress build docs"
}
- npm 命令

# dependencies
{
  "dependencies": {
    "colors":   "*",
    "foo": "1.0.0 - 2.9999.9999",
    "bar": ">=1.0.2 <2.1.2",
    "baz": ">1.0.2 <=2.3.4",
    "boo": "2.0.1",
    "qux": "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0",
    "asd": "http://asdf.com/asdf.tar.gz",
    "til": "~1.2",
    "elf": "~1.2.3",
    "two": "2.x",
    "thr": "3.3.x",
    "lat": "latest",
    "dyl": "file:./path/to/dyl",
    "pla": "https://github.com/user/project/tarball/branch",
    "stu": "git://github.com/user/project.git#commit-ish"
  }
}
- 开发版和发布版需要的依赖
- 可以指定一个确切的版本、一个最小的版本 (比如 >=) 或者一个版本范围 (比如 >= ... <)。 包也可以指向本地的一个目录文件夹，参考：https://docs.npmjs.com/cli/v8/configuring-npm/package-json#dependencies

# devDependencies
{
  "devDependencies": {
    "package-2": "^0.4.2"
  }
}
- 只在你的包开发期间需要，但是生产环境不会被安装的包
```
#### Robot.txt文件格式
```
# -----------------------------------------------------------------------------
# author ricky.chen
# fileEncoding = UTF-8
#
# 禁止爬虫爬取无效URL，提升网站核心静态资源抓取及索引效率。
# 无效URL包含:登陆后页面的URL，全动态URL
# 等各种无需被SE收录的URL。
# -----------------------------------------------------------------------------

User-agent: *
Disallow: /enstar
Disallow: /*?*
Disallow: /m/
```