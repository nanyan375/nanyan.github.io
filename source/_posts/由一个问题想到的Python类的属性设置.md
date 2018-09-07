---
title: 由一个问题想到的Python类的属性设置
date: 2018-09-06 21:14:43
tags: python 类属性
---
## 1. 前言

Python的面向对象编程是Python语言中非常重要的一部分内容，对于Python新手来说也是比较难懂的。它的难不在于说逻辑有多复杂，内容有多繁琐，而是在于抽象的类对于没有经验的人来说很难想象它所代表的实际意义和在一个程序中所起到的作用。笔者也是如此，初学之时懵懵懂懂，但是在不断地实践过程中，对Python的理解也在逐步加深，这也是网上的大牛们往往会推荐大家无论如何要找一份实习的原因。

## 2. 一个问题

笔者在工作中遇到一个这样的问题：在一个mongoengine的类GoodsOrder里面有一个属性status，也叫字段，我需要对这个属性进行一些加工,需要在商品(Goods)里的库存stock变为0时status变为EXPIRED，请问我该如何去做？<!-- more -->(代码如下)

``` python
class Goods(db.Document):
    """ 商品 """
    name = db.StringField(max_length=255, required=True, verbose_name='名称')
    stock = db.IntField(default=0, verbose_name='库存')

class GoodsOrder(db.Document):
    """ 商品订单 """
    STATUS = db.choices(
        WAITING_PAY='等待兑换',
        WAITING_DELIVER='等待发货',
        WAITING_RECIEVE="等待收货",
        FINISHED='完成兑换',
        EXPIRED="已过期")

    goods = db.ReferenceField('Goods', required=True, verbose_name='名称')
    status = db.StringField(default=STATUS.WAITING_PAY, choices=STATUS.CHOICES,
            verbose_name='订单状态')
```

很显然，我们需要对status进行重写，于是笔者一开始将代码写为如下：

```Python
class GoodsOrder(db.Document):
    ......
    status = db.StringField(default=STATUS.WAITING_PAY, choices=STATUS.CHOICES,
            verbose_name='订单状态')

    @property
    def status(self):
        if self.goods.stack <= 0:
            self.status = STATUS.EXPIRED
        return self.status

```

但是很显然，这种写法是错误，从语法上就过不了关，在类里面就会有两个status的属性，一个的字段，一个是装饰器下的方法。那么我们应该怎样做呢？

## 3. 解决办法

对于这种情况，也许会有很多解决办法，笔者即将阐述的是比较常用的一种，其思路是将GoodsOrder的`status`字段改为私有属性`_status`然后再对status进行改写，代码如下：

```python
....

class GoodsOrder(db.Document):
    """ 商品订单 """
    STATUS = db.choices(
        WAITING_PAY='等待兑换',
        WAITING_DELIVER='等待发货',
        WAITING_RECIEVE="等待收货",
        FINISHED='完成兑换',
        EXPIRED="已过期")

    goods = db.ReferenceField('Goods', required=True, verbose_name='名称')
    _status = db.StringField(default=STATUS.WAITING_PAY, choices=STATUS.CHOICES,
            verbose_name='订单状态')

    @property
    def status(self):
        if self.goods.stack <= 0:
            self._status = STATUS.EXPIRED
        return self._status
```

这样一来，`status`将取代`_status`供给外界调用，而每次调用时又都会执行if判断语句，从而确保了status在库存为0时的状态是`EXPIRED`。

## 4. 推广与升华

上述的做法除了方便对属性进行加工以外其实还具有其他的作用.`_status`这种形式的我们称之为私有属性,用来表示只有在类内部才能使用,但这只是形式上的,事实上我们仍然可以从外部对其进行改写.为了实现真正的私有,在Python中也有一套经常会被用到的'组合拳'.我们以如下代码为例:

```python
from flask_login import UserMixin
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

db = SQLAlchemy()

class User:
    ....
    _password = db.Column('password', db.String(255), nullable=False)

    @property
    def password(self):
        return self._password

    @password.setter
    def password(self, orig_password):
        self._password = generate_password_hash(orig_password)

    def check_password(self, password):
        return check_password_hash(self._password, password)

    ....
```

(代码为flask里的一个用户model)

在所有的应用当中,密码这一属性是非常私密的,我们要防止外部有任何程序能够对其进行随意的修改,当然也需要对其进行加密,上述的代码正是说明了这一过程.为了更好地理解其中的妙处,我们还需要清楚几点深层次的东西:

* python中用`"."`操作来访问和改写类的属性成员时，会调用`__get__`和`__set__`方法，python会查找`class.__dict__`字典，对对应值进行操作。比如`C.x`会调用`C.__get__`访问最终读取`C.__dict__[x]`元素;

* 通过 `property`函数或者装饰器调用属性,实际是使用`fget`, `fset`, `fdel`函数对应变量操作地读取`(get)`，设置`(set)`和删除`(del)`函数。而`property`对象`<property object>`有三个类方法，即`setter`, `getter`和`delete`，用于之后设置相应的函数;

因此,上述代码的妙处在于:

1. 将密码`password`变为了真正的私有属性,防止可以直接从外部对其进行赋值而造成其的随便改动;
2. 对密码这一属性进行了`加密`这一加工过程,增加了密码的功能性;
3. 通过使用`装饰器`使代码更加的美观和简洁.
