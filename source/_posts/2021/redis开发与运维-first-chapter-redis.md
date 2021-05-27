---
title: Redis开发与运维 第一章
date: 2021-05-13 08:27:35
tags:
  - Redis
  - Book
categories: Book
description: Redis  阅读
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2020/04/02/22/09/santorini-4996846_1280.jpg
---

## Chapter 1: 初始Redis

### 1.1 盛赞Redis

> Redis 是一种基于`键值对(key-value)`的 `NoSQL` 数据库, `Redis`的值可以是 `string`, `list`, `hash`, `set`, `zset`, `Bitmaps`(位图), `HyperLogLog`(地理信息定位) 等多种数据结构和算法组成,
`Redis` 所有数据都放在内存中,读写性能非常优秀，`Redis` 还可将内存中的数据利用快照和日志的形式保存到硬盘上, 此外`Redis`还提供了`键过期`，`发布订阅`，`事物`，`流水线`，`Lua脚本附加功能`, 总之就是Redis 很牛。。。。

### 1.2 Redis特性

> `Redis` 之所以被管饭应用, 主要是具备了以下特性

#### 速度快

* `Redis` 所有的数据都是放在内存中的，这是 Redis 速度快的主要原因
* `Redis` 是用 C 语言实现的，C语言实现的程序距离操作系统更近，执行速度相对更快
* `Redis` 使用了单线程，预防了多线程可能产生的竞争问题
* `Redis` Redis 作者对于源代码的精打细磨

#### 基于键值对的数据结构服务器

* 数据类型丰富，开发者可以根据数据类型实现各种的业务需求

#### 丰富的功能

> Redis 除了提供物种基本数据类型，还提供了很多额外的功能

* 提供了键过期的功能，可以用来是实现缓存
* 提供了发布与订阅功能，可以用来实现消息系统
* 支持 Lua 脚本功能，可以利用Lua创造新的 Redis命令
* 提供了简单的事物功能，能在一定程度上保证事物的特性
* 提供了流水线（Pipline）功能，客服端能将一批命令一次性的传到 Redis， 减少了网络的开销

#### 简单稳定

* Redis 源码很少， 早起只有2万行左右
* Redis 使用单线程模型
* Redis 不需要依赖于操作系统中的类库，`Memcache` 需要依赖与`libevent`这样的类库

#### 客服端语言多

> Redis 提供了简单的 TCP 通信协议，主流语言似乎都支持Redis客户端

#### 持久化

> 将数据放在内存中是不安全的，可能回出现数据丢失，Redis 提供了两种持久化方式, `RDB` 和 `AOF` 两种策略方式将内存中的数据存储到硬盘上

#### 主从赋值

> Redis 提供了主从复制功能, 实现了多个相同数据的Redis副本,复制功能是分布式 Redis的基础

#### 高可用和分布式

> Redis 从 `2.8` 版本正式提供了高可用实现 `Redis Sentinel`, 它能保证Redis节点的故障发现和故障自动转移, 从 `3.0` 版本提供了分布式实现 `Redis Cluster`

### 1.3 Redis 使用场景

#### Redis 可以做什么

* 缓存
  * 合理的使用缓存不仅可以加快数据的访问速度, 而且能够有效的降低后端数据源的压力
* 排行榜系统
  * Redis 提供了列表和有序集合数据，合理的使用这些数据能够很方便的构建各种排行榜系统
* 计数器的应用
  * Redis 天然支持计数功能而且计数性能也非常好，可以说是计数器系统的重要选择
* 社交网络
  * 赞/踩，共同好友/喜好，这种类型的数据，传统的关系型数据库不太适合这种数据类型，Redis的数据结构可以相对容易的实现这些功能
* 消息队列系统
  * 消息队列能够业务解耦合，非实时业务削峰，Redis提供了发布订阅功能和阻塞队列功能

#### Redis 不可以做什么

* `数据规模的角度`, 数据可以大规模数据和小规模数据，数据存放在内存中，数据量很大的话，`经济成本很高`
* `数据的冷热角度`, Redis 通常存放热数据, 不适宜存放冷数, 存放大量的冷数据其实是对内存的一种浪费

### 1.4 安装 Redis

#### Linux 安装

> Redis在 Linux 安装通常有两种方式, 第一种是通过操作系统的软件相对较快，但是缺点是不一定能更新到最新的版本，Redis 安装方式并不负载，一般推荐源码进行安装

```shell
wget http://download.redis.io/releases/redis-3.0.7.tar.gz
tar xzf redis-3.0.7.tar.gz
ln -s redis-3.0.7 redis
cd redis
make
make install
```

> 在上面安装中，建立了一个redis的目录的软连， 这样做是为了不把 redis 目录固定在指定版本上, 有利于 Redis 未来版本升级,Redis 的安装是将 Redis 的相关运行文件放在 `/usr/local/bin` 下

#### 配置、启动、操作、关闭Redis

> Redis 安装之后， src和/usr/local/bin 目录下多了几个以 redis 开头的可执行文件，称为 Redis Shell， 这些 Shell 文件可以做很多事，启动、停止、检测和修复Redis持久化文件、检测Redis的性能

|  可执行文件   | 作用  |
|  ----  | ----  |
| redis-server  | 启动Redis |
| redis-cli  | Redis 命令行客户端 |
| redis-benchmark  | Redis 基准测试工具 |
| redis-check-aof | Redis AOF 持久化文件检测和修复工具 |
| redis-check-dump  | Redis RDB 持久化文件检测和修复工具 |

##### 启动Redis

> Redis 有三种方法启动 Redis: 默认配置、运行配置、配置文件启动
**默认配置**

* 这种方式会使用Redis默认配置来启动文件， 然后输出相关日志, 执行 `redis-server` 后输出相关日志
**运行启动**
* `redis-server` 加上要修改的配置名和值(可以是多对), 没有设置的配置将使用默认配置

```shell
redis-server --configKey1 configValue1 --configKey2 configValue2
# 例如要用 6380 作为端口启动 Redis 可以执行如下
redis-server --port 6380
```

> 虽然运行配置可以自定义配置, 但是如果需要修改的配置文件较多或者希望将配置保存到文件中，不建议使用这种方式
**配置文件启动**

* 将配置文件写到执行的文件中, 例如将配置文件写到 `/opt/redis/redis.conf` 中，以下面的启动方式启动即可

```shell
redis-server /opt/redis/redis.conf
```

> 下面是一些 redis 的重要配置

|  配置名   | 配置说明  |
|  ----   | ----  |
|  port   | 端口  |
|  logfile   | 日志文件  |
|  dir   | Redis 工作目录(存放持久化文件和日志文件)  |
|  daemonize   | 是否以守护进程的方式启动Redis  |

> Redis 目录下都有有一个 redis.conf 配置文件，就是 Redis 默认配置，配置文件不是完全手写的，而是将redis.conf 作为模版进行修改

##### Redis 命令行客户端

> 已经启动的 Redis 可以通过 redis-cli 连接，操作Redis服务，redis-cli 可以使用两种服务连接Redis服务器
**交互式方式**
> 通过 `redis-cli-h{host} -p {port}` 的方式连接到Redis服务

```shell
redis-cli -h 127.0.0.1 -p 6379
```

**命令方式**
> `redis-cli-h ip{host} -p{port} {command}`, 就可以直接得到命令行的返回结果

```shell
redis-cli -h 127.0.0.1 -p 6379 get hello
"world"
```
