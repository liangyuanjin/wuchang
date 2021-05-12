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

### 处理无效属性名

`关键字冲突`

> `FrozenJSON` 类有一个缺陷, 没有对名称为 Python 关键字的属性做特殊处理, 比如下面 `grad.class` 是无法读取的, 因为 class 是保留字, 可以用 `geattr(grad, "class")`, 但是 `FrozenJSON` 是为了方便访问数据, 更好的解决方法是 `ForzenJSON.__init__` 方法的映射中做处理，如果有的话在键名后面加上 `_` 然后通过 `grad.class_` 读取

```python
grad = FrozenJSON({"name": "Jim Bo", "class": 1982})
print(grad.class)
```

```python
class FrozenJSON:
    """
    一个只读接口，使用属性表示法表示访问JSON类对象
    """

    def __init__(self, mapping):
        self.__data = dict(mapping)
        for key, value in mapping.items():
            if keyword.iskeyword(key):
                key += "_"
                self.__data[key] = value

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

`无效标识符`

> 如果JSON对象的键不是`有效的Python标识符`，也会遇到无效属性名, 这种问题Python3中易于检测, str 类提供了 `s.isidentifier()`, 一般处理无效标识符是抛出异常，或者替换成同一的通用名称

```python
class FrozenJSON:
    """
    一个只读接口，使用属性表示法表示访问JSON类对象
    """

    def __init__(self, mapping):
        self.__data = dict(mapping)
        for key, value in mapping.items():
            if keyword.iskeyword(key):
                key += "_"
                self.__data[key] = value
            if not key.isidentifier():
                raise ValueError()

x = FrozenJSON({'2be':'or not'})
print(x.2be)
```

> `FrozenJSON` 类的另一个重要功能, `build` 逻辑, 这个方法把嵌套结构转换成 `FrozenJSON` 实例或者 `FrozenJSON` 实例列表, `__getattr__` 方法访问属性时， 能为不同的值返回不同类型的对象

### 使用 __new__ 方法以灵活的方式创建对象

> Python 中 `__init__` 称为`构造方法`, 是从其他语言借鉴过来的, 但是用于`构建实例的是特殊方法__new__`, 是一个`特殊的类方法`(不必使用@classmethod装饰器), 必须返回一个实例, 返回的实例作为第一个参数(self) 传给 `__init__`方法, `__init__`方法要传入实例, 禁止返回任何值,`__init__`方法其实是 `初始化方法`, 真正的构造方法是 `__new__`。

```python
class FrozenJSON:
    """
    一个只读接口, 使用属性表示访问JSON类对象
    """
    def __new__(cls, arg):
        if isinstance(arg, Dict):
            return super().__new__(cls)
        elif isinstance(arg, List):
            return [cls(item) for item in arg]
        else:
            return arg

    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if iskeyword(key):
                key += "_"
            self.__data[key] = value

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        return FrozenJSON(self.__data[name])
```

* `__new__` 是类方法, 第一个参数是类本身, 余下的参数与 `__init__` 方法一样, 只不过没有 `self`
* `return super().__new__(cls)` 默认行文是委托给超类的 `__new__`方法, 这里调用的是object 基类的 `__new__`方法, 把唯一参数设为 `FrozenJSON`
* 之前调用的是 `FrozenJSON.build`方法，现在只需要调用 `FrozenJSON` 构造方法

> `__new__`方法的第一个参数是类, 创建的对象是那个类的实例, 在 `FrozenJSON.__new__`方法中, `super().__new__(cls)` 表达式会调用 `object.__new__(FrozenJSON)`, 而 object 类构建的实例是`FrozenJSON`实例, 实例的 `__class__`属性存储的是 `FrozenJSON`类的引用,真正的构建操作是由解释器C语言实现的的 `object.__new__` 方法执行的

## 使用shelve模块调整JSON数据源的结构

> `JSON`数据源有一个明显的缺点, 如果索引为 40 的事件, 有两位演讲者, 3471和5199, 但是却不容易找到他们，因为提供的是编号，而 Schedule.speakers 列表没有使用编号建立索引, 每对象中都有 venue_serial 字段，存储的值也是编号，如果想要找到相应的记录，就要执行线性搜索列表。
