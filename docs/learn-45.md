#### PHP安装
cenots7安装php7.4

下载php包

[php官网](https://www.php.net/downloads.php)

```
wget https://www.php.net/distributions/php-7.4.29.tar.gz
```

解压
```
tar -xvf php-7.4.29.tar.gz
```
切换到目录
```
cd php-7.4.29
```

提前安装依赖工具：
```
yum -y install libxml2-devel openssl-devel sqlite-devel libcurl-devel libicu-devel gcc-c++ oniguruma oniguruma-devel libxslt-devel libpng-devel libjpeg-devel freetype-devel
```
说明：
- 若出现`package not found`错误，那就可能需要更换yum源。

配置安装模块
```
./configure  \
--prefix=/opt/php/php  \
--with-config-file-path=/opt/php/php/etc  \
--enable-fpm  \
--enable-gd  \
--with-external-gd  \
--with-fpm-user=nginx  \
--with-fpm-group=nginx  \
--enable-inline-optimization  \
--disable-debug  \
--disable-rpath  \
--enable-shared  \
--enable-soap  \
--with-libxml \
--with-xmlrpc  \
--with-openssl  \
--with-external-pcre \
--with-sqlite3  \
--with-zlib  \
--enable-bcmath  \
--with-iconv  \
--with-bz2  \
--enable-calendar  \
--with-curl  \
--with-cdb  \
--enable-dom  \
--enable-exif  \
--enable-fileinfo  \
--enable-filter  \
--enable-ftp \
--with-openssl-dir \
--with-jpeg \
--with-zlib-dir \
--with-freetype \
--enable-gd-jis-conv \
--with-gettext \
--with-gmp \
--with-mhash \
--enable-json \
--enable-mbstring \
--enable-mbregex \
--enable-pdo \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-pdo-sqlite \
--with-readline \
--enable-session \
--enable-shmop \
--enable-simplexml \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--with-xsl \
--with-zip \
--enable-mysqlnd-compression-support \
--with-pear \
--enable-opcache
```
说明：
- `--prefix=/opt/php/php` 指定php的安装位置
- `--enable-fpm` 加上这个选项，编译安装后的目录才会有fpm

- 若遇到` (libpcre2-8 >= 10.30) were not met`问题，是因为模块`--with-external-pcre --with-pcre-jit`用到了`pcre2`

  若包不可下载，可前往`http://www.pcre.org/`下载

安装配置pcre2:
```
https://sourceforge.net/projects/pcre/files/pcre2/10.34/pcre2-10.34.tar.bz2/download

tar xjvf pcre2-10.34.tar.bz2

cd pcre2-10.34

./configure

make && make install

export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
```

- 若遇到`(libcurl >= 7.15.5) were not met`
```
yum install libcurl-devel
```
说明：若遇到`libcurl-devel` not found，可能需要添加yun源:
```
cd /etc/yum.repos.d
yum clean all # 清除缓存
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo

sed -i 's/$releasever/7/g' ./CentOS7-Base-163.repo # 替换成7
yum makecache # 建立索引目录
yum install libcurl-devel
```
- 若遇到`No package 'gdlib' found`
```
wget https://github.com/libgd/libgd/releases/download/gd-2.2.5/libgd-2.2.5.tar.gz
tar zxvf libgd-2.2.5.tar.gz
cd libgd-2.2.5
./configure 
make
make install

```

- 若遇到`GNU MP Library version 4.2 or greater required`，则：
```
yum -y install gmp-devel
```

- 若遇到`No package 'libzip' found`，则：
```
wget https://libzip.org/download/libzip-1.2.0.tar.gz
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install

```

配置完成会看到以下界面
```
...
Thank you for using PHP.
...
```
编译和安装
```
make && make install
```

安装完成目录结构
```
$ cd /opt/php/php
$ tree . -L 1

├── bin
├── etc
├── include
├── lib
├── php
├── sbin
└── var
```
编辑`/etc/profile`，文件末尾添加
```
PATH=$PATH:/opt/php/php/bin
export PATH
```

`source /etc/profile`后即可使用`php -v`等命令

安装之后的目录没有`php.ini`，从之前用来安装php的源码包的根目录拷贝`php.ini-production`到`/opt/php/php/etc`目录，改名：
```
mv php.ini-production php.ini
```

php命令
```
php --ini # 查看ini位置
php -v # 查看版本

$ php -a # 交互式运行php
php > $a = 5;
php > echo $a;

```

给php-fpm建个软链
```
ln -s /opt/php/php/sbin/php-fpm /usr/sbin/php-fpm
```

配置php-fpm.conf文件
```
cd /opt/php/php/etc
mv php-fpm.conf.default php-fpm.conf
```
配置www.conf文件
```
cd /opt/php/php/etc/php-fpm.d
mv www.conf.default www.conf
```
启动php-fpm服务
```
php-fpm
```