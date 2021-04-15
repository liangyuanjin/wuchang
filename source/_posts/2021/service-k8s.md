---
title: Service
date: 2021-04-10 16:05:21
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
> `Pod` 的生命周期是有限的，死亡过后不会复活，`RC` 和 `Deployment` 可以动态创建和销毁`Pod`, 这样会出现我们重新启动了`Pod`的话，那么它的`IP`也可能会发生变化,如果一组后端的`Pod`的集合为集群中的其他前端 `Pod` 提供`API`服务，如果在前端的 `Pod` 中把所有的这些后端的 `Pod` 都写死，后端`Pod` 正常的话这样是没有问题的，但是如果后端`Pod`挂掉，然后重新启动了，`IP`地址很有可能会变化，这是前端就有可能访问不到后端的服务了

`解决办法`

> 解决办法有很多种，从早期的`nginx`负载均衡来处理，以及后面的 `服务发现`工具解决，`Consul`, `Zookeeper`, `etcd`, 我们只需要服务注册到服务发现中心就可以了，这些工具会动态的更新 `Nginx` 的配置，这样就不用手动去修改 `Nginx` 的配置文件

## Service

> `Kubernetes` 也为我们提供了这样一个资源对象，`Service`, 当我们的 `Pod` 被销毁或者新创建过后，我们可以把这个`Pod` 的地址注册到这个服务发现中心去就可以了，前端不用直接去连接后台的`Pod`集合，连接到 `Service` 上即可， `Service` 是一种抽象对象，它定义了一组 `Pod` 的逻辑集合和用于访问他们的策略，一个`Service` 下面包含的 `Pod`集合一般是由 `Label Selector` 来决定的，假如后端运行了很多副本，这些副本可能会发生变化，前端不需要关心这些变化，也不需要自己用一个列表来记录这些后端的服务, `Service` 可以帮我们达到解耦的目的

### Service 的三种 IP

* `Node IP`: `Node`节点的`IP`地址
* `Pod IP`: `Pod`的`IP`地址
* `Cluster IP`: `Service` 的 `IP` 地址

#### Node IP

> `Node IP` 是 `Kubernetes` 集群节点的物理IP地址, 属于这个网络之间的服务器之间可以直接通信，所以 `Kubernetes` 集群要想访问 `Kubernetes` 集群内部的某个节点或者服务，是通过 `Node IP` 进行通信的

#### Pod IP

> `Pod IP` 是每个`Pod` 的`IP`地址, 它是`Docker Engine` 根据 `docker0`网桥的IP地址段进行分配的

#### Cluster IP

> `Cluster IP` 是一个虚拟IP, 仅仅作用于 `Kubernetes Service` 这个对象， 由`Kubernetes` 自己进行管理和分配地址，无法ping这个地址。

### 定义Service

> `Service` 的定义方式和我们之前定义的各种资源对象方式类似，我们有一组 `Pod`， 对外暴露的端口是 `8080`, 同时都被打上了 `app=myapp` 的标签，那么我们可以定义出下面的 `YAML`文件

`Pod` 定义

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
          name: nginxweb
```

`Service` 定义

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: nginx
  ports:
  - name: mynginx-http
    protocol: TCP
    port: 80
    targetPort: nginxweb
```

### 创建 Service 对象

```bash
kubectl apply -f nginx-deploy.yaml
deployment.apps/nginx-deploy created

kubectl apply -f myservice.yaml
service/myservice created
```

> `myservice`的 `Service` 对象，他会将请求代理到使用 TCP 端口为 8080,同时具有标签 `app=myapp` 的`Pod`上，同时会被系统分配一个 `Cluster IP`, 而且`Service` 还会持续的监听 `selector` 下面的 `Pod`, 这些`Pod`的集合都被收集到 `myservice` 的 `Endpoints`对象上去

### 查看Service 对象

```bash
kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        4m52s
myservice          ClusterIP   10.100.215.23   <none>        80/TCP         90s
```

### 查看Service 详细

```bash
kubectl describe svc myservice
Name:              myservice
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"myservice","namespace":"default"},"spec":{"ports":[{"name":"myngi...
Selector:          app=nginx
Type:              ClusterIP
IP:                10.96.132.1
Port:              mynginx-http  80/TCP
TargetPort:        nginxweb/TCP
Endpoints:         172.18.0.6:80,172.18.0.7:80,172.18.0.8:80
Session Affinity:  None
```

> 可以看到 `Endpoints` 上已经收集了符合规则的`Pod`集合, TargetPort 可以是端口，也可以设置成一个字符串

### 验证访问 Service

```bash
kubectl run -it testservice --image=busybox /bin/bash
```