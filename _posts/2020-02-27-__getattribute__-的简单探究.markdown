---
layout: post
title:  "__getattribute__的简单探究"
date:   2020-02-26 13:03:36 
---
```__getattribute__```是基类 object 的一个类方法，所有继承 object 的类，都存在这个方法。
当类的实例的 attribute（属性和方法）被访问时，该方法会被自动调用。

我们可以通过重写 ```__getattribute__``` 来验证这个定律。

```python
class Thing(object):

    def __init__(self,name=None):
        self.name = name

    def __getattribute__(self,key):
        print('the __getattribute__ is running')
```
```python
t = Thing('apple') 

print(t.name) # 访问 name 属性
# out: the __getattribute__ is running 
```
那么 ```__getattribute__```  有什么作用? 它是如何帮助查询 attribute 的？
根据[文档](https://docs.python.org/3/howto/descriptor.html#id6)，```__getattribute__``` 的 Python 实现如下：

```python
def __getattribute__(self, key): # self 即类的实例，key 即查询的 attribute 名。
# 所以如果只查询类的 attribute，是不会调用该方法的。
    "Emulate type_getattro() in Objects/typeobject.c"
    v = object.__getattribute__(self, key) # 必须父类查询，不然无限递归，程序崩溃。
    
    if hasattr(v, '__get__'): # 如果 v 也是一个类并拥有 __get__ 方法，则调用该方法。
    # 这里是帮助描述器工作的机制。
        return v.__get__(None, self) #
        
    return v # 如果不是，则返回查询的 attribute 值。
```