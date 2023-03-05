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