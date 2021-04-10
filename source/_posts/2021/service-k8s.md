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
