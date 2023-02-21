#### 安装rufus
rufus是一个制作u盘启动盘的工具。

安装地址：https://rufus.ie/zh/

#### 安装proxmox
类似vmware的虚拟化平台工具，可以生成虚拟机。

安装地址：https://www.proxmox.com/en/downloads/category/iso-images-pve

下载`Proxmox VE 7.3 ISO Installer`或其他版本。

#### 制作proxmox U盘启动盘步骤 和 创建系统
1.启动rufus-引导类型选择`镜像文件`，选择之前下载好的proxmox iso镜像，以`dd模式`写入，其他默认，点击`开始`进行制作。

2.格式化掉即将用于proxmox系统的磁盘。重启电脑，进入U盘引导系统，按提示步骤即可完成proxmox系统的创建。

3.安装完毕后，可使用浏览器去访问`https://192.168.1.162:8006/`，注意是https