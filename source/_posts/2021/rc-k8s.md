---
title: Replication Controller 和 ReplicaSet
date: 2021-03-29 08:38:33
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
## 背景

> 前面都只是使用了 `Pod` 的一些基本使用方法，都是直接来操作 `Pod`，在生产中有一些业务场景并不适合直接操作 `Pod`, 比如 `某次运营活动非常成功，网站访问量突然暴增`， `运行当前 Pod 的节点发生故障了， Pod 不能正常提供服务了`。这些不能靠我们人工的去解决这些问题，我们希望 `Pod` 不够自动帮我门增加一个，`Pod` 挂了自动帮我们在合适的节点上重新启动一个 `Pod`, kubernetes 其实为我们提供了这样的资源对象

* `Replication Controller`: 用来部署，升级 `Pod`
* `Replica Set`: 下一下的 `Replication Controller`
* `Deployment`: 可以更加方便的管理 `Pod` 和 `Replica Set`

## Replication Controller (Rc)

> `Replication Controller` 简称 `RC`, `RC` 是 `Kubernetes` 系统中的核心概念之一，`RC` 可以保证在任意时间运行 `Pod` 的副本数量， 能够保证 `Pod` 总是可用的， 如果实际 `Pod` 数量比指定的多那就结束掉多余的，如果实际数量比指定的少就启动一些 `Pod`， 当 `Pod` 失败，被删除，或者挂掉后，`RC` 都会去自动创建新的 `Pod`来保证副本数量，所以即使只有一个 Pod, 也应该用 `RC` 来管理我们的 Pod 

### Rc 示例

```yaml
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-demo
  labels:
    name: rc
spec:
  replicas: 3
  selector:
    name: rc
  template:
    metadata:
     labels:
       name: rc
    spec:
     containers:
     - name: nginx-demo
       image: nginx
       ports:
       - containerPort: 80
```

* `kind`: `ReplicationContainer`
* `spec.replicas`: 指定 `Pod` 副本数量, 默认为 1
* `spec.selector`: `RC` 通过该属性来筛选要控制的 `Pod`
* `spec.template`: 就是我们之前`Pod` 定义的模块，不需要 `apiVersion` 和 `kind`
* `spec.template.metadata.labels`: 注意这里的 `Pod` 和 `labels` 要和 `spec.selector` 相同， 这样 `RC` 就可以控制当前这个 `Pod`

> `YAML` 文件 定义了一个 `RC` 资源对象，它的名字叫 `rc-demo`, 保证一直会有3个 `Pod` 运行，`Pod`的镜像就是 `nginx` 镜像

创建 Pod

```bash
kubectl create -f rc-demo.yaml
```

查看服务 `RC`

```bash
kubectl get rc
```

查看具体信息

```bash
kubectl describe rc rc-demo
```

### 修改副本数

修改副本数量，直接修改 `YAML`中的 `spec.replicas` 属性, 然后使用下面的方式生效

```bash
kubectl apply -f rc-demo.yaml
```

或者

```bash
kubectl edit rc rc-demo
```

### 滚动更新

可以用 `RC` 来进行滚动升级，比如修改镜像地址

```bash
kubectl rolling-update rc-demo --image=nginx:1.7.9
```

> 如果我们 `Pod` 中有多个容器的话，就需要通过修改 `YAML` 配置文件来进行修改了

```bash
kubectl rolling-update rc-demo -f rc-demo.yaml
```

## Replication Set (RS)

> `Replication Set` 简称 `RS` 官方现在已经推荐使用 `RS` 和 `Deployment` 来代替 `RC`， `RC` 和 `RS`的功能基本一致, 区别在于 `RC`只支持基于等式的 `selector` (env=dev 或 enviroment != qa), 但 `RS` 还支持基于集合 `selector` (version in (v1.0, v2.0))

> 在实际开发中，我们很少去使用 `RS` 资源对象，它主要被 `Deployment` 资源对象使用，一般情况下，推荐使用 `Deployment` 而不是直接使用 `Replication Set`

## RC/RS特性和作用

* 我们可以通过定义一个 `RC` 实现 `Pod` 的创建和副本数量的控制
* `RC` 中包含一个完整的 `Pod` 定义模块(不包含`apiVersion`, 和 `kind`)
* `RC` 是通过 `label selector` 机制来实现对 `Pod` 副本的控制
* 通过改变 `RC` 里面的 `Pod` 副本数量，可以实现 `Pod` 的扩缩容功能
* 通过改变 `RC` 里面的 `Pod` 模版中镜像版本，可以实现 Pod 的滚动升级功能(但是不支持一键回滚)