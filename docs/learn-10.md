##### cron
crontab用于定时跑一些任务。

格式：
```
*  *  *  *  *  <要执行的命令>
分 时 日 月 周 
```

Crontab命令：
```
crontab -l # 列出定时任务
crontab -e # 编辑定时任务
crontab -r # 删除crontab文件
systemctl start crond # 启动定时服务
```

系统root级别cron文件目录
- /etc/crontab
- /etc/cron.d

示例：创建/etc/cron.d/php文件，内容如下：
```
# 设置编码
LANG=C.UTF-8
LC_ALL=C.UTF-8

0 0 1 * * nobody cd /tmp/test;php artisan anal:test >> storage/logs/analy-test.log 2>&1

*/1 * * * * root echo "message from /etc/crontab" > /tmp/root.log
```
说明：
- `root` 指定运行定时任务的用户，不可省略，否则定时任务不会运行。

查看定时任务日志`/var/log/cron`

用户级别文件目录
/var/spool/cron/root   # root创建的定时任务，等价于root用户执行crontab -e

示例
执行`crontab -e`打开定时列表
```
# 定时更新时间
5 * * * * /usr/sbin/ntpdate 133.100.11.8 133.100.9.2 time.nist.gov time.windows.com

*/1 * * * * echo "this is a song for you" > tmp.song.log

*/1 * * * * /usr/sbin/chroot --userspec=nobody.nobody / sh -c "echo 'nobody' > /tmp/nobody.log "

* * * * * echo 'Run this command every minute' >> /directory/path/file.log       # 打印并追加日志到文件

* * * * * /usr/bin/php /var/www/domain.com/backup.php > /dev/null 2>&1           # 不管错误或正确的日志都抛到/dev/null

# 执行多条语句
0 0 * * * /usr/sbin/chroot --userspec=nobody.nobody / sh -c "cd /data/tester; /opt/php/bin/php artisan enr:creverdue"

# 置空文件
0 1 * * * sh -c "cd /opt/lelot/logs && > lelot_access.log"

# 查找指定目录文件，修改时间大于300分钟的文件将被压缩；pigz压缩gz文件，-9压缩级别，越高级别压缩慢但质量好，-p用2个进程跑
0 * * * * find /opt/llog/ -mmin +300 -name "*log*" -type  f|grep -v "\.gz"|xargs pigz -9 -p 2

# 压缩修改时间大于1天的文件，只计算1层目录深度下的文件
0 1 * * * find /tmp/webs*/*/storage/logs/ -mtime +1 -maxdepth 1 -name "*.log" -type f|xargs gzip -9

# 删除修改时间超过120分钟的后缀文件
*/20 * * * * find /tmp/strord/tmp -mmin +120 |grep -E ".jpg|.mp4|.txt|.jpg|.ts|.m3u8|.png"|xargs rm -f

# 删除修改时间大于360分钟且文件大小大于5G的文件
0 * * * * find /tmp/webse*/clii*/storage/logs/ -mmin +360 -size +5G -name "*.log"|xargs -I {} truncate -s 0 {}

# 定时同步文件
0 * * * * /usr/bin/rsync -avz -e "ssh -p 22" /tmp/webe/cllease/storage/hots/*.log root@192.168.124.34::hots/hz/ --exclude "*.gz"
```
