#### ansible介绍
ansible，就是通过ssh登录到远程主机执行命令的工具。

#### centos 安装ansible和配置
```
yum install ansible -y
```

配置Ansible管理节点和主机的连接，在管理节点执行：
```
ssh-keygen
ssh-copy-id root@ip # 复制Ansible管理节点的公钥到远程主机，这样它登录到远程主机就不用密码了。
```

设置远程主机地址的配置文件默认在`/etc/ansible/hosts`
```
# 不限定组的写法
192.168.31.166

# 命名一个组：web，一个组内可以设置多个远程主机节点
[web]
192.168.31.167
```

#### ansible命令
```
ansible all -m ping # 检查ansible能否ping通管理主机，all表示hosts文件内写的所有主机
ansible all -a "ls /opt/zz" # 在所有主机下执行命令
ansible web -m copy -a "src=/etc/hosts dest=/tmp/hosts" # 拷贝文件到web组的/tmp/hosts
ansible web -m yum -a "name=acme state=present" # 在web组安装acme
ansible all -m user -a "name=foo password=<crypted password here>" # 添加用户
ansible web -m service -a "name=httpd state=started" # 启动web组的httpd服务
ansible all -m setup # 查看远程主机的全部系统信息
```
#### ansible 脚本编排 之ansible-playbook
ansible脚本名字叫playbook，以yml或yaml形式书写。

执行脚本playbook命令
```
ansible-playbook deploy.yml
```

#### ansible-playbook.yml书写注意
- 文件名以`yml`或`yaml`结尾
- 以`---`开头表示文件开头
- `#` 表示注释
- 都是以`-`加空格再写内容
- hosts, variables, roles tasks等对象都是以`:`分割，冒号后要加一个空格
- 执行脚本文件输出绿色代表成功，红色为失败

核心的对象元素
```
hosts 主机组，如- hosts: node1
tasks 任务列表
```
#### ansible-playbook使用
```
ansible-playbook -i hosts deploy.yml
```
说明:
- 命令后加选项 `--syntax-check` 可以检查语法，如 `ansible-playbook deploy.yml --syntax-check`
- `--start-at="Install kubelet, kubeadm and kubectl"`从指定任务开始执行

##### 示例1：安装软件
编辑ansible-demo.yaml
```
---
- hosts: all
  tasks:
  - name: Install docker and its dependecies
    yum: name=docker-ce state=present
```

在ansible-demo.yaml同级目录下，新建hosts文件，内容：
```
[masters]
master ansible_host=192.168.1.146 ansible_user=root

[workers]
worker1 ansible_host=192.168.1.180 ansible_user=root
worker2 ansible_host=192.168.1.181 ansible_user=root
```

执行：`ansible-playbook -i hosts ansible-demo.yaml` # -i 指定host文件

- 问：`yum: name=docker-ce state=present`可以替换成`shell: yum install docker-ce -y`?

- 答：不好，建议使用yum模块而不是yum命令。yum命令可能进行重新安装升级版本。yum模块的`state=present`代表如果安装的软件存在则不进行安装，如果不存在则安装。

##### 示例1.2：安装软件-安装多个软件
- name: Install required dependiences
    yum: name={{ item }} state=present
    with_items:
      - yum-utils
      - device-mapper-persistent-data
      - lvm2

##### 示例2：复制文件-将ansible管理节点的文件通过ansible复制到hosts列表机器
编辑ansible-demo.yaml
```
---
- hosts: masters # 代表只操纵masters组下的机器
  tasks:
  - name: copy file to masters
    copy: 
      src: /opt/ansible-demo/hosts
      dest: /tmp/hosts
```
执行：`ansible-playbook -i hosts ansible-demo.yaml`

##### 示例3：重启服务-重启masters组的kubelet服务
编辑ansible-demo.yaml
```
---
- hosts: masters # 代表只操纵masters组下的机器
  tasks:
  - name: Restart kubelet
    service:
      name: kubelet
      state: restarted
```

##### 示例4：执行shell-执行单条shell命令
编辑ansible-demo.yaml
```
---
- hosts: masters 
  tasks:
  - name: Add docker repo /etc/yum.repos.d
    shell: yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
##### 示例5：执行shell-执行多条shell命令
编辑ansible-demo.yaml
```
---
- hosts: masters 
  tasks:
  - name: set bridge filter rules
    shell: |
      cat << EOF > /etc/sysctl.d/99-kubernetes-cri.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      user.max_user_namespaces=28633
      EOF
      sysctl -p /etc/sysctl.d/99-kubernetes-cri.conf
```

##### 示例6：执行shell-执行多条shell命令
编辑ansible-demo.yaml
```
---
- hosts: masters 
  tasks:
  - name: set bridge filter rules
    shell: |
      cat << EOF > /etc/sysctl.d/99-kubernetes-cri.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      user.max_user_namespaces=28633
      EOF
      sysctl -p /etc/sysctl.d/99-kubernetes-cri.conf
```

##### 示例7：保留结果-将命令执行后输出的内容保存到一个变量内 + 打印 + 输出到文件
编辑ansible-demo.yaml
```
---
- hosts: masters 
  tasks:
  - name: Extract the join command
    become: true
    command: "kubeadm token create --print-join-command"
    register: join_command # 存储变量
  - name: show join command
    debug:
      var: join_command  # 打印变量
  - name: Save kubeadm join command for cluster
    local_action: copy content={{ join_command.stdout_lines | last | trim }} dest=command.txt # 将join_command的内容处理后输出到文件并保存在当前主机的执行目录下
```


参考：https://austinsnerdythings.com/2022/04/25/deploying-a-kubernetes-cluster-within-proxmox-using-ansible/

https://getansible.com/README

