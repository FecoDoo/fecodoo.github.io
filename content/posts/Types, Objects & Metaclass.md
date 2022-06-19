---
title: "Type, Object & Metaclass in Python"
date: 2019-08-15T11:22:14+08:00
draft: false
---

## type和object的关系

一句话简述：`types`是`objects`的一个子类，`objects`是`type`的一个实例。在Python的世界中，`object`是**类父子关系**的顶端，所有的数据类型的**父类**都是它；`type`是**类型实例关系**的顶端，所有**对象**都是它的**实例**。它们两个的关系可以这样描述：

![rel](https://pic1.zhimg.com/80/ca54cfa2cc510d2dcc40e3cc7fb2e051_1440w.jpg?source=1940ef5c)

- 白板上的第一列，目前只有`type`，我们先把这列的东西叫`Type`。
- 白板上的第二列，它们既是第三列的**类型**，又是第一列的**实例**，我们把这列的对象叫`TypeObject`。
- 白板上的第三列，它们是第二列类型的实例，而没有**父类**（\_\_bases\_\_）的，我们把它们叫`Instance`。


[详见知乎：jeff kit](https://www.zhihu.com/question/38791962/answer/78172929)

## metaclass

`metaclass`常被用来在**类实例化**前做一些**动态更改类属性**的事情，比如，依赖于自省，控制继承等等。它的作用简而言之为：
- 中断类的默认创建
- 修改类属性，方法等
- 返回修改后的类

`metaclass`继承自`type`，是类的类。
以下是一个最简单的`metaclass`例子

```python
class MyMetaclass(type):
    def __new__(cls, name: str, bases: set, attrs: dict) -> type:
        # some custom process
        attrs_processed = {}
        for name, val in attrs.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val
        
        return super().__new__(cls, name, bases, attrs_processed)
```

`metaclass`的一个主要用途就是构建API。Django(一个python写的web框架)的ORM就是一个例子。

用Django先定义了以下Model:
```python
class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()
```
然后执行下面代码:
```python
    guy = Person.objects.get(name='bob')
    print guy.age  # result is 35
```
这里打印的输出并不是IntegerField，而是一个int，int是从数据库中获取的。
这是因为`models.Model`使用`metaclass`来实现了复杂的数据库查询。但对于你看来，这就是简单的API而已，不用关心背后的复杂工作。

[本节引用](https://segmentfault.com/a/1190000007255412#:~:text=%E8%87%AA%E5%AE%9A%E4%B9%89metaclass,%E4%BD%BF%E7%94%A8%E5%AE%83%E6%9D%A5%E5%88%9B%E9%80%A0%E7%B1%BB%E3%80%82)