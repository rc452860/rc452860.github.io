---
title: python __slots__ 内存占用问题
date: 2016-10-09 10:59:10
tags:
---

>[通过Python的__slots__节省9GB内存](https://segmentfault.com/a/1190000002492474)

今天看python的时候看到这片文章觉得很有意思于是自己试验了一下。但是并没有传说中的效果不知道我姿势不对还是什么问题下次再来试试。

本文使用 memory_profiler测试python内存占用

安装memory_profiler
``` shell
pip install -U memory_profiler
pip install psutil
```

使用命令
```
python -m memory_profiler main.py
```

测试源码:
``` python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
this is test for learn python
"""

import os, sys

from memory_profiler import profile


class Date(object):
    __slots__ = ['year', 'month', 'day']

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day


class Date2(object):

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day


@profile
def test():
    for i in range(0,10*10000):
        Date(2016,1,1)


@profile
def test2():
    for i in range(0,10*10000):
        Date2(2016,1,1)


if __name__ == '__main__':
    test()
    test2()
```

最后拿到的测试结果是:
```
skydeiMac-6:untitled2 sky$ python -m memory_profiler main.py 
Filename: main.py



Line #    Mem usage    Increment   Line Contents
================================================
    28     10.1 MiB      0.0 MiB   @profile
    29                             def main():
    30     13.2 MiB      3.1 MiB       for i in range(0,10*10000):
    31     13.2 MiB      0.0 MiB           Date(2016,1,1)


Filename: main.py


Line #    Mem usage    Increment   Line Contents
================================================
    33     13.2 MiB      0.0 MiB   @profile
    34                             def main2():
    35     13.2 MiB      0.0 MiB       for i in range(0,10*10000):
    36     13.2 MiB      0.0 MiB           Date2(2016,1,1)
```


2016年10月09日11:10:43 更新：

上面测试代码有问题，python采用引用计数的方式来实现内存回收。如果一个实例没有被引用的话会在一定时间内被回收。所以上面的只有一个变量被保存在内存里所以看不出来什么，我改进了一下测试代码。

``` python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
this is test for learn python
"""

import os, sys

from memory_profiler import profile


class Date(object):
    __slots__ = ['year', 'month', 'day']

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day


class Date2(object):

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day


@profile
def test():
    for i in range(0,10*10000):
        locals()["sakura"+str(i)] = Date(2016,1,1)


@profile
def test2():
    for i in range(0,10*10000):
        locals()["sakura2"+str(i)] = Date2(2016,1,1)


if __name__ == '__main__':
    test()
    test2()
```
这一次的测试结果和预期一样。__slots__确实能节省内存。

这是本次的测试代码。
```
Filename: main.py

Line #    Mem usage    Increment   Line Contents
================================================
    30     10.0 MiB      0.0 MiB   @profile
    31                             def test():
    32     34.5 MiB     24.5 MiB       for i in range(0,10*10000):
    33     34.5 MiB      0.0 MiB           locals()["sakura"+str(i)] = Date(2016,1,1)


Filename: main.py

Line #    Mem usage    Increment   Line Contents
================================================
    36     23.0 MiB      0.0 MiB   @profile
    37                             def test2():
    38     62.4 MiB     39.4 MiB       for i in range(0,10*10000):
    39     62.4 MiB      0.0 MiB           locals()["sakura2"+str(i)] = Date2(2016,1,1)
```
还有一个问题没有搞明白。
dir() 与 __dict__，__slots__ 的关系
下面是我做的一些测试，不过我后来找到一篇博客写的非常好
[[Python] dir() 与 __dict__，__slots__ 的区别](http://www.cnblogs.com/ifantastic/p/3768415.html)
```
class Date(object):

    __slots__ = ('year', 'month', 'day')

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day


class Date2(object):

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day




if __name__ == '__main__':
    print dir(Date(2016,1,1))
    print dir(Date2(2016,1,1))
    print dir(Date)
    print dir(Date2)
    print dir(object)
```
```
['__class__', '__delattr__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__slots__', '__str__', '__subclasshook__', 'day', 'month', 'year']
['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'day', 'month', 'year']
['__class__', '__delattr__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__slots__', '__str__', '__subclasshook__', 'day', 'month', 'year']
['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
['__class__', '__delattr__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']

```
下结论之前一定要多测试几次。