#### sed集合
```
# 把文件所有c1的文本变成d1，`-i`表示对文本文件进行字符替换
sed -i 's/c1/d1/g' test.log

# 当替换字符存在特殊的转义字符，如`"`，可加`\`进行防转义
sed -i "s/sandbox_image = \"k8s.gcr.io\/pause:3.5\"/sandbox_image = \"registry.aliyuncs.com\/google_containers\/pause:3.4.1\"/g" /etc/containerd/config.toml
