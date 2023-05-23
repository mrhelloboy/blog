---
title: 简谈Python特性01
date: 2023-02-20T20:14:22+08:00
tags:
  - HTML
  - eager
  - lazy
  - loading
categories:
  - HTML
draft: true
---

1. 什么是特性（property)
2. 特性的作用是什么

## 什么是特性
首先特性是对**对象的属性**而言的。所以说特性是对作用于属性。

在《流畅的Python》书中这样描述特性，

> 在不改变类接口的前提下，使用存取方法（即读值方法和设值方法）修改数据属性  

> 特性经常用于把公开的属性变成使用读值方法和设值方法管理的属性，且在不影响客户端代码的前提下实施业务规划  

在Python中我们很少看到典型的读值方法和设值方法，我们一般使用的都是`obj.attr`  或者 ` obj.attr = value` 这样的点号来读取和设值属性。而在Java中，如果读取一个对象的属性（读值方法）一般是：
```java
obj.getX()
```

设置对象的属性（设值方法）是：
```java
obj.setX(value)
```

## 特性的典型作用 —— 验证属性
一般情况下,我们我们自定义一个类型时，都会在初始化方法 `__init__()` 中定义好属性。
```python
class Rectangular:
    """
    定义一个矩形
    """
    def __init__(self, hight, width):
        self.hight = hight
        self.width = width

    def area(self):
        """
        求矩形的面积
        """
        return self.hight * self.width

```

创建一个矩形对象`r`, 但在现实中，我们知道矩形的长宽是必须大于0的，这在这个矩形对象中，我们并没有对对属性进行限制。在__init__中我们一般不会加入限制逻辑。
```python
def __init__(self, hight, width):
    self.hight = hight
    self.width = width
    if self.hight <= 0:
        raise ValueError('self.hight must be greater than 0')
    if self.width <= 0:
        raise ValueError('self.widht must be greater than 0')
```
这样挺怪异。

而特性的一个典型用法就是用来**限制属性**的。

这样我们就可以在__init__中简单定义下对象的属性，而对属性的限制就交给特性就可以了
```python
class Rectangular:
    """
    定义一个矩形
    """
    def __init__(self, h, w):
        self.hight = h  #<1>
        self.width = w

    @property  #<2>
    def hight(self): #<3>
        return self._hight

    @hight.setter #<4>
    def hight(self, value): #<5>
        if value < 0:
            raise ValueError("hight must be greater than 0")
        else:
            self._hight = value

    @property
    def width(self):
        return self._width

    @width.setter
    def width(self, value):
        if value < 0:
            raise ValueError("widht must be greater than 0")
        else:
            self._width = value

    def area(self):
        """
        求矩形的面积
        """
        return self.hight * self.width
```

在<2>和<4>中，我们使用了 `@property` 和 `hight.setter` 装饰器。
@propert装饰器实现了只读特性，@hight.setter实现了可写特性。

在__init__的<1>的`self.hight = h`赋值操作，就是一个设值操作，会调用可写特性，即调用<5>的	`hight()`方法。

对`@hight.setter`装饰器中的hight，其实是只读特性装饰的`hight`方法。这样写可能比较清晰：
```python
@property
def get_hight(self):
    pass

@get_hight.setter
def hight(self, value):
    pass
```

被装饰的读值方法`get_hight`有个`.setter`的属性，而这个属性也是一个装饰器，这个装饰器的作用就是将读值方法和设值方法绑定在一起。

在求面积的方法方法中，self.hight这个读取对象属性操作中，就会调用<3>这个读值方法了。

回到最初的特性定义中。我们想要这个矩形类型中的公开的属性是hight和width，但我们又要对这两个属性进行限制。虽然我们可以像Java那样使用存取方法来完成这个限制，但不可避免破坏了获取属性(obj.attr)和设值属性(obj.attr = v)的接口。而使用特性后，我们既没有破坏我们常用的接口，又实用了存取方法做到了对属性的限制。

