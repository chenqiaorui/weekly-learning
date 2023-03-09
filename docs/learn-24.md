#### 常用命令
```
git checkout master # 切换到主干代码
git pull # 拉取代码
git checkout -b myfeature # 新建一个功能分支
git push --set-upstream origin myfeature # 分支推送到远程仓库
git add . # 保存变更
git status # 查看变更文件名
git diff # 查看变更的具体内容
git log -n 1 # 查看最新的一次提交记录
```
#### gitea简介
代码版本控制系统

#### gitea安装
切换到`/opt/gitea`目录，编辑`docker-compose.yml`:
```
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.17.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=mysql
      - GITEA__database__HOST=db:3306
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
    depends_on:
      - db

  db:
    image: mysql:8
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
    networks:
      - gitea
    volumes:
      - ./mysql:/var/lib/mysql
```

启动服务和其他：
```
docker-compose up -d # 启动
docker-compose down # 删除容器，但挂载内容保留
docker-compose ps # 展示容器
docker-compose logs # 查看日志
```

访问服务：
```
http://192.168.31.87:3000/
```
