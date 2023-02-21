#### Jenkins简介
开源的基于Java的提供可持续化集成服务的平台

#### install jenkins on centos7
java11安装
```
yum install java-11-openjdk-devel
```
说明：使用jdk8可能启动不了jenkins

添加安装Jenkins的yum源
```
curl --silent --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo
```

导入仓库密钥
```
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
```

安装jenkins最新稳定版
```
yum install jenkins
```

服务管理
```
systemctl start jenkins
```
说明：
- 越新版本的jenkins可能需要的jdk版本越高，jdk11适配jdk8却不一定

开放8080端口
```
firewall-cmd --permanent --zone=public --add-port=8080/tcp
firewall-cmd --reload
```

设置Jenkins
```
http://your_ip_or_domain:8080
sudo cat /var/lib/jenkins/secrets/initialAdminPassword # 获取临时密码
```

选择`Install suggested plugins box`后继续，创建admin用户。

jenkins工作目录：
```
/var/lib/jenkins/
/var/lib/jenkins/config.xml # 配置文件
```
#### jenkins重置密码
```
编辑/var/lib/jenkins/users/admin_491010919283058211/config.xml  # admin_491010919283058211需要根据你的实际文件名调整

# 替换<passwordHash>成下列内容：
<passwordHash>#jbcrypt:$2a$10$ltzg1Kwtef0ymmNAqR8JR.961lHdnrsoFsE.huZ.G4r1AiIaENRC6</passwordHash>
```
说明：
- `#jbcrypt:$2a$10$ltzg1Kwtef0ymmNAqR8JR.961lHdnrsoFsE.huZ.G4r1AiIaENRC6`是hash加密`123456`后的值。
#### 部署java项目实践
配置JDK
```
Manage Jenkins - Global Tool Configuration - JDK（不选择自动安装）- 配置name和JAVA_HOME为`/usr/local/java/jdk1.8.0_351` - 保存
```
说明：
- 若选择自动安装需要设置oracle账号密码
- 手动安装jdk8步骤
  mkdir /usr/local/java
  tar -xzvf jdk-8u351-linux-x64.tar.gz (jdk-8u351-linux-x64.tar.gz包自行到java官网下载)，解压后生成目录`jdk1.8.0_351`

  
配置MAVEN
```
Add Maven - 命名 - Install Automatically - 选择版本3.8.6 - 保存
```

安装插件 Maven Integration，具体步骤：
```
Manage Jenkins - Manage Plugins - 可选插件 - 搜索maven - 安装Maven Integration
```

安装完后，创建一个项目：
```
new Item - 命名emog-test - 选择构建一个maven项目 - 保存
```

进入project：emog-test
```
点击配置 - 设置git：git@192.168.31.87:alias/emog.git
```

Pre Steps
选择`执行shell`
```
mvn clean install

num=`ls target/emog*.jar | wc -l`

[[ $num -ne 1 ]] && { echo "num is not equal 1"; exit 1; }
echo "continue"
jar_path=`ls target/emog*.jar`
jar=`basename "$jar_path"`


\cp -fp target/$jar /opt/emog/jar
\cp -fp /opt/emog/jar/$jar /opt/emog/emog.jar

cd /opt/emog && sh restart.sh
```
说明：restart.sh内容
```
workspace=/opt/emog
cd $workspace
pid=`ps aux|grep "java -jar emog.jar"|grep -v grep|awk '{print $2}'`

if [[ $pid ]];then
  kill $pid && nohup java -jar emog.jar &
else
  nohup java -jar emog.jar &
fi
```
说明：`su jenkins`非登录用户没有bash环境，可以使用`su -s /bin/bash jenkins`以jenkin身份进入。