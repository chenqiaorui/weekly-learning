nc -v host port # 类telnet功能，侦测端口

#### rsync 增量同步
删除策略： 只确保所有内容复制到目标目录，不会删除目标目录文件

权限： 与源端权限一致，除非是目的端已有文件

说明：
- 软链无法同步

```
# 传一个文件
rsync ./justfile root@192.168.31.87:/opt/transaction

# -r 传目录
rsync -r  ./rsync-test root@192.168.31.87:/opt/transaction
```