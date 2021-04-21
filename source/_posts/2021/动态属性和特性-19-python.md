---
title: Python 动态属性和特性
date: 2021-04-21 08:18:36
tags:
  - Python
categories: Python
description: Python  语法
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2020/09/07/17/40/birds-5552482_1280.jpg
---

> Python中, `数据的属性`和`处理数据的方法`统称为属性, `方法只是可调用的属性`, 除此之外我们可以`创建特性(property)`, 不改变类接口的前提下, 使用存取方法修改数据属性。出了特性, Python 还提供了很多的API, 例如控制属性的访问权限，实现动态属性,`使用点号访问属性时`, Python 解释器会调用特殊方法 `__getattr__` 和 `__setattr__`, `动态创建属性是一种元编程`, 框架的作者经常这样使用

## 使用动态属性转换数据

`osconfeed.json` 文件内容如下

```json
{
  "Schedule": {
    "conferences": [
      {
        "serial": 115
      }
    ],
    "events": [
      {
        "serial": 34505,
        "name": "Why Schools Don´t Use Open Source to Teach Programming",
        "event_type": "40-minute conference session",
        "time_start": "2014-07-23 11:30:00",
        "time_stop": "2014-07-23 12:10:00",
        "venue_serial": 1462,
        "description": "Aside from the fact that high school programming...",
        "website_url": "http://oscon.com/oscon2014/public/schedule/detail/34505",
        "speakers": [
          157509
        ],
        "categories": [
          "Education"
        ]
      }
    ],
    "speakers": [
      {
        "serial": 157509,
        "name": "Robert Lefkowitz",
        "photo": null,
        "url": "http://sharewave.com/",
        "position": "CTO",
        "affiliation": "Sharewave",
        "twitter": "sharewaveteam",
        "bio": "Robert ´r0ml´ Lefkowitz is the CTO at Sharewave, a startup..."
      }
    ],
    "venues": [
      {
        "serial": 1462,
        "name": "F151",
        "category": "Conference Venues"
      }
    ]
  }
}
```

`osconfeed.py`

```python
from urllib.request import urlopen
import warnings
import os
import json


URL = 'http://www.oreilly.com/pub/sc/osconfeed'
JSON = "osconfeed.json"


def load():
    if not os.path.exists(JSON):
        msg = "downloading {} to {}".format(URL, JSON)
        warnings.warn(msg)
        with urlopen(URL) as remote, open(JSON, 'wb') as local:
            local.write(remote.read())

    with open(JSON) as fp:
        return json.load(fp)


if __name__ == "__main__":
    feed = load()
    for key, value in sorted(feed["Schedule"].items()):
         print('key:-> {} value:-> {}'.format(key, value))

    print(feed["Schedule"]["speakers"][-1]["name"])
    print(feed["Schedule"]["speakers"][-1]["serial"])
    print(feed["Schedule"]["events"][-1]["name"])
    print(feed["Schedule"]["events"][-1]["speakers"])
```

* `with` 语句中使用了两个上下文管理器, 分别用于读取和保存文件
* `json.load` 函数解析 JSON 文件, 返回 Python 原生对象
* `feed` 的值是一个字典，里面潜逃这字典和列表, 可以通过循环打印 `key` 和 `value`


### 使用动态属性访问JSON类数据

> `feed["Schedule"]["speakers"][-1]["name"]` 这种句法很冗长, 在 Python 中 可以实现一个字典的类, 打到 `feed.Schedule.events[-1].name` 这种取值方式,下面`FronzenJSON` 类支持读取， 支持递归，自动处理嵌套的映射和列表

```python
class FrozenJSON:
    """
    一个只读接口，使用属性表示法表示访问JSON类对象
    """

    def __init__(self, mapping):
        self.__data = dict(mapping)

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        return FrozenJSON.build(self.__data[name])

    @classmethod
    def build(cls, obj):
        if isinstance(obj, Dict):
            return cls(obj)
        elif isinstance(obj, List):
            return [cls.build(item) for item in obj]
        return obj
```

* 使用 `mapping` 参数构建一个字典, 确保传入是字典, 创建一个字段的副本
* 当没有指定名称(name)的属性时， 才会调用 `__getattr__方法`
* 如果name是实例 `__data`的属性, 返回那个属性，调用`keys`方法就是通过这种方式
* 否则, `self.__data` 中获取name键对应的元素,返回调用 `FrozenJSON.build()` 方法得到结构

> 这一行中的 `self.__data[name]` 表达式可能抛出 KeyError 异常。我们应该处理这个异常，抛出AttributeError 异常，因为这才是 `__getattr__` 方法应该抛出的异常种类

* 如果 obj 是映射，那就构建一个 FrozenJSON 对象。
* 如果是列表类型， 因此，我们把 obj中的每个元素递归地传给 .build() 方法，构建一个列表
* 如果既不是字典也不是列表，那么原封不动地返回元素。

> `FrozenJSON` 类的关键是 `__getattr__方法`, 当无法使用常规的方式获取属性(在实例，类或超类找不到指定的属性) 解释器才会调用 `__getattr__方法`, `FrozenJSON` 只有两个方法 `__init__` 和 `__getattr__` 和一个实例属性 `__data`, 尝试获取其他属性会触发解释器调用 `__getattr__方法`, 这个方法首先会查看 `self.__data` 字典有没有指定的名称的属性(不是键), `FrozenJSON` 便可以处理字典的所有方法, 把 `items` 方法委托给 `self.__data.items()`, 如果`self.__data` 没有指定名称的属性， 那么 `__getattr__` 会以那个名称为键, 从`self.__data` 中获取一个元素，传给`FronzenJSON.build方法`, 这样便可以做到嵌套结构

`osconfeed.py`

```python
from urllib.request import urlopen
import warnings
import os
import json
from typing import List, Dict

URL = 'http://www.oreilly.com/pub/sc/osconfeed'
JSON = "osconfeed.json"


def load():
    if not os.path.exists(JSON):
        msg = "downloading {} to {}".format(URL, JSON)
        warnings.warn(msg)
        with urlopen(URL) as remote, open(JSON, 'wb') as local:
            local.write(remote.read())

    with open(JSON) as fp:
        return json.load(fp)


class FrozenJSON:
    """
    一个只读接口，使用属性表示法表示访问JSON类对象
    """

    def __init__(self, mapping):
        self.__data = dict(mapping)

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        return FrozenJSON.build(self.__data[name])

    @classmethod
    def build(cls, obj):
        if isinstance(obj, Dict):
            return cls(obj)
        elif isinstance(obj, List):
            return [cls.build(item) for item in obj]
        return obj


if __name__ == "__main__":
    feed = load()
    for key, value in sorted(feed["Schedule"].items()):
        print('key:-> {} value:-> {}'.format(key, value))

    print(feed["Schedule"]["speakers"][-1]["name"])
    print(feed["Schedule"]["speakers"][-1]["serial"])
    print(feed["Schedule"]["events"][-1]["name"])
    print(feed["Schedule"]["events"][-1]["speakers"])

    print()
    raw_feed = load()
    feed = FrozenJSON(raw_feed)
    print(feed.Schedule.speakers)
    print(feed.Schedule.keys())
    for key, value in sorted(feed.Schedule.items()):
        print('key:->{} value:-> {}'.format(key, value))

    talk = feed.Schedule.events[0]
    print(type(talk))
    print(talk.serial)
    print(talk.flaver)
```

* 传入嵌套的字段和列表组成的 `raw_feed`, 创建一个 `FrozenJSON`实例
* 可以使用底层字典的方法, `keys()` 获取所有的 key, 使用 `items()` 方法获取各个记录集合及其内容
* `feed.Schedule.speakers，仍是列表`, `如果里面的元素是映射，会转换成 FrozenJSON 对象`
* 读取不存在的属性会抛出 `KeyError`, 而不是通常抛出的 `AttributeError`

