---
title: kubeadm 搭建单节点 K8s
date: 2021-03-18 18:36:27
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---

## 硬件配置

| 系统 | 角色 | IP | 内存 |
| ---- | ---- | --- | --- |
| CentOs7.8 | master | 172.17.0.3 |  2核4GB |
| CentOs7.8 | node | 172.17.0.4 |  2核4GB |
| CentOs7.8 | node | 172.17.0.5 |  2核4GB |

## 系统设置

### 修改节点名称

```bash
yum update -y
hostnamectl  set-hostname  k8s-master

sudo vim /etc/hosts

172.17.0.3 k8s-master
172.17.0.4 k8s-node1
172.17.0.4 k8s-node2
```

> `master` hostname `k8s-master`, `node` hostname `k8s-node...`


### 安装依赖

```bash
yum install -y conntrack ntpdate ntp ipvsadm ipset jq iptables curl sysstat libseccomp wget vimnet-tools git
```

### 设置防火墙为Iptables并设置规则

```bash
systemctl  stop firewalld  &&  systemctl  disable firewalld
yum -y install iptables-services  &&  systemctl  start iptables  &&  systemctl  enable iptables &&  iptables -F &&  service iptables save
```

### 关闭SELINUX分区

```bash
swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
setenforce 0 && sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```

### 关闭邮件服务

```bash
systemctl stop postfix && systemctl disable postfix
```

### 设置 rsyslogd 和 systemd journald

```bash
# 持久化保存日志的目录
mkdir /var/log/journal
mkdir /etc/systemd/journald.conf.d

cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent
# 压缩历史日志
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000 # 最大占用空间10G
SystemMaxUse=10G # 单日志文件最大200M
SystemMaxFileSize=200M
# 日志保存时间 2 周
MaxRetentionSec=2week
# 不将日志转发到syslog
ForwardToSyslog=no
EOF

systemctl restart systemd-journald
```

### 同步时间

```bash
# 同步时间
yum install -y ntpdate
# 设置时区
timedatectl set-timezone Asia/Shanghai

# 检查服务器时间是否同步
date
```

### 升级系统内核为 4.44

```bash
# 安装完成后检/boot/grub2/grub.cfg 中对应内核menuentry 中是否包含initrd16 配置，如果没有，再安装一次！
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

# 设置开机从新内核启动
yum --enablerepo=elrepo-kernel install -y kernel-lt

# 查看默认启动顺序
awk '$1=="menuentry" {print $2,$3,$4}' /etc/grub2.cfg

# 设置默认启动项，0是按menuentry顺序。比如要默认从第四个菜单项启动，数字改为3，若改为 saved，则默认为上次启动项。
sed -i "s/GRUB_DEFAULT=saved/GRUB_DEFAULT=0/g" /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg

# 重启机器
reboot

# 查看 内核
uname -r
```

> **如果内核高于 `4` 版本可以不用升级**

## 调整内核参数,对于 k8s

```bash
cat > kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1 # 不检查物理内存是否够用
vm.panic_on_oom=0 # 开启 OOM
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF

cp kubernetes.conf /etc/sysctl.d/kubernetes.conf

sysctl -p /etc/sysctl.d/kubernetes.conf
```

## kube-proxy开启ipvs的前置条件

```bash
modprobe br_netfilter
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules &&
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

## 安装 Docker

```bash
# 如果已经安装，需要卸载的话，使用以下命令
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \

# 安装依赖包
yum install yum-utils device-mapper-persistent-data lvm2

# 设置稳定的仓库
yum-config-manager --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker CE.
yum update && yum install \
  containerd.io-1.2.10 \
  docker-ce-19.03.4 \
  docker-ce-cli-19.03.4

## 创建 /etc/docker 目录
mkdir /etc/docker
# 配置 daemon.
cat > /etc/docker/daemon.json <<EOF
{
"exec-opts": ["native.cgroupdriver=systemd"],
"registry-mirrors": ["https://7w2ycc3f.mirror.aliyuncs.com"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
}
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# 重启docker服务
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```

## 安装 kubeadm

### 设置 kubeadm 下载地址

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 安装 kubelet kubeadm kubectl

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 设置开机启动
systemctl enable --now kubelet

```

## kubeadm 初始化 k8s 集群

---
> 在 Master 节点服务器上 初始化集群

### 初始化配置文件

```bash
kubeadm config print init-defaults --kubeconfig ClusterConfiguration >  kubeadm-config.yaml
```

> 修改配置文件 `kubeadm-config.yaml`, 修改`镜像地址`，使用的是国内镜像仓库, `Pod所在网段` ,如果你服务器所在的集群网段不和 192.168.0.0/16 冲突，可以不修改这个网段地址，使用默认即可, 完整配置文件如下。

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  # 修改为主节点IP
  advertiseAddress: 172.17.0.3
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
# 修改镜像仓库
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.0
networking:
  # 配置 pod 所在网段和虚拟机所在网段不重复（这里用的是Flannel 默认网段），如果宿主机已经使用该网段，则必须更改
  podSubnet: 10.244.0.0/16
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

### 初始化

```bash
# 查看需要下载镜像
kubeadm config images list --config kubeadm-config.yaml

# 步骤一：拉取镜像
kubeadm config images pull --config kubeadm-config.yaml

# 步骤二：初始化k8s
kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.log
```
> 如果是安装成功会看到日志中输出 `Your Kubernetes control-plane has initialized successfully!`

### 创建 kubectl

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 记录加入Master节点的token

```bash
kubeadm join 172.17.0.3:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:b0e18c9900ff18103df0c36da270a425f17b8269149e418d7d778828cd8a82ba

```

> 初始化的日志会保存在 `kubeadm-init.log` 中，方便后期查看

## 整理安装脚本

```bash
mkdir -p install-k8s/core
# 移动安装yaml文件和init日志
mv kubeadm-config.yaml kubeadm-init.log ./install-k8s/core/
mv install-k8s /usr/local/  # 将配置文件移到 /user/local/ 目录下
```

## 部署网络

```bash
mkdir -p install-k8s/plugin/flannel/
cd install-k8s/plugin/flannel/
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 安装 flannel
kubectl create -f kube-flannel.yml
# 查看 flannel状态
kubectl get pod -n kube-system
```

![kubernetes 运行](/images/k8s/k8s运行.png)

## 安装 Dashboard 仪表盘

```bash
mkdir -p /usr/local/install-k8s/plugin/dashboard/
cd /usr/local/install-k8s/plugin/dashboard/
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

> 为了测试方便，我们将Service改成NodePort类型，注意 YAML 中最下面的 Service 部分新增一个type=NodePort

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard
```

### 部署 Dashboard

> 部署成功如下图显示

```bash
kubectl apply -f recommended.yaml

kubectl get pods -n kubernetes-dashboard

kubectl get pods,svc -n kubernetes-dashboard
```

![Dashboard](/images/k8s/dashboard.png)

### 生成token

```bash
cd /usr/local/install-k8s/plugin/dashboard/
vim auth.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

执行如下命令可创建 ServiceAccount 和 ClusterRoleBinding

```bash
kubectl apply -f auth.yaml
```

获取Bearer Token

```bash
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

token详情如下
![Dashboard_token](/images/k8s/dashboard-token.png)

### 更换 Dashboard 证书

> 虽然上面显示安装成功 dashboard 但是用浏览器打开显示不是私密链接
![Dashboard_ssh](/images/k8s/dashboard-ssh.png)
