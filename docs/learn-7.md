### <<数据库运维系列操作>>
#### 连接数据库
mysql -h127.0.0.1 -P3306 -uroot -prootpassword

#### 查看数据库
show databases;

#### 选择数据库
use test;

### <<表结构系列操作>>
#### 修改字段
ALTER TABLE user
ADD age int(3); 

#### 创建索引
CREATE INDEX idx_name
ON user (name);


### <<数据系列操作>>
#### 插入
INSERT INTO user
VALUES (10, 'root', 'root', 'xxxx@163.com');

#### 更新
UPDATE user
SET username='robot', password='robot'
WHERE username = 'root';

#### 删除
DELETE FROM user
WHERE username = 'robot'; // 删除行

TRUNCATE TABLE user; // 清空表

#### 查询
SELECT * FROM mytable LIMIT 0, 5;

### <<Mysql支持EMOJI>>

