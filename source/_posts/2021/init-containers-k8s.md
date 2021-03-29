---
title: 初始化容器
date: 2021-03-28 10:35:45
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
## 初始化容器

> 容器的健康检查可以用`探针`来解决，`liveness probe`(存活探针)和`readiness probe`(可读性探针)， 探针和容器的两个钩子函数是可以影响到容器的生命周期，本篇介绍的是 `Init Container` (初始化容器)

初始化容器
> `Init Container` 是用来做`初始化工作的容器`，可以是一个或者多个，这些容器会定义的顺序依次执行，只有所有的`Init Container` 执行完毕后，主容器才会被启动，由于`Pod` 里面的所有容器是共享数据卷和网络命名空间的，所以 `Init Container` 立面产生的数据可以被主容器使用到
![pod生命周期](/images/k8s/Pod生命周期.png)

* `PostStart`和`PreStop`和 `liveness` 和 `readiness` 是属于主容器的生命周期范围内的

* `Init Container` 是独立于主容器之外的，他们都属于 `Pod` 的生命周期范围之内.

### Infra 容器

> 上面图中还包含一个 `infra` 的容器，集群中的每一个 `Pod` 对应运行的 `Docker` 容器，都包含一个 `pause-amd64` 的镜像，这就是我们的 `infra` 镜像， `Pod` 下面的所有容器是共享一个网络命名空间的，这个镜像就是来做这个是的

### Init Container 应用场景

* 等待其他模块Ready: 解决服务之间的依赖问题
* 初始化配置: 集群中检测所有以存在的成员节点，为主容器准备好就去那的配置信息，这样主容器起来后就能用这个配置信息加入集群

初始化容器定义方法

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: init-pod1
  labels:
    app: init
spec:
  containers:
  - name: init-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 6377
```