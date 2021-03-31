---
title: Linux中buff-cache占用过高
date: 2021-03-31 15:58:32
tags: Linux
categories: Linux
description: linux 命令详解
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2021/03/06/11/28/ships-6073537_1280.jpg
---

## 背景

---
> 在服务器运行中 `内存` 是一个非常重要的指标，如果内存占用过高，会导致服务不可用，今天发现某一个服务挂掉了，然后上服务器查看，可用可用内存已经没有多少了，同时 `buff/cache` 占用过高。

## 查看内存使用情况

> 我们可以使用 `top` 或者 `free -h`命令查看当前系统的内存使用情况

```bash
[root@VM_16_58_centos ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           7.6G        5.7G        1.5G        2.6M        522M        1.7G
Swap:            0B          0B          0B
```

* `available`: 表示应用程序还可以申请到的内存

## buff 和 cache

`buff`
> `buff(buffer cache)`: 是一种 I/O 缓存，用于内存和硬盘的缓冲，是io设备的读写缓冲区，根据磁盘的读写设计，把分散的写操作集中进行，减少磁盘碎片和硬盘的反复寻道，从而达到提高系统性能

`cache`
> `cache(page cache)`: 是一种高速缓存，用于 CPU和内存之间的缓冲，是文件系统的cache,把读取过来的数据保存起来，重新读取时找到需要的数据，就不要区读硬盘了，若没有命中就读硬盘。其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不再读的内容不断往后排，直至从中删除, `通常我们在频繁存取文件后，会导致buff/cache的占用量增高。`

## 解决方式

### 手动清除

```bash
[root@VM_16_58_centos ~]# sync
[root@VM_16_58_centos ~]# echo 1 > /proc/sys/vm/drop_caches
[root@VM_16_58_centos ~]# echo 2 > /proc/sys/vm/drop_caches
[root@VM_16_58_centos ~]# echo 3 > /proc/sys/vm/drop_caches
```

* `sync`：将所有未写的系统缓冲区写到磁盘中，包含已修改的i-node、已延迟的块I/O和读写映射文件

* `echo 1 > /proc/sys/vm/drop_caches`：清除page cache

* `echo 2 > /proc/sys/vm/drop_caches`：清除回收slab分配器中的对象（包括目录项缓存和inode缓存）。slab分配器是内核中管理内存的一种机制，其中很多缓存数据实现都是用的pagecache。

* `echo 3 > /proc/sys/vm/drop_caches`：清除pagecache和slab分配器中的缓存对象。
/proc/sys/vm/drop_caches的值,默认为0

### 定时清除

创建 clean-cache.sh

```bash
#!/bin/bash
# 每两小时清除一次缓存
sync;
# 写入硬盘，防止数据丢失
sleep 10
# 延迟10秒
echo 1 > /proc/sys/vm/drop_caches
echo 2 > /proc/sys/vm/drop_caches
echo 3 > /proc/sys/vm/drop_caches
```

创建定时任务

```bash
crontab -c
```

添加定时任务

```bash

0 */2 * * * /opt/crontab-task/clean-cache.sh
```

设置 crond 启动以及开机自启

```bash
systemctl start crond.service
systemctl enable crond.service
```

查看定时任务是否被执行

```bash
cat /var/log/cron | grep cleanCache
```
