---
title: 选择排序
date: 2021-03-16 08:56:31
tags:
    - 数据结构
    - Python
categories: Arithmetic
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2021/01/24/20/47/tabby-5946499_1280.jpg
---

## 内存的工作原理

> 假设你去逛超市， 需要将东西寄存，寄存处有柜子，柜子有很多抽屉，`计算机内存就像是很多抽屉的集合`，每个抽屉都有地址，需要将数据存储到内存时，向计算机申请存储空间，计算机给你一个存储空间，但是需要存储多项数据的时候怎么办呢？有两种基本方式 `数组` 和 `链表`。

## 数组

---

> 当我们需要在内存中存储一系列元素。例如编写一个管理代办事项的应用程序，数组和链表都可以实现，这里先介绍数组，我们可以将代办事项都存在数组中，使用数组意味着`所有代办事项在内存中都是相连的`

### 数组出现的问题

> 你之前只添加了三个代办事项，这是你想添加第四个代办事项，但是你发现`地址已经被别人占用`,在这种情况下，你需要请求计算机重新分配一块可以容纳4个代办事项的内存，再将所有代办事项都移到哪里。

### 解决之道

* 提前申请内存

> 即便当前只有3个代办事项，也像计算机申请10个位置，以防需要添加代事项,这样是要代办事项不超过10个,就无需转移，但是这也存在两个缺点， 一是 `额外的内存可能根本就用不上，造成内存浪费`, 二是 `当代办事项超过10个过后，还得转移`。这种权益措施虽然不错，但绝非完美的解决方案

## 链表

---

> 链表和数组的最大的`区别`就是，`链表的元素可以存储在内存的任何地方`， `链表的每个元素都存储了下一个元素的地址`，从而是一系列随机的内存地址串在一起。在链表中添加元素很容易，只需将其放入内存，并将其地址存储到前一个元素中。

### 链表出现的问题

> 当需要读取链表的最后一个元素时，你不能直接读取，因为你不知道它所处的内存地址。必须从元素1开始访问，从而获取元素2的地址，直到访问到最后一个元素，`需要同时读取所有元素时,链表效率很高;如果你需要跳跃，链表的效率很低`。`需要随机的读取元素时，数组的效率很高`。

### 数组和链表操作的运行时间

| 操作 | 数组 | 链表 |
| :---- | :---- | :---- |
| 读取 | O(1) | O(N) |
| 写入 | O(N) | O(1) |
| 删除 | O(N) | O(1) |

> 链表和数组谁用的比较多，数组用的很多，因为它支持水机访问，有两种访问方式: `随机访问`和`顺序访问`

## 选择排序

---
> 假设计算机存储很多歌曲，对于每首歌曲，都记录了其作品被播放的次数，现在你要将这个列表播放次数从多到少的顺序排列，`一种办法是遍历这个列表，找出作品播放次数最多的歌曲，并将歌曲添加到一个新的列表中`，再次这样做，找出播放次数第二多的歌曲添加到列表中，`O(n) 时间意味着查看列表中的每个元素一次`，对歌曲列表进行简单查找时，意味这每首歌曲都要查看一次，要找出播放次数最多的歌曲，必须检查列表中的每个元素，这需要的时间为 O(n) 因此对于这种时间为 O(n) 的操作，你需要执行 n 次， 需要的总时间为 `O(n * n )` 即 即`O(n2)`

> 选择排序是一种灵巧的算法，`但其速度不是很快`，`快速排序`是一种更快的排序算法，其运行时间为`O(n log n)`，

### Python 实现

```python
def find_smallest(arr):
    """
    寻找列表中最小的元素
    :params arr 列表
    """
    smallest = arr[0]
    smallest_index = 0
    for i in range(1, len(arr)):
        if arr[i] < smallest:
            smallest = arr[i]
            smallest_index = i
    return smallest_index


def select_sort(arr):
    """
    选择排序
    """
    new_arr = []
    for i in range(len(arr)):
        smallest = find_smallest(arr)
        new_arr.append(arr.pop(smallest))
    return new_arr
```