# 每周学习第 3 期

这里记录过去一周，我看到的值得学习的东西，每周六发布。
## 怎么去部署SpringBoot项目
### 示例项目
参考文档: https://www.macrozheng.com/mall/deploy/mall_deploy_docker_compose.html#docker%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E4%BD%BF%E7%94%A8

```
# 服务器配置
CenterOS7.6版本，推荐6G以上内存。

# pull docker 镜像
docker pull mysql:5.7
docker pull redis:7
docker pull nginx:1.22
docker pull rabbitmq:3.9-management
docker pull logstash:7.17.3
docker pull mongo:4

# ...

# 业务篇
## 商品列表 pms_product 
表设计：
1.表名以下划线隔开
2.id设计为AUTO_INCREMENT，类型长度bigint(20)
3.图片字段类型长度设计为varchar(255)
4.name设计为NOT NULL，类型长度varchar(64)
5.delete_status这种int(1)，添加COMMENT '删除状态：0->未删除；1->已删除'
6.价格price类型长度为decimal(10,2)
7.文本describe类型设计为text
8.create_time时间类型为datetime

## 商品管理Controller
1.支持swagger
2.创建商品
- 接口：/product/create，方法名create，参数传入@RequestBody ProductParam，返回值为int类型，但此处可封装CommentResult

- body传递给service接口方法create，impl实现类标注@Service，自动装配ProductMapper接口，再根据resources/mapper/productMapper.xml文件进行sql操作。（mbg能根据数据库表生成相应model和mapperxml和mapper类）

- 用到druid管理数据库连接池、分页功能
```
