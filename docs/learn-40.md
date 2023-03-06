#### kubeadm, kubelet, kubectl介绍
kubeadm: 用于创建k8s集群的工具。

命令
```
kubeadm init # 初始化集群
kubeadm join <ip节点和端口> # 添加节点到集群 
```

kubectl：对集群资源(pod,deployment)进行管理。
```
kubectl exec -ti deployment/cron-release -n=$namespace --container=php-fpm bash # 进入容器
kubectl cp -c php-fpm $namespace/cron-release-6f4d49956b-pn6r4:/tmp/百度.pdf ./百度.pdf # 拷贝容器文件到本地
kubectl cp ./百度.pdf -c php-fpm namesp/cron-release-6f4d49956b-pn6r4:/tmp/百度.pdf  # 拷贝本地文件到容器
kubectl rollout restart deployment/$project --namespace=$namespace # 更新deployment
```

kubelet: 运行在每个node上，负责pod的生命周期。

#### 环境架构
```
Master: 管理集群节点。
Nodes: 容器运行环境。

# Apiserver
- 与node的kubelet交互，存储node节点状态信息到etcd；
- 提供外部api接口，对集群进行操作

代表组件：kube-apiserver

# Scheduler
具备调度能力，将服务部署到合适的node。代表组件：kube-scheduler

# Controller
- 监控node上的pod监控状态，进行服务自愈
- 管理pod生命周期

代表组件：kube-controller-manager

节点Controller类型：
- Depolyment：管理无状态应用，方便扩缩
- StatefulSet：管理有状态应用

# kubelet
每个节点都会部署一个，管理pod，上传pod状态

# kube-proxy
维护pod和svc的路由规则

# Pod
同一pod网络和volume共享

# Select 标签选择器
举例：deployment通过标签选择器管理其下的pod

# Service
暴露服务，通过标签选择器找到Deployment等。
- 访问格式：svcname.namespace.svc.cluster.local
```
#### kubectl
kube-apiserver的客户端，实现k8s资源的增删改查等操作。

常用命令：
```
# 创建deployment
kubectl run nginx --image=nginx --replicas=5

# 查看pod
kubectl get pods -n kube-system -o wide

# 删除pod，会重建
kubectl delete pods nginx-deploy-5c9b546997-jsmk6

# 拷贝文件到pod
kubectl cp flannel.tar  nginx-58cd4d4f44-8pwb7:/usr/share/nginx/html

# 进入pod
kubectl exec -it nginx-58cd4d4f44-8pwb7 -- /bin/bash

# 扩容
kubectl scale --replicas=5 deployment nginx-deploy

# 修改service为NodePort
kubectl edit service nginx # type: ClusterIP -> type: NodePort

# 查看日志
kubectl logs pod-demo busybox
```
#### pod配置清单
```
# pod重启策略key：restartPolicy
- Always：容器挂了总是重启
- OnFailure：只有其状态为错误的时候才去重启它
- Never：从来不重启，挂了就挂了

# pod生命周期
Pending：已经创建但是没有适合运行它的节点

用户提交创建请求后，交给apiserver，接着scheduler进行调度，调度结果存到etcd，apiserver通知节点的kubelet去部署，部署状态返回存到etcd。

# probe探针监测
livenessprobe # 容器是否运行，失败按restartPolicy策略重启
readinessProbe # 容器探测端口是否可用，如果失败service会移除pod的ip

failureThreshold    # 探测几次才判定为探测失败，默认为 3 次。
periodSeconds       # 每次探测周期的间隔时长。
timeoutSeconds      # 每次探测发出后等待结果的超时时间，默认为 1 秒。
initalDelaySeconds  # 在容器启动后延迟多久去进行探测，默认为启动容器后立即探测。

# lifecycle 生命周期钩子
postStart           # 在容器启动后立即执行的命令，如果这个操作失败了，那么容器会终止，且根据 restartPolicy 来决定是否重启
preStop             # 在容器终止前立即执行的命令
```