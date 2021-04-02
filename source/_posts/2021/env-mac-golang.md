---
title: Mac配置Go开发环境
date: 2021-03-30 11:10:39
tags: Go
categories: Go
description: Golang 环境搭建
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/golang/golang-framework-choose.jpeg
---

## 背景

> 目前我的主要开发语言是 `Python`, 当然 `Python` 在使用过程中还是很爽的, 但总还是想学习一门编译语言用于生产中， 由于舍友使用的 Golang 开发，这样我也就顺利入坑了 Golang。

## 安装 Go 语言开发环境

### 方式一, 官网下载

下载地址: [https://golang.google.cn/dl/](https://golang.google.cn/dl/)

![golang官网](/images/golang/golang官网.png)

> 下载 `Mac` 版本 `pkg` 安装包安装即可


### 方式二, Homebrew 安装

```bash
brew update
brew install golang
go version  # 安装完成了
go version go1.12.6 darwin/amd64
```

## GOPATH

> `GOPATH` 环境变量表示 Go的工作目录， 这个目录指定需要从那个地方寻找GO的包，可执行程序等，使用 `go get` 下载的包都放在这个目录下，Go语言1.14版本之后推荐使用`go modules`管理以来，也不再需要把代码写在GOPATH目录下了。

配置 GOPATH

> GOPATH默认是$HOME/go，如果你希望使用其他目录可以在你用的Shell配置文件(如~/.zshrc、~/.bashrc、.bash_profile)里面指定，我推荐显示的指定GOPATH，哪怕用了默认的目录。

## GOROOT

> `GOROOT` 是指 Go 语言编译、工具、标准库等安装路径，不必显示地设置GOROOT环境变量。通常这个路径是/usr/local/go，当使用Homebrew安装Golang时目录会放在”$(brew –prefix golang)/libexec”这个目录下。

.zshrc 配置

```bash
# GOROOT
export GOROOT=/usr/local/go
# GOPATH 这个目录可以自定
export GOPATH=$HOME/Documents/Go
# GOPATH目录和GOROOT下面的bin目录加入系统环境变量中
export PATH=$PATH:$GOPATH/bin:$GOROOT/bin
```

## GOPROXY

> `Go1.14`以后，建议使用 `go mod` 模式来管理依赖环境， 也不再强制我们把代码写在 `GOPATH`下面的 `src`目录下，你可以在你的电脑的任意位置编写 Go 代码

修改 GOPROXY

> 默认GoPROXY配置是：`GOPROXY=https://proxy.golang.org,direct`，由于国内访问不到`https://proxy.golang.org`，所以我们需要换一个PROXY，这里推荐使用`https://goproxy.io`或`https://goproxy.cn`。

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

## Go Module 依赖管理

---

> `go module` 是 `go 1.11` 版本之后出来的 版本管理工具, 从`go 1.13`开始， `go module` 将是默认的依赖管理

### go mod 命令

```bash
go mod download    下载依赖的module到本地cache（默认为$GOPATH/pkg/mod目录）
go mod edit        编辑go.mod文件
go mod graph       打印模块依赖图
go mod init        初始化当前文件夹, 创建go.mod文件
go mod tidy        增加缺少的module，删除无用的module
go mod vendor      将依赖复制到vendor下
go mod verify      校验依赖
go mod why         解释为什么需要依赖
```

### go.mod

```text
module github.com/wuchang/blogger

go 1.14

require (
    github.com/DeanThompson/ginpprof v0.0.0-20190408063150-3be636683586
    github.com/gin-gonic/gin v1.4.0
    github.com/go-sql-driver/mysql v1.4.1
    github.com/jmoiron/sqlx v1.2.0
    github.com/satori/go.uuid v1.2.0
    google.golang.org/appengine v1.6.1 // indirect
)
```

* `module`: 用来定义包名
* `require`: 用来定义依赖包及版本
