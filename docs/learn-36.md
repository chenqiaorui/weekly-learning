#### Centos7更换成aliyun步骤
```
# 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 下载
yum install -y wget
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

或者

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 更改/etc/yum.repos.d/CentOS-Base.repo
使用vi将所有http替换成https `:%s/http/https/g`

# 更新镜像源
yum clean all # 清除缓存
yum makecache # 生成缓存
```

#### Debian更换阿里云软件源
备份
```
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

修改
vim /etc/apt/sources.list
```
deb http://mirrors.aliyun.com/debian/ bullseye main non-free contrib

deb-src http://mirrors.aliyun.com/debian/ bullseye main non-free contrib

deb http://mirrors.aliyun.com/debian-security/ bullseye-security main

deb-src http://mirrors.aliyun.com/debian-security/ bullseye-security main

deb http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib

deb-src http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib

deb http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib

deb-src http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
```

更新
```
apt-get update
```
