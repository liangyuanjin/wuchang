---
title: Golang 程序结构
date: 2021-04-15 08:41:38
tags: Go
categories: Go
description: Golang 环境搭建
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/golang/golang-framework-choose.jpeg
---

> Go 和其他语言一样，程序是由很多小的基础构件组成的，变量存值，简单的加法和减法运算被组合成复杂的表达式，还有基础类型被聚合为数组或结构体，后面这些都会一一学习到，先从最简单的 `命令`, `声明`, `变量`, `赋值`，`类型`, `包和文件`, `作用域`这些基础的概念学习

## 命名

> 在 Go 中`函数名`, `变量名`, `常量名`, `类型名`, `语句标识`和`包名`, 遵循的规则为 `一个名字必须以一个字母或下划线开头，后面可以跟任意数量的字母，数字或下划线`, `严格区分大小写`。

### 关键字

> Go 的关键字并不是很多，有`25个`, 关键字`不能用于自定义的名字`,在特定的语法结构中使用

```GO
break      default       func      interface     select
case       defer         go        map           struct
chan       else          goto      package       switch
const      fallthrough   if        range         type
continue   for           import    return        var
```

### 预定义名字

> `预定义名字并不是关键字`,可以在定义中重新使用他们, 但是要要注意`避免过度使用而引起的语义混乱`, 在Go 中这样的预定义名字大约有 30 多个， 主要对应 `内建的常量`, `类型` 和 `函数`

```go
// 内建常量:
            true  false  iota  nil
// 内建类型:
            int      int8      int16       int32      int64
            uint     uint8     uint16      uint32     uint64
            float32  float64   complex128  complex64  uintptr
            bool     byte      rune        string     error
// 内建函数:
            make    len     cap   new   append   copy   close   delete
            complex real    imag
            panic   recover
```

### 命名推荐

> 名字出现的位置, 影响到这个名字的有效使用范围，和作用域有关系，作用域这个后面会有降到讲到,名字在`函数内部定义, 就只在函数内部有效`,在 `函数外部定义, 当前包的所有文件中都可以访问`, `名字的开头字母的大小决定了包外的可见性`, `大写字母开头,是可以导出的, 可以被外部访问`, `包名`采用`小写字母`, Go语言名字的风格推荐`短小的名字`, `驼峰式命名`, `多个单词单用大小写分割`