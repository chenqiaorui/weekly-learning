#### 安装
环境: centos7

OpenJDK1.8下载地址：https://jdk.java.net/java-se-ri/8-MR3

OpenJDK9以上版本下载地址：http://jdk.java.net/archive/

安装JDK8
```
cd /opt/java

wget https://download.java.net/openjdk/jdk8u42/ri/openjdk-8u42-b03-linux-x64-14_jul_2022.tar.gz

tar -xzvf openjdk-8u42-b03-linux-x64-14_jul_2022.tar.gz

mv java-se-8u42-ri  java8

# vi /etc/profile

export JAVA_HOME="/opt/java/java8"
export JRE_HOME="/opt/java/java8/jre"
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin/:$PATH

source /etc/profile

java -version
```

#### 使用
```
cd /opt/java-demo
vi HelloWorld.java

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}

javac HelloWorld.java # 编译生成.class
java HelloWorld # 执行
```
#### Maven
属性介绍
```
<groupId>   // 团队标志。约定为组织的逆向名称，如org.apache
<artifactId> // 项目唯一标识符。如tomcat，不要在artifactId包含.
<packaging>  // 项目打包后的输出，默认为jar，类型为war产生一个web应用。
```

#### Maven安装
环境：centos7

官网地址：https://maven.apache.org/download.cgi

```
cd /opt/maven
wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz --no-check-certificate
tar -xzvf apache-maven-3.8.6-bin.tar.gz
mv apache-maven-3.8.6 maven

# vi /etc/profile，加入

export PATH=$PATH:/opt/maven/maven/bin

# 执行 source /etc/profile

mvn -v
```

#### pom.xml介绍
```
# <dependences>标签包含的属性标签：
<groupId> // 项目组织，对应包结构
<artifactId> // 项目名称，对应根目录
<version> // jar版本，可以在<properties>中定义
<scope> // jar包作用范围。可以填写compile，runtime，test，system，provided。
<dependenceManage> // 依赖声明，包不会被mvn加载，但可以被子pom.xml继承，这样子pom只需要声明<groupId>和<artifactId>，dependenceManage就是用来做版本统一的。
<properties> // 定义属性，${property}调用
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  // 指定maven构建时的编码
  <maven.compiler.source>1.7</maven.compiler.source> // 指定构建时的jdk版本
  <maven.compiler.target>1.7</maven.compiler.target>
</properties>
```

如何寻找jar包？

在 (http://mvnrepository.com/) 搜索关键字，如log4j。

#### 如何一次编译多个project？
父工程app包含子app1和app2，在app的pom.xml引入两个字module
```
<project>
...
<modules>
    <module>app1</module>
    <module>app2</module>
  </modules>
...
</project>
```
#### Maven命令
```
mvn compile // 编译项目源码
mvn test // 使用合适单元测试框架进行测试
mvn verify // 执行所有检查，验证包是有效的
mvn install // 安装包到本地仓库
```

#### Maven全局配置settings.xml
settings.xml存在两个位置
- ${maven_home}/conf/settings.xml // 全局配置
- ${home}/.m2/settings.xml // 用户配置

pom.xml是本地项目配置

说明：pom.xml > 用户配置 > 全局配置（使用优先级比较）

settings.xml内容：
```
<localRepository>~/.m2/repository</localRepository> // 本地仓库路径配置
<servers> // 一些包的拉取需要认证，这些信息不适合配在pom
<mirrors> // 仓库镜像地址
```

#### 常用问题
```
# 执行 mvn package 或 mvn install 时，会自动编译所有单元测试(src/test/java 目录下的代码)，如何跳过这一步？
  在执行命令的后面，添加命令行参数 -Dmaven.test.skip=true 或者 -DskipTests=true