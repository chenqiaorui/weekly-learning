#### docker-compose 安装
```
#下载可执行包

curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# docker-compose版本见：`https://github.com/docker/compose/releases`

# 赋可执行权限
chmod +x /usr/local/bin/docker-compose

# 测试是否可用
docker-compose --version
```

#### Dockerfile解析
```
# build 构建镜像
docker build -f /path/to/a/Dockerfile .

# 镜像继承
From <image> AS <name>

# 设置环境变量
ENV APP_HOME /data
ENV NAME="ricky"

# 设置工作目录
WORKDIR /data

# 创建挂载点
VOLUME ["/data"]

# 复制宿主机文件到容器
ADD a.log /opt/a.log
COPY a.log /opt/a.log

# 容器内运行
RUN apt install telnet

# 执行shell
CMD ["sh a.sh", "para1", "para2"]
ENTRYPOINT ["sh a.sh", "para1", "para2"]

# docker build --build-arg key=value 指定参数传递到Dockerfile
ARG user=someuser

# 避免 ADD 或 COPY 将敏感文件添加到镜像中
通过编写.dockerignore:
*/temp*
*/*/temp*
temp?
```
#### Helm目录结构
```
Chart.yaml # 该chart包的版本信息

templates # 存放创建k8s资源的模板yaml文件

values.yaml # 给k8s的yaml文件使用的变量
```
#### 使用helm创建一个模板
```
helm create mychart
```
生成
```
mychart
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```
说明：
- `templates/` 目录下是模板文件，当Helm需要生成chart的时，会渲染该目录下的模板文件，将渲染结果发送给kubernetes
- `values.yaml` 文件保存模板的默认值，用户可以在helm install 或者 helm upgrade可以指定新的值来覆盖默认值
- `Chart.yaml`文件保存chart的基本描述信息，这些描述信息也可以在模板中被引用
- _helper.tpl 用于保存一些可以在该chart中复用的模板

删除templates/下所有文件，创建 `mychart/templates/configmap.yaml`文件
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```
```
helm install clunky-serval ./mychart # clunky-serval是releaseNamed
```

部署成功后，使用`helm get manifest clunky-serval`命令可以看到被部署的yaml文件：
```
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: clunky-serval-configmap
data:
    myvalue: "Hello World"
```