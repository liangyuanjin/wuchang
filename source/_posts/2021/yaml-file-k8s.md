---
title: YAML 基本语法
date: 2021-03-19 16:34:23
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
> 在使用 kubernetes 集群时候会用到 `YAML` 文件来创建相关的资源，本篇博客对`YAML`基本语法进行一些总结

## 基本语法

YAML 基本语法规则如下:

* 大小写敏感
* 使用缩进表示层级关系
* 缩进时不允许使用 Tab 键， 只允许使用空格
* `#` 表示注释

> 在多数情况下，我们只需要`Lists`, `Maps` 两种结构类型就行了

## Maps

`Maps` 是字典，就是 `key:value` 的键值对，Maps 可以让我们更加方便的去写配置信息。

```yaml
---
apiVersion: v1
kind: Pod
```

> 第一行的 `---` 是分隔符， 是可选的，在同一个文件中，可以用`---` 区分多个文件，上面的 `YAML` 文件中有两个键，`kind` 和 `apiVersion`, 他们对应的值分别是 `v1` 和 `Pod`, 对应的 JSON的格式如下

```json
{
    "apiVersion": "v1",
    "kind": "pod"
}
```

下面我们来看一个稍微复杂一点的 YAML 文件， 创建一个 KEY 对应的值是一个 Maps

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
```

上面这个文件中，用了两个空格作为缩进，空格的数量并不重要，但是的保持一致，并且至少要求一个空格， `name` 和 `labels` 是相同级别的缩进，所以 YAML 就知道他们属于同一个 MAP, 而 `app` 是 `labels` 的值，是因为 `app` 的缩进更大, 在 YAML 文件中绝对不要使用 tab 键, 上面的 YAML 文件转换成 JSON 文件如下:

```yaml
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "kube100-site",
    "labels": {
      "app": "web"
    }
  }
}
```

## Lists

Lists 就是和列表或者数组类似，在YAML 文件中我们这样定义

```yaml
args
  - Cat
  - Dog
  - Fish
```

> 可以有任何数量的项在列表中，每个项定义以破折号(-) 开头，与父元素直接可以缩进一个空格，对应的JSON格式如下:

```yaml
{
    "args": ["Cat", "Dog", "Fish"]
}
```

Lists 的子项也可以是Maps, Maps 的子项也可以是Lists

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: flaskapp-demo
      image: jcdemo/flaskapp
      ports:
        - containerPort: 5000
```

> 定义了 `containers` 的 `List` 对象, 每个子项都有 name, image, ports 组成， 每个ports都有一个 key 为 containerPort 的 Map 组成，上面的 YAML 文件对象的 JSON 格式如下:

```json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "name": "kube100-site",
        "labels": {
            "app": "web"
        }
    },
    "spec": {
        "containers": [{
            "name": "front-end",
            "image": "nginx",
            "ports": [{
                "containerPort": 80
            }]
        }, {
            "name": "flaskapp-demo",
            "image": "jcdemo/flaskapp",
            "ports": [{
                "containerPort": 5000
            }]
        }]
    }
}
```
