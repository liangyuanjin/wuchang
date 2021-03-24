---
title: YAML 创建 Pod
date: 2021-03-23 08:20:24
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
上一篇我们已经对`YAML` 两种经常用的语法 `Maps`和 `Lists` 有了基本了解，下面我们可以使用 `YAML` 来创建 Pod

## YAML 定义 Pod 文件

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
spec:
  containers:
  - name: fronted-end
    image: nginx
    ports:
      - containerPort: 80
  - name: flaskapp-demo
    image: jcdemo/flaskapp
    ports:
      - containerPort: 5000
```

* `apiVersion`: 版本号，需要根据我们安装的kubernetes 版本和资源类型进行更改
* `kind`: 创建的资源类型， 这里我们创建的是一个 Pod, 也可以是其他，`Deployment`, `job`, `ingress`, `Service`。
* `metadata`: 包含我们定义的Pod的一些 meta 信息， 包含名称, namespaces, 标签等信息
* `spec`: 一些 containers, storage, volumes, 或者一些其他 kubernetes 需要知道的参数，以及是否在容器失败时重启容器的属性

### 容器的定义

```yaml
…spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
…
```
> 上面是一个典型的容器定义，一个名字 `fronted-end`, 基于 `nginx` 镜像， 以及`容器监听的端口`, 在这些参数中，只有名字是非常需要的，还可以指定一些更为复杂的属性，下面是一些可选的设置属性

* name
* image
* command
* args
* workingDir
* ports
* env
* resources
* volumeMounts
* livenessProbe
* readinessProbe
* livecycle
* terminationMessagePath
* imagePullPolicy
* securityContext
* stdin
* stdinOnce
* tty

## kubectl 创建 Pod

上面用于创建Pod的yaml 文件已经创建好，文件保存为 `pod.yaml` 我们可以用 `kubectl` 创建 Pod

```bash
kubectl create -f pod.yaml

pod "kube100-site" created
```

## 管理 Pod

Pod 现在已经创建成功，现在我们可以来查看 Pod 的状态

```bash
kubectl get pods

NAME           READY     STATUS    RESTARTS   AGE
kube100-site   2/2       Running   0          1m
```

### 查看pod 创建详情

在我们创建pod的时候有可能状态一直不是 `Running` 的状态，这是我们就需要查看创建 Pod 的详细记录，然后排查错误

```bash
kubectl describe pod kube100-site
```

### 删除 Pod

可以看到Pod 已经 running 起来了，我们可以使用 kubectl 删除上面的 Pod

```bash
kubectl delete -f pod.yaml
pod "kube100-site" deleted
```

## 创建 Deployment

上面的 `YAML` 文件只是一个单纯的 Pod 实例，但是如果这个 Pod 出现故障的话，意味着服务也就挂掉了，在 kubenetes 中提供了 `Deployment` 资源类型，可以让 kubenetes 管理一组 `Pod` 副本，也就是所谓的副本集，可以保证一定数量的副本一直可用，不会因为一个 Pod 挂掉而导致整个服务挂掉。

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube100-site
spec:
  replicas: 2
```

* `kind`: 指定的是 Deployment 资源类型
* `spec`: 指定了我们需要两个副本

完整的 `YAML` 格式文件如下

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube100-site
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: flaskapp-demo
          image: jcdemo/flaskapp
          ports:
            - containerPort: 5000
```

> 注意其中的 `template` 其实就是对 Pod 对象的定义， 将上面的 YAML 文件保存为 deployment.yaml, 然后创建 Pod

```bash
kubectl create -f deployment.yaml
deployment "kube100-site" created
```

查看状态， 检查 Deployment 的列表

```bash
kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube100-site   2         2         2            2           2m
```
