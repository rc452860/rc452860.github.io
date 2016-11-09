---
title: JavaScript 类型判断
date: 2016-11-03 10:58:51
tags:
---

### 什么是鸭子类型
现在没空查先记一下

```
Value               function   typeof
-------------------------------------
"foo"               String     string
new String("foo")   String     object
1.2                 Number     number
new Number(1.2)     Number     object
true                Boolean    boolean
new Boolean(true)   Boolean    object
new Date()          Date       object
new Error()         Error      object
[1,2,3]             Array      object
new Array(1, 2, 3)  Array      object
new Function("")    Function   function
/abc/g              RegExp     object
new RegExp("meow")  RegExp     object
{}                  Object     object
new Object()        Object     object 
```

`Object.prototype.toString()`有一个妙用，如果我们以某个特别的对象为上下文来调用该函数，它会返回正确的类型。我们需要做的就是手动处理其返回的字符串，最终便能获得typeof应该返回的正确字符串。可以用来区分：Boolean, Number, String, Function, Array, Date, RegExp, Object, Error等等。