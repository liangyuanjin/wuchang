---
title: Docker 搭建 Jenkins
date: 2021-03-09 08:31:24
tags: 平台搭建
categories: Linux
description: 技术分享
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2021/03/04/07/31/mountains-6067150_1280.jpg
---
## 背景

> 在日常开发中，项目发布与部署，版本的迭代部署，隐形中占据了很长时间，特别是一些前端项目，样式的更改很频繁，这就需要我们频繁的发布版本，有没有一种很好的解决方式呢？Jenkins 就是这样一个持续平台，主要包含 `普通项目构建`,`流水线构建`, `多分支流水构建`。

## 准备工作

由于在项目部署中服务器多是 `CentOs`，在这里我也采用 `CentOS` 搭建

## 前提条件

* 检查是否安装了 `Docker`,如果本机没有请移步 [Docker安装](https://docs.docker.com/get-docker/)

## 安装 Jenkins

### 下载镜像

```bash
docker pull jenkins
```

### 启动 Jenkins

```bash
docker run -d --name jenkins -p 9090:8080 -v /home/jenkins_home:/var/jenkins_home jenkins
```

> 设置宿主机的 `9090` 映射端口，并且挂载数据卷到宿主机 `/home/jenkins_home` 目录下
查看运行中的容器 `docker ps`

```bash
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                                                                                         NAMES
727f1fb35d37        jenkins                  "/bin/tini -- /usr/l…"   39 hours ago        Up 39 hours         50000/tcp, 0.0.0.0:9090->8080/tcp jenkins
```

> 在这个过程中可能会遇到容器运行失败，通过 `docker logs jenkins` 查看日志

```bash
cannot touch '/var/jenkins_home/copy_reference_file.log': Permission deniedCan not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions
```

> 出现上面这种报错信息是因为volume 权限问题,需要修改下目录权限, 因为当映射本地数据卷时，/home/docker/jenkins目录的拥有者为root用户，而容器中jenkins user的uid为1000 执行如下命令即可
`方式一`

```bash
chown -R 1000:1000 /home/docker/jenkins
```

`方式二`

```bash
docker run -d --name jenkins -p 9090:8080 -v /home/jenkins_home:/var/jenkins_home -u 0 jenkins
```

### 初始化 Jenkins

容器启动成功后输入 `http://服务器地址:9090`,如果无法访问，请检查一下防火墙端口是否开放，如果是云服务器还需要检查安全组设置，第一次访问图片如下所示:

![jenkins初始化图片](/images/linux/jenkins-init.png)

> 首次启动jenkins需要输入密码，需要进入容器内获取密码。密码位于`/var/jenkins_home/secrets/initialAdminPassword`。

`方式一`

```bash
docker exec -it jenkins /bin/bash
```

获取密码

```bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

`方式二`

> 由于我们将`/var/jenkins_home` -- 挂载到--> `/home/jenkins_home`所以也可以直接`cat /home/jenkins_home/secrets/initialAdminPassword` 获取密码。

输入密码过后会进入下面界面: 安装插件

![jenkins安装插件](/images/linux/jenkins-install.png)

等待安装会出现一下界面: 不用关错误信息，直接跳过。

![jenkins安装插件结果](/images/linux/jenkins-plugins.png)

设置jenkins 的默认登录账号和密码，就可以进入仪表盘了。

### 插件安装失败

> 进入jenkins的主页面右上角可能会出现一些报错信息，主要是提示jenkins 需要的某些插件没有安装，或者说jenkins版本太低了，插件无法使用这个时候我们需要先升级jenkins做一个升级。

`方式一: 自动升级`
![jenkins自动升级](/images/linux/jenkins-upgrade.png)

`方式二: 手动升级`
Jenkins的官网下载好最新jar包上传到服务器，也可以使用wget命令。

```bash
wget http://updates.jenkins-ci.org/download/war/2.239/jenkins.war
```

> Jenkins的更新主要是替换jenkins镜像里面的war包 ，我们可以把下载好的war包使用docker cp直接进行复制命令如下

```bash
docker cp jenkins.war jenkins:/usr/share/jenkins
docker restart jenkins
```

### 更换插件源

![jenkins自动升级](/images/linux/jenkins-upgrade-1.png)

![jenkins自动升级](/images/linux/jenkins-upgrade-2.png)

Update Site 中更改插件源, 保存提交

```bash
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

```
### 安装必要插件
* Localization: Chinese (Simplified) 1.0.14 汉化包 搜索关键字 chinese
* Publish Over SSH 1.20.1 搜索关键字 ssh
* DingTalk 钉钉通知 2.3.0

![jenkins安装插件](/images/linux/jenkins-plugins-1.png)
> 到这里基本上就可以正常使用了 jenkins 发布项目了
