---
title: kubeadm 搭建单节点 K8s
date: 2021-03-18 18:36:27
categories: K8s
description: 容器编排
cover: https://d19ta9rijs3cxg.cloudfront.net/wp-content/uploads/2020/03/Kubernetes-blog_680x451.png
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
