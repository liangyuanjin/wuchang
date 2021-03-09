---
title: Docker 搭建 Jenkins
date: 2021-03-09 08:31:24
tags:
categories: Linux
description: 技术分享
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://user-gold-cdn.xitu.io/2020/6/24/172e537ca5ea910e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1
---
## 背景

> 在日常开发中，项目发布与部署，版本的迭代部署，隐形中占据了很长时间，特别是一些前端项目，样式的更改很频繁，这就需要我们频繁的发布版本，有没有一种很好的解决方式呢？Jenkins 就是这样一个持续平台，主要包含 `普通项目构建`,`流水线构建`, `多分支流水构建`。

## 准备工作

由于在项目部署中服务器多是 `CentOs`，在这里我也采用 `CentOS` 搭建

## 前提条件

* 检查是否安装了 `Docker`,如果本机没有请移步[Docker安装](https://docs.docker.com/get-docker/)

