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

kubelet: 运行在每个node上，负责pod的生命周期