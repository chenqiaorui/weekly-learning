#### ElasticSearch概念
普通索引：诗名为key，内容为value。
倒排索引：以内容中的一个关键字为key，内容为value。考虑以内容为索引导致索引量巨大，便以诗名为value，value可包含多个诗名。

搜索引擎原理：核心是建立倒排索引、停顿词过滤(的)即分词。

ElasticSearch关键概念：
- 索引：不同于倒排索引，它相当于数据库，用来存数据
- 类型：相当数据库表
- 文档：表行

示例：Poems(索引名 -> 相当数据库名)，类型为poem，一个mapper结构以json形式存放。

- Poems 索引

- 类型
```
{
	"poem": {
		"properties": {
			"author": {
				"type": "keyword"
			},
			"words": {
				"type": "integer"
			}

		}
	}
}

// keyword类型是不会分词的。
```
- 文档
```
{
    "author": "李白"，
    "words": 20
}
```
#### logstash
采集业务数据

#### kabana
可视化平台

http://192.168.1.181:5601/

使用：

1. "三横线菜单" -> Analytics -> Discover -> Create index pattern（输入looklook-*） -> Next Step -> 选择@timestamp -> Create index pattern

2. 然后点击左上角菜单，找到Analytics->点击discover ，日志都显示了。

#### ELK应用场景
日志分析 + 实时监控
