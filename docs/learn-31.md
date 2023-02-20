#### 正则表达式
正则表达式是从左到右匹配字符串的模式。

如：

^[a-z0-9_-]{3,15}$

^是开始标记，[a-z0-9_-]是数字字母_-中的一个，{3,15}是3~15个字符长度，$是结束标志。以上正则表达式可以接受_john，但不能接受Jo，因为包含大写且短。

[在线练习](https://regex101.com/r/dmRygT/1)

more check `https://github.com/ziishaned/learn-regex/blob/master/translations/README-cn.md`