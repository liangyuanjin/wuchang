---
title: docker stats 与 docker logs
date: 2021-04-09 09:08:07
tags: Docker
categories: Docker
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2020/03/06/08/43/digital-paper-4906487_1280.jpg
---

## 背景

> 发现线上的服务器 `CPU` 占用过高, 部署在服务器上的项目都是用 `Docker` 部署的，我们想要知道某个容器占用系统的资源的详情，才能排查出问题出在那个项目上。

## docker stats

> 在容器运行过程中， 我们想要知道容器使用的系统资源，`Docker` 为我们提供了 `docker stats` 命令来查看容器占用资源的情况, 命令参数如下

```bash
[root@VM_16_57_centos ~]# docker stats --help

Usage:	docker stats [OPTIONS] [CONTAINER...]

Display a live stream of container(s) resource usage statistics

Options:
  -a, --all             Show all containers (default shows just running)
      --format string   Pretty-print images using a Go template
      --help            Print usage
      --no-stream       Disable streaming stats and only pull the first result
```

执行命令会得到关于容器一些信息

```bash
[root@VM_16_57_centos ~]# docker stats --no-stream
CONTAINER           CPU %               MEM USAGE / LIMIT       MEM %               NET I/O             BLOCK I/O           PIDS
```

* `CONTAINER`: 容器ID
* `CPU`: CPU 的使用情况
* `MEM USAGE / LIMIT`: 当前使用的内存和最大可以使用的内存
* `MEM `: 以百分比的形式显示内存使用情况
* `NET I/O`: 网络 I/O 数据
* `BLOCK I/O`: 磁盘 I/O 数据
* `PIDS`: PID 号

## docker logs

> 通过 `docker logs`命令可以查看容器的日志，可以通过参数查看某一时间段的日志

```bash
[root@VM_16_57_centos ~]# docker logs --help

Usage:	docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --help           Print usage
      --since string   Show logs since timestamp
      --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
```

查看指定时间后的日志，只显示最后100行:

```bash
docker logs -f -t --since="2021-02-01" --tail=100 CONTAINER_ID
```

查看某时间段日志:

```bash
docker logs -t --since="2021-02-01T14:30:30" --until "2021-03-30T14:40:50" CONTAINER_ID

```

查看某时间段日志:

```bash
docker logs -t --since="2021-02-01T14:30:30" CONTAINER_ID
```

查看最近60分钟的日志:

```bash
docker logs --since 60m CONTAINER_ID
```
