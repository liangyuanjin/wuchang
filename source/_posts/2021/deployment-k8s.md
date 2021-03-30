---
title: Deployment
date: 2021-03-30 08:35:32
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
## 背景

> `RC` 和 `RS` 两种资源对象的功能基本上是差不多的，唯一的区别是在 `selector` 上 `RS` 支持集合，但是在生产中更推荐用 `Deployment` 这种资源对象来管理我们的 `Pod`,下面就详细说明为什么推荐用 `Deployment` 这种资源对象为什么比 `RC` 和 `RS` 更好

> `RC` 是 `Kubernetes` 的一个核心概念，当我们把应用服务部署到集群中去，要保证应用能够持续稳定的运行，`RC` 就是这个保证的关键它能提供以下功能

* 确保 `Pod`数量: 它会确保 `Kubernetes` 中有指定数量的 `Pod` 在运行，如果少于指定的 `Pod`,`RC` 就会创建新的，反之会删除多余的，保证 `Pod` 的副本数量不变

* 确保 `Pod` 健康: 当`Pod`运行出现错误，不能提供服务了，`RC` 会杀死不健康的 `Pod` 重新创建新的

* 弹性伸缩: 在业务高峰到来之前，可以通过 `RC` 来调整 `Pod` 的数量，来确保服务的正常访问

* 滚动升级: 滚动升级是一种平滑的升级方式，通过逐步替代的策略，保证整体系统的稳定性

## Deployment

> `Deployment` 同样也是 `Kubernetes` 系统的一个核心概念，主要职责和 `RC` 一样的都是保证 `Pod` 的数量和健康，基本功能都是完全一直的，可以看成是一个升级版的 `RC` 控制器，它和 `RC` 对比新的特性如下:

* `RC` 的全部功能: `Deployment` 具备上面描述的 `RC` 的全部功能

* 事件和状态查看: 可以查看 `Deployment` 的升级详细进度和状态

* 回滚: 当升级 `Pod` 的时候如果出现问题，可以使用回滚操作恢复到之前的任一版本

* 版本记录: 每一次对 `Deployment` 的操作，都能够保存下来，这也是保证可以回滚到任一版本的基础

* 暂停和启动: 对于每一次升级都能够随时暂停和启动

> `Deployment` 不仅在功能上更为丰富，官方也是推荐 `Deployment` 来管理 `Pod`，一些官方的组件 `kube-dns`, `kube-proxy` 也都是使用 `Deployment` 来管理的

## Deployment 示例

### 定义配置文件

> 一个 Replica Set 拥有一个或者多个 Pod, 一个 `Deployment` 控制多个 `RS` 主要是为了支持回滚机制，每当`Deployment` 操作时， `Kubernetes`会重新生成一个 `Replica Set` 并保留，后面有需要的话就可以回滚到之前的状态

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    k8s-app: nginx-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

### 创建 Pod

```bash
kubectl create -f nginx-deployment.yaml
deployment "nginx-deploy" crea
```

### 查看 Deployment

```bash
kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy       3/3     3            3           24s
```

> `Deployment` 已经创建了 `1` 个 `Replica Set`, 可以 查看 `RS` 和 `Pod`

```bash
kubectl get rs
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deploy-54f57cf6bf      3         3         3       39s
```

```bash
kubectl get pod --show-labels
NAME                               READY   STATUS    RESTARTS   AGE   LABELS
nginx-deploy-54f57cf6bf-685fz      1/1     Running   0          50s   app=nginx,pod-template-hash=54f57cf6bf
nginx-deploy-54f57cf6bf-s5hs4      1/1     Running   0          50s   app=nginx,pod-template-hash=54f57cf6bf
nginx-deploy-54f57cf6bf-w9zd7      1/1     Running   0          50s   app=nginx,pod-template-hash=54f57cf6bf
```

> `Deployment` 的 `YAML` 的配置文件中 `replicas: 3` 将会保证我们始终有 3 个 Pod 在运行

