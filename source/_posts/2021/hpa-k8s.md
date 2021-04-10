---
title: HPA
date: 2021-04-10 10:25:26
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
> 我们可以通过 `Dashboard` 上操作`Pod`的扩缩容，但是在生产中这样手动的去操作肯定是不怎么显示的，因为我们并不清除业务请求什么时候达到需要扩容的时候，`Kubernetes` 为我们提供这样一个对象: `Horizontal Pod Autoscaling` 简称 `HPA`(Pod水平自动扩容), `HPA` 通过监控分析 `RC` 或者 `Deployment` 控制的所有 `Pod` 的负载变化情况来确定是否需要调整 `Pod` 的副本数量.

![HPA](/images/k8s/HPA.png)