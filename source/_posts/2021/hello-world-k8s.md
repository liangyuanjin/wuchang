---
title: K8s 初遇见
date: 2021-03-17 08:52:19
tags: K8s
categories: K8s
description: 容器编排
cover: https://d19ta9rijs3cxg.cloudfront.net/wp-content/uploads/2020/03/Kubernetes-blog_680x451.png
---

## 简介

Kubernetes 是 `Google` 发起的一个开源项目，它的目标是为了`管理多个主机的容器`，`容器编排`，`自动部署`，`扩展和容器化应用程序`，是现在市场上主流的同期编排工具，主要实现语言为 `Go`。本地搭建一个 k8s 集群还是相对于比较复杂，所以我们现在 [katacoda](https://www.katacoda.com/courses/kubernetes) 上体验使用 K8s。

## katacoda 使用k8s

> katacoda 网站提了 k8s 的课程，可以在网站上帮我们启动一个 minikube 的环境用于学习， 登录上去会看到下面这个页面。
![k8s学习页面](/images/k8s/katacode学习页面.png)
开始一个 minikube 单阶段集群之旅吧,可以照着教程操作，这里就不在详细列举了

## K8s 基本概念与组件

---

### 集群

> 集群是一组节点，这些节点可以是物理服务器，也可以是虚拟机，这上面安装了 `Kubernetes` 环境

![集群详情图](/images/k8s/集群详情图.png)
**Master** `负责管理集群`, master 协调集群中的所有活动，调用应用程序，维护应用程序的所需状态，扩展和滚动更新等等，**节点** `Kubernetes集群中的工作机器可以是物理机或虚拟机`, 每个节点都有一个 `kubelet`， 管理节点并与 `Kubernetes Master` 节点进行通信的代理，Master 管理集群，而节点 用于托管正在运行时的应用程序

### Pod

Pod 是一组关系紧密的容器集合，他们`共享 PID, IPC, Network, UTS namespace` 是 Kubernetes `调度的基本单位`, Pod 是支持多个容器在一个 Pod 中共享网络和文件系统，可以通过进程间通信和共享的方式完成服务
![Pod详情图](/images/k8s/Pod内部详情图.png)

> 在 Kubernetes 中，所有对象都使用 `mainifest(yaml 或 json)` 来定义，下面是一个简单的 nginx 服务的yaml 文件定义，它包含一个镜像为 nginx 的容器

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

### Label

`Label` 是识别 Kubernetes 对象的标签，以 `key/value 的方式附加到对象上`，(key 最长不能超过63字节，value 可以为空，也可以是不超过253字节的字符串)， `Label 不提供唯一性`，并且实际上经常是很多对象都使用相同的label来标志具体的应用。

### Namespace

`Namespace` 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组，常见的 pods，services, deployments 等都是属于同一 namespace, 而Node， PersistentVolumes 等则不属于任何 namespace。

### Deployment

如果想要`创建同一个容器的多份拷贝`，需要一个一个分别创建出来，能否将 Pods `划到一个逻辑组里`，Deployment 确保任意时间都能有指定数量的 Pod `副本` 在运行，如果为某个 Pod 创建了Deployment 并且指定了3个副本，它会创建3个副本，它会创建3个副本，并且持续监控他们，如果某个 Pod 不响应，那么 Deployment 会替换它，保持总数为3

如果之前不响应的Pod恢复了，现在就有4个Pod了，那么Deployment 会将其中一个终止保持总数为3，如果在运行中将副本总数该为5，Deployment 会立即启动2个 Pod，保证总数为5， Deployment 还支持回滚和滚动升级。
当创建 Deployment时，需要指定两个东西:

* Pod 模版: 用来创建Pod副本的模版
* Label 标签: Deployment 需要监控的Pod的标签

> 我们已经创建了Pod一些副本，这些副本如何负载均衡，下面将介绍 Service

### Service

`Service` 是应用服务的抽象，通过labels 为应用提供负载均衡和服务发现，匹配 labels 的 Pod IP 和端口列表组成 endpoints，由 kube-proxy 负责将服务 IP 负载均衡到这些 endpoints 上。

每个 Service  都会自动分配一个 cluster IP 和 DNS 名，其他容器可以通过该地址或 DNS 来访问服务，而不需要了解后段容器的运行

![服务详情图](/images/k8s/Service详情图.png)

> 此文章学习K8s所记，来源于[优点知识](https://youdianzhishi.com/web),如有侵权，联系删帖
