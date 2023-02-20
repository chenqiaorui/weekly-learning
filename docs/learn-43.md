#### 应用场景
lvm能做什么？组装多个物理磁盘，划分逻辑磁盘，逻辑磁盘可以动态扩缩。

#### lvm原理
```
lvm(Logic Volume Manage)逻辑卷管理。

# 先分区

`fdisk -l`查看新加的磁盘`/dev/sda`，给它分区：`fdisk /dev/sda`

输入`n`，选择p，起始扇区回车选择默认，终止扇区输入`+10G`容量。

输入`t`，设置id为`8e`，`8e`代表分区类型为linux-lvm。

输入`w`保存配置。

# 安装lvm2
yum -y install lvm2

# 创建pv(Physical Volume)物理卷
pvcreate /dev/sda1
pvscan # 查看pv

# 创建VG(Volume Group)卷组，把pv加到它下面
vgcreate vgdata /dev/sda1 # vgdata 是卷组名称 

# 从卷组vgdata中创建lv(Logic Volume)逻辑卷
lvcreate -L 9G -n lv001 vgdata # 分配9G给逻辑卷lv001，这个lv的目录为/dev/vgdata/lv001

# 格式化lv001的文件系统为ext4
mkfs -t ext4 /dev/vgdata/lv001

# 创建/data目录，并将/dev/vgdata/lv001挂载到它
mkdir /data
mount /dev/vgdata/lv001 /data 

# 设置开机自动挂载
编辑 /etc/fstab，添加一行:

/dev/vgdata/lv001   /data  ext4 defaults 0  0

以上完成就可以使用了。

## 其他操作

```
参考：https://www.cnblogs.com/large-show/p/16203274.html

应用场景：机器新加了一块磁盘，`df -h`查看，没有看到新加的磁盘。怎么办？

#### 磁盘分区和挂载
`fdisk -l` 查看当前系统所有磁盘设备。

`fdisk /dev/sdb` 进入交互界面，准备对sdb磁盘进行分区。

指令选项说明: 
```
m 帮助信息；
n 新建分区
d 删除分区
t 转换系统id，即转换格式
w 保存信息，将变更写入磁盘。
q 退出
```

输入`n`进行分区: p表建立主分区

以上设置仅保存在内存中，输入`w`正式写到磁盘。

说明：
- 如果出现"All primary partitions are in use"，可输入`d`删除分区。

分区完成后，需要格式化磁盘指定文件系统类型，centos7默认为ext4。

`mkfs -t ext4 /dev/sdb1` # 设置分区sdb1的文件系统类型为ext4。

格式化完成后，挂载:
`mount /dev/sdb1 /mnt/sdb1` # 其中`/mnt/sdb1`是挂载目录，又称为挂载点。

执行`df -h`即可查看到磁盘完成挂载。

但是，系统在重启之后，发现挂载失效了。

`blkid |grep /dev/sda1` # 获取分区的uuid

编辑`/etc/fstab`，加入：
```
UUID=ea31269e-1e6b-4aac-8b14-19f4c577ce14 /mnt/sda1       ext4   defaults 0    0
```

执行`mount -a`加载磁盘挂载。 
