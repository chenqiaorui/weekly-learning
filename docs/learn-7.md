### 数据库运维系列操作
#### 连接数据库
mysql -h127.0.0.1 -P3306 -uroot -prootpassword

#### 查看数据库
show databases;

#### 选择数据库
use test;

### 表结构系列操作
#### 添加字段
ALTER TABLE user
ADD age int(3); 

#### 创建索引
CREATE INDEX idx_name
ON user (name);

### 数据系列操作
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

### Mysql备份恢复
编辑/root/script/mysql_backup.sh

```
#!/bin/bash
 
#指定连接数据库信息(用户名、密码、连接地址、端口、安装目录)
DB_USER="root"
DB_PWD="password"
DB_IP="host"
DB_PORT="3306"
#是指mysqldump命令所在目录
DB_DIR="/usr/local/mysql"
 
#获取系统当前时间并格式化为：20210729
BAK_DATE=`date +%Y%m%d`
 
#指定备份文件保存的天数
BAK_DAY=7
#指定备份的数据库,可以指定多个中间用空格隔开，或者不指定则默认全部备份
BAK_DATABASES=("")
#指定备份路径
BAK_PATH="/data/mysql_back"
 
#创建备份目录
mkdir ${BAK_PATH}/$BAK_DATE
 
#开始执行备份
echo "------- $(date +%F_%T) Start MySQL database backup-------- " >>${BAK_PATH}/back.log
#循环遍历
for database in "${BAK_DATABASES[@]}"
do  
    ${DB_DIR}/bin/mysqldump -u${DB_USER} -p${DB_PWD} --host=${DB_IP} --port=${DB_PORT} --databases $database > ${BAK_PATH}/${BAK_DATE}/${database}.sql
done
 
#创建压缩文件
cd ${BAK_PATH}
tar -zcPf db_backup_${BAK_DATE}.tar.gz $BAK_DATE
 
#删除备份目录
mv ${BAK_PATH}/$BAK_DATE /tmp1
 
#遍历备份目录下的文件
LIST=$(ls ${BAK_PATH}/db_backup_*)
 
#获取截止时间，将早于改时间的文件删除
 
SECONDS=$(date -d "$(date +%F) - ${BAK_DAY} days" +%s)
 
for index in ${LIST}
do 
  #获取文件名并格式化，获取时间，如20210729
 timeString=$(echo ${index} | egrep -o "?[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]")
 if [ -n "$timeString" ]
 then
     indexDate=${timeString//./-}
     indexSecond=$( date -d ${indexDate} +%s )
     #与当前时间做比较，把早于7天的文件删除
     if [ $(( $SECOND - $indexDate )) -gt 0 ]
     then 
        rm -f $index
        echo "-------deleted old file $index -------" >> ${BAK_PATH}/back.log
     fi
 fi
done
 
echo "-------$(date +%F_%T) Stop MySQL database backup-------- " >>${BAK_PATH}/back.log
```
定时任务：crontab -e

```
# 每天凌晨3点执行备份，避免影响业务使用，备份时会锁表
0 3 * * * sh /root/mysql_back.sh
```

#### 设置远程连接权限
```
$ docker exec -it mysql mysql -uroot -p
##输入密码：PXDN93VRKUm8TeE7
$ use mysql;
$ update user set host='%' where user='root';
$ FLUSH PRIVILEGES;
```

#### 字符集和字符排序规则
- character set，字符集定义了字符和编码。如字符A的编码为0。

- 排序规则(collation): 定义了字符比较规则。如怎么比较A和B的大小？最直接就是比较编码，因为 0 < 1, 所以 A < B。还有一种规则，不论大小写，如a等于A。这些规则构成了排序规则。

#### Mysql运用
```
# 测试int的最大值

INSERT INTO `test`.`students`(`id`, `name`, `age`, `address`) VALUES (11, '测试int范围', 2147483649, NULL)
> 1264 - Out of range value for column 'age' at row 1

# 测试varchar(20)

INSERT INTO `test`.`students`(`id`, `name`, `age`, `address`) VALUES (11, '测试范围测试范围测试范围测试范围测试范围围', 214748, NULL)
> 1406 - Data too long for column 'name' at row 1

# 测试时间格式datetime

INSERT INTO `test`.`students`(`id`, `name`, `age`, `address`, `create_time`) VALUES (11, '测试时间datetime', 37, 'xx孤社区63号', '2023-03-02 10:47:57');

-- 创建数据库
create database andongni; 

-- 展示所有数据库
show databases;

-- 切换数据库
use andongni;

-- 当前使用的数据库
select database();

-- 创建表
create table dior(
	id int,
  name varchar(20) not null,
	gender varchar(10) default 'male', 
	primary key(id)
);

-- 查看所有表
show tables;

-- 查看表字段
desc dior;

-- 修改字段
alter table dior change name sname varchar(10);

alter table dior modify sname varchar(30);

-- 插入
insert into dior (id, sname) values(3,'a');

-- 更新
update dior set sname='a-update' where id = 2;

-- 删除
delete from dior where id = 2;

-- 过滤重复数据
select distinct sname from dior;

-- 数数
select count(*) from dior;

-- 最大值
select max(id) from dior;

-- 合计
select sum(id) from dior;

-- where
select * from dior where id >= 3;

select * from dior where id in (2,3);

-- limit
select * from dior order by id desc limit 1;

-- 分组统计个数
select count(*),id from dior group by id having id > 2;

-- 关联查询

select * from dior where id = (select id from dior where id =3);

-- left join
select class.cid,class.cname,student.sname from class left outer join student on class.cid=student.classid;

```

#### Basic
```
show processlist; # 查看正在运行的所有客户端连接/工作线程。
alter table user add column password varchar(50); # 添加表字段
alter table user add index idx_username(username); # 添加索引
select version(); # 查看版本
```
### MySQL 主从复制原理
```
mysql主从复制可以实现负载，读写分离，master主要负责写，node负责读。

主从复制类型：
- 主从同步：master和node都写完才通知用户
- 主从异步：master一写完就通知用户
- 主从半同步：master和任一个node写完就通知用户

主从复制原理：
- master需开启了二进制日志跟踪，node服务器通知master：我现在读到了最新的更新位置，然后封锁继续等待master更新通知。

主从复制具体过程：
1/ node启动2个线程，一个IO，另一个sql线程；
2/ IO线程去请求master的binlog日志，且将binlog写到redo log(中继日志)；master特地开了一个log dump进程传输binlog。
3/ node的sql进程用来读redo log，解析成insert等具体操作执行。

```
## Mysql主备搭建
```

```