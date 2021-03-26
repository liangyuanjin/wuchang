---
title: Pod 健康检查
date: 2021-03-26 08:31:52
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
> 在 `kubernetes` 集群中，我们可以通过配置 `liveness probe` (存活探针) 和 `readiness probe`(可读性探针) 来影响容器的生存周期，

## liveness probe

> kubelet 通过使用 `liveness probe` 来确定你的应用程序是否正在运行， 如果程序一旦崩溃了，kubernetes 就会立刻知道这个程序已经终止了，然后就会重启这个程序，而 `liveness probe` 的目的就是来捕获到当前应用程序的状态，如果崩溃，那么就会重启处于该状态下的容器，使应用程序在存在 bug 的情况下依然能够继续运行下去。

## readiness probe

> `readiness probe` 来确定容器是否已经就绪，可以接收流量过来了，只有当 Pod 中的容器都处于就绪状态的时候，kubelet 才会认定该 Pod 处于就绪状态，因为一个 Pod 下可能有多个容器。

## 探针的配置方式

* exec: 执行一段命令
* http: 检测某个 http 请求
* tcpSocket: kubelet 将尝试在指定端口上打开容器的套接字。如果可以建立连接，容器被认为是健康的，如果不能就认为是失败的。实际上就是检查端口

## exec 示例

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
  labels:
    test: liveness
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
