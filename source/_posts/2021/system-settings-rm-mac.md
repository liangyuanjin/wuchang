---
title: 修改 rm 命令，删除文件进废纸篓
date: 2021-03-25 09:05:05
tags: 系统
categories: Mac
description: 系统设置
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2021/02/21/18/48/elks-6037526_1280.jpg
---
## 背景

> 在日常使用 Mac 的过程中，在终端中经常会使用 `rm` 命令，使用这种方式删除的资源不会出现在废纸篓中，若是误删的话，想要找回比较困难

## 解决方式

> 可以使用 `trash` 脚本替换 `rm`命令，它其实是调用的 `finder` 的api 进行删除，这样终端中删除的资源也会出现在废纸篓中。

## 安装trash

使用 `homebrew` 就可以安装 `trash`，非常方便

```bash
brew install trash
```

安装成功过后可以使用 `trash -fr filename`, 命令和 `rm` 一样，对于我们已经习惯了 `rm` 命令, 我们可以用 `trash` 替换 `rm` 命令。

### 修改配置文件

本人使用的是 `zsh` 模式的终端，在 `~/.zshrc` 文件中配置，如果是 `bash` 模式的终端，在 `~/.bash_profile` 中配置

```bash
echo alias rm="trash" >> ~/.zshrc
source ~/.zshrc
```
