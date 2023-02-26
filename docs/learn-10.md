##### cron
crontab用于定时跑一些任务。

格式：
```
*  *  *  *  *  <要执行的命令>
分 时 日 月 周 

例子：
0 */1 * * * 每1小时
*/1 * * * * 每分钟
0 4 * * * 每天上午4点
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
*/1 * * * * echo "this is a song for you" > tmp.song.log

*/1 * * * * /usr/sbin/chroot --userspec=nobody.nobody / sh -c "echo 'nobody' > /tmp/nobody.log "
```
