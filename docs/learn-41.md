#### LDAP简介
LDAP(Lightweight Directory Access Protocol)轻量级目录访问协议，用于用户管理的统一认证服务。

解决的痛点：比如说公司有多个系统有不同套用户密码，但接入开源的openldap之后，就可以共享同一套认证密码。
#### LDAP原理
目录是一个搜索查询、树结构数据库，ldap是一套由目录加协议组成的认证系统。

目录树组成成分：
- `dn` -- Distinguished Name 一条记录的具体位置，如dn:"uid=er.xiong,ou=OA,dc=xiongchumo,dc=com"表示熊出没公司OA组的熊二
- `dc` -- Domain component 域名部分，一条记录可包含多个dc
- `ou` -- Organization Unit 组织单元，一条记录所属组织 
- `uid` -- user id 用户id
- `sn` -- Surname 姓，如“许”
- `cn` -- Common Name，公共名称，一条记录的名称
- `rdn` -- 相对辨别名，类似于文件系统中的相对路径，它是与目录树结构无关的部分，如“uid=tom”或“cn= Thomas Johansson”

#### openldap安装使用
ldap只是一套协议，openldap是基于这套协议的实现。

安装机器：centos7

yum安装openldap
```
yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-devel
```
说明：
- openldap 它是openldap客户端和服务端的公共库
- compat-openldap openldap兼容性库
- openldap-clients 启动服务和设置
- openldap-servers 启动服务和设置
- openldap-devel 工具包，可选择安装
- openldap-servers-sql 支持sql模块，可选择安装
- migrationtools 通过migrationtools实现OpenLDAP用户及用户组的添加，导入系统账户，可进行选择性安装

相关目录
```
/etc/openldap/slapd.d/ # 存放配置文件
```

服务管理命令
```
service slapd start # 启动openldap服务
slapd -VV # 查看版本
```
#### 示例说明：添加管理员密码
生成管理员密码
```
slappasswd -s 123456 # {SSHA}XoEOHoalUAXb7dNiirKs7yFl/1/N/QYF
```

创建目录并添加修改管理员密码的ldif文件
```
mkdir /opt/ldap-workspace
touch changepwd.ldif

# 文件内容如下：
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}XoEOHoalUAXb7dNiirKs7yFl/1/N/QYF
```
说明：
- 第一行表将要变更配置的文件：/etc/openldap/slapd.d/cn=config/olcDatabase={0}config.ldif
- 第二行表修改类型
- 第三行表要添加的配置项是olcRootPW
- 第四行是加配置项值

执行changepwd文件
```
ldapadd -Y EXTERNAL -H ldapi:/// -f changepwd.ldif
```

结果
```
cat /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{0\}config.ldif，新增了一个olcRootPW项
```
参考：https://www.cnblogs.com/swordfall/p/12119010.html

https://www.cnblogs.com/wilburxu/p/9174353.html

https://www.jianshu.com/p/7e4d99f6baaf


#### 部署生产环境可用的openldap
```
# 安装启动
yum install -y openldap-clients openldap-servers

cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

chown ldap:ldap /var/lib/ldap/DB_CONFIG

systemctl start slapd

systemctl enable slapd

# 导入预设模式
find /etc/openldap/schema/ -name "*.ldif" -exec ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f {} \; 

# 设置管理用户root密码：
```
slappasswd -s 123456 # 生成加密密码

chrootpwd.ldif # -H 指定ldap服务器地址；
```

# 设置域名cn=admin,dc=emog,dc=com对olcDatabase={1}monitor等文件有存取权限

mkdir -p /etc/openldap/init_ldif # init_ldif当作工作区目录
cd /etc/openldap/init_ldif

ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif # chdomain.ldif文件在后面的说明处。

# 创建emog根域名，并在其下设置admin用户管理整个根组织，再创建两个组织：Group和People。

ldapadd -x -D cn=admin,dc=emog,dc=com -W -f addOrg.ldif

# UI连接管理 LdapAdmin: http://www.ldapadmin.org/download/ldapadmin.html
Host: 192.168.1.146  Port: 389  Version: 3
Base: dc=emog,dc=com
Username: cn=admin,dc=emog,dc=com
Password: 123456

# 新建用户
右击 ou=People 目录新建用户
仅配置sn: chenxiaoming,
cn: chenxiaoming,
homeDirectory为 /

# 新建Group
右击 ou=Group 目录新建Organizational Unit类型，名为Jenkins，再在Jenkins下新建group类型，名为jenkins-users，这时候有一个属性`member`可配置为`	cn=lisi,ou=People,dc=emog,dc=com`，代表lisi这个人归属于Jenkins-users这个group下面。

# 至此，配置完成了。
```
说明：
- chrootpwd.ldif文件内容：
```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}mMZx/2fkVQDpjnQELwYlILXVW/ybXnMy
```

- chdomain.ldif文件内容：
```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"  read by dn.base="cn=admin,dc=emog,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=emog,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=emog,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=admin,dc=emog,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=admin,dc=emog,dc=com" write by * read

```
- add-org.ldif文件内容：
```
dn: dc=emog,dc=com
dc: emog
objectClass: top
objectClass: domain
o: emog

dn: cn=admin,dc=emog,dc=com
objectClass: organizationalRole
cn: admin
description: LDAP admin

dn: ou=Group,dc=emog,dc=com
ou: Group
objectClass: organizationalUnit

dn: ou=People,dc=emog,dc=com
ou: People
objectClass: organizationalUnit
```

#### gitea接入openldap
前提：
gitea和openldap部署好了。

[gitea安装](../Git/gitea.md)

接入步骤
1. 浏览器登入gitea，点击右侧的个人头像-管理后台-认证源-添加认证源
2. 认证类型(LDAP (via BindDN))-认证名称(allsd11-openldap)-安全协议(Unencrypted)-主机(192.168.31.87)-端口(389)-用户搜索基准(ou=People,dc=emog,dc=com)-用户过滤规则((&(objectclass=top)(cn=%s)))-勾选 从Bind DN中拉去属性信息，该认证源已经启用。

说明：
- 因为没有用TLS证书，所以选Unencrypted
- 其他选项默认不动即可

#### jenkins接入openldap

1. 登录jenkins-系统管理-全局安全配置-安全域(LDAP)
2. 开始配置LDAP：Server(192.168.31.87:389)-roo DN(dc=emog,dc=com)-勾选Allow blank rootDN-User search base(ou=People)-User search filter(cn={0})-Group search base(ou=Jenkins,ou=Group)-点选Search for LDAP groups containing user-Manager DN(cn=admin,dc=emog,dc=com)-Manager Password
(123456)-保存

说明：
- `root DN`的设置是会后面的`User search base`和`Group search base`服务的，如`User search base`设置成ou=People，认证的时候就会到`ou=People,dc=emog,dc=com`下面查找用户。
- 其他选项没有提及的就保持不动。

3. 配置完成后，点击`保存`退出。点击`全局安全配置`进入，可以点击`Testing LDAP settings`测试。接着往下，看到`授权策略`，点击`Add user`添加用户，User name是你早早就设置在`ou=People`下面的用户才行，如`zhangsan` ，最后勾选那些"Read"等权限。最好规划一个administer用户，因为系统认证方式的admin已经不能用了。

4. 登录：使用`zhangsan`用户就可以登录jenkins了。

参考：https://blog.csdn.net/GX_1_11_real/article/details/109511636

#### 遇到问题
- 1.jenkins认证方式修改成ldap认证后，原来系统认证用户admin使用不了了，怎么办？
```
编辑/var/lib/jenkins/config.xml文件

把：
<securityRealm class=...>
  ....    
</securityRealm>

替换成系统用户认证：

<securityRealm class="hudson.security.HudsonPrivateSecurityRealm">
  <disableSignup>false</disableSignup>
  <enableCaptcha>false</enableCaptcha>
</securityRealm>

最后，重启：systemctl restart jenkins
```