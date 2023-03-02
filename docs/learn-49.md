#### containerd
ctr ns ls # 列出命名空间
ctr -n k8s.io images ls   # 列出命名空间k8s.io下的镜像


#### 拉取镜像脚本
setup.sh

```
#!/bin/bash
repo=registry.aliyuncs.com/google_containers

name=k8s.gcr.io/metrics-server/metrics-server:v0.6.1

# remove prefix
#src_name=${name#k8s.gcr.io/}
#src_name=${name#metrics-server/}
src_name=metrics-server:v0.6.1

ctr -n=k8s.io image pull $repo/$src_name

# rename to fit k8s
ctr -n=k8s.io image tag $repo/$src_name $name
ctr -n=k8s.io image rm $repo/$src_name
```
