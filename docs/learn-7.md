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

### <<Mysql备份恢复>>
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

