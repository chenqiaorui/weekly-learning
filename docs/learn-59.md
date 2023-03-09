#### deploy wordpress
编辑docker-compose.yml
```
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:

# more message check: https://hub.docker.com/_/wordpress
```

### 缓存策略 
```
# 开启缓存插件 -- 用于加速网站访问性能
WP Super Cache # 主要是对于静态页面的缓存

说明：不同缓存插件同时使用可能会造成tcp连接无法释放问题。

# 网站评估--页面加速建议
https://pagespeed.web.dev/

# apache开启apcache
预加载字节码，把php脚本字节码预先加载到内存，提高性能，代价是消耗内存；

修改预加载的脚本不方便，如果想要修改预加载的脚本，必须要重启整个PHP进程树以刷新opcache，因此opcache预加载只适用于生产环境，不适合开发环境；

apcache对于用于一次性执行脚本任务的cli进程而言是不适用的。

# 查看apcache是否开启
php -i |grep opcache| grep "enable"

说明：
zend_extension=opcache.so
opcache.enable=1
opcache.enable_cli=0 //disable php_cli
opcache.memory_consumption=256 //共享内存大小
opcache.max_accelerated_files=40000 //最大缓存的文件数目
opcache.revalidate_freq=60 //60s检查一次文件更新
opcache.fast_shutdown=1
opcache.save_comments=0 //不保存文件/函数的注释
```