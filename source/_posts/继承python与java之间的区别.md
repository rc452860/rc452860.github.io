---
title: 关于继承java和python的一些区别
date: 2016-10-18 15:00:10
tags: python
---
# 关于继承java和python的一些区别

为什么要写这篇博客。因为最近在弄树莓派鉴于的c++只是二级水平而python又比较容易以前也接触了一些所以选择用python来做这些琐事
但是学的语言太多了难免有些记混，觉得不对的地方所以拿出来单练一下

**首先谈谈语法：**

python中继承使用 `class 类名(父类)` 来实现继承
java中使用`extnds`关键字来实现继承

**java继承的实现与输出:**
```
class Root {
    public Root(){
        System.out.println("enter root");
    }
}
class Root2 extends Root{
    public Root2(){
        System.out.println("enter root2");
    }

    public void say(){
        System.out.println("father");
    }
}

public class Main extends Root2 {

    public Main(){
        System.out.println("main");
    }

    @Override
    public void say() {
        super.say();
        System.out.println("樱の泪");
    }

    public static void main(String[] args) {
        Main main = new Main();
        main.say();
    }
}


-------------------console-------------------
enter root
enter root2
main
樱の泪

```

**python版本的：**

```

class Root(object):
    def __init__(self):
        print("this is Root")


class B(Root):
    def __init__(self):
        print("enter B")
        # print(self)  # this will print <__main__.D object at 0x...>
        super(B, self).__init__()
        print("leave B")


class C(Root):
    def __init__(self):
        print("enter C")
        super(C, self).__init__()
        print("leave C")


class D(B, C):
    def __init__(self):
        print("en  ter D")
        super(D,self).__init__()
        print("enter D")



d = D()
print(d.__class__.__mro__)


-------------------console-------------------
enter D
enter B
enter C
this is Root
leave C
leave B
enter D
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.Root'>, <type 'object'>)

```

这里需要注意下这两个语言的区别：
关于Super这个关键字的
刚才试验中我发现java的构造函数中带不带`super()`调用都会默认调用父类的构造函数
但是其他方法必须手写`super()`才会调用父类的方法

关于 `super` 学java的同志们肯定都知道在java中这个关键字其实就是指向父类
但是在python中 这个关键字并不是指向父类 而是指向MRO中的下一个类

MRO 全称 Method Resolution Order，它代表了类继承的顺序

```
def super(cls, inst):
    mro = inst.__class__.mro()
    return mro[mro.index(cls) + 1]
```

上面python继承的例子中就已经可以看到最后一行输出就是python的继承顺序首先是自身然后是B，C最后才是root

于是，MRO 中类的顺序到底是怎么排的呢？[Python’s super() considered super!][1]中已经有很好的解释，我翻译一下：
在 MRO 中，基类永远出现在派生类后面，如果有多个基类，基类的相对顺序保持不变。


[1]:https://rhettinger.wordpress.com/2011/05/26/super-considered-super/