---
title: 在flask框架内构造一个消息系统
date: 2018-08-06 10:40:17
tags: Python Flask 数据库 消息通知系统
---
在前不久的一段工作历程中，我曾经遇到一个问题。我当时要做一个消息通知系统，但是因为之前没有过类似的经验，而且对编程之道也是一片迷茫，所以完全不知该从何下手，于是在网上找了搜索各种资源，希望能够一劳永逸地能找到一个类似的项目，但是找了很久都没有找到符合自己需要的。倒是在这个过程中看到有网友推荐使用flask-socketIO来构造消息系统，也有人推介用socket+redis+gunicorn来构造。尽管当时这些东西对我来说还是很陌生而且高端，但为了解决自己的难题就硬着头皮看了好几天的这些内容。结果却让我很失望，看了这些知识以后，我无法把它们和自己的项目联系起来，还是和之前一样没有头绪。最后只能又厚着脸皮去问技术负责人，结果是我想复杂了，这个消息系统并不是即时通信，没必要用到socket编程，只需要每次请求接口时能够显示消息并统计未读消息就行了。这样一来，顿时觉得心里的压力小了很多，经过过去几天的思考和资料收集，对于这样一个相对简单的消息系统还是有些头绪的，很快我就有了思路，这篇文章即是对这一过程的总结。

#### 1.程序设计思路

这个消息通知系统要求能够实现的功能如下：

* 运营人员发送系统公告；
* 所有用户都能收到系统公告，并列出消息列表；
* 当用户有未读消息时，页面要进行提示(例如标记小红点);
* 当用户读取消息详情时，红点要消失；
* 用户可以删除消息；

<!-- more -->
程序的设计思路为设计两个数据库表，一个用来表示系统运营消息（OperationMessage），另一个则用来表示用户接收到的消息(Message)，两个数据库表都设置一个`mid`的字段，且`mid`需要打上时间戳。当运营发送消息时，即是将消息存储进OperationMessage类，而当用户接受消息（即请求消息列表这个接口）时，考虑两个情况：

1. 用户在请求接口之前的`Message`库表为空，那么需要将`OperationMessage`库表的所有内容都复制到`Message`；
2. 用户在请求接口之前的`Message`库表不为空，那么需要将`OperationMessage`里比`Message`新的消息复制过来即可；

所有消息都设置一个`is_read`字段，用来表征是否已读,设置默认值为`False`；一个`is_able`字段，用来表征是否可用（不可用即表示已经删除），设置默认值为`True`。从消息列表读取某一条消息的详情时，将该消息的`is_read`字段改为`True`，通过统计`is_read`为`False`字段的数量即可知道未读消息的数目，如果为0，即表示没有未读消息。而如果要删除消息，则只要将消息的`is_able`字段改为`False`即可。

#### 2.数据库设计

依照前述的程序设计思路,数据库(mongoengine)的设置如下:

``` python
import time
from mongoengine import Document, StringField, FloatField, ListField, BooleanField, \ 
        DateTimeField, ReferenceField

class OperateMessage(Document):
    mid = FloatField(required=True, verbose_name="信息ID")
    title = StringField(max_length=40, required=True, verbose_name='标题')
    content = StringField(max_length=256, required=True, verbose_name='内容')
                                    verbose_name='接收人')
    is_read = BooleanField(default=False, verbose_name='是否已读')
    is_system = BooleanField(default=True, verbose_name='是否为系统消息')

    created_at = DateTimeField(default=datetime.now, verbose_name='创建时间')
    updated_at = DateTimeField(default=datetime.now, verbose_name='更新时间')

    meta = dict(
        indexes=['mid'],
        ordering=['-mid']
    )

    def __init__(self, **kwargs):
        super(OperateMessage, self).__init__(**kwargs)
        if not self.mid:
            self.mid = time()

    def __repr__(self):
        return '<OperateMessage {mid!r}>'.format(mid=self.mid)

class Message(Document):
    SYSTEM_MESSAGE = 'system'
    PERSON_MESSAGE = 'person'
    COMMENT_MESSAGE = 'comment'
    TYPE_CHOICES = (
        (SYSTEM_MESSAGE, '系统消息'),
        (PERSON_MESSAGE, '个人消息'),
        (COMMENT_MESSAGE, '回复')
    )

    mid = FloatField(required=True, verbose_name="信息ID")
    title = StringField(max_length=40, verbose_name='标题')
    content = StringField(max_length=256, required=True, verbose_name='内容')
    type = StringField(choices=TYPE_CHOICES, verbose_name='类型')
    reply = ListField(ReferenceField('Message'), verbose_name='回复')
    from_user = ReferenceField('User', verbose_name='发送人')
    to_user = ReferenceField('User', verbose_name='接收人')
    is_read = BooleanField(default=False, verbose_name='是否已读')
    is_system = BooleanField(default=False, verbose_name='是否为系统消息')
    is_enable = BooleanField(default=True, verbose_name="是否禁用")

    created_at = db.DateTimeField(default=datetime.now, verbose_name='创建时间')
    updated_at = db.DateTimeField(default=datetime.now, verbose_name='更新时间')

    meta = dict(
        indexes=['mid'],
        ordering=['-mid']
    )

    def __init__(self, **kwargs):
        super(Message, self).__init__(**kwargs)
        if not self.mid:
            self.mid = time()

    def __repr__(self):
        return '<Message {mid!r}>'.format(mid=self.mid)
```
其中,`from_user`和`to_user`对应了User表格的用户,这里为了简略就不写出来了.`mid`通过`time()`来打上时间戳,之所以使用`time()`是因为我们是根据`mid`来比较消息的新旧,并且对时间的精度要求也较高.但是`mid`会是一个比较长的浮点型小数,不太好看,原来我曾想通过乘法将其变为整数,但是因为数位太长,mongoengine处理不了(mongoengine处理的最大整数是32位的),或者将其转化为字符串,但是字符串在使用mongoengine的条件查询时不能用于比较大小,所以作罢.

#### 3.程序设计

项目程序采用前后端分离的设计,代码如下:

``` python
from datetime import datetime
from flask import request, jsonify, Blueprint
from flask_login import login_required, current_user
from .models import User, OperateMessage, Message

@msg.route('/messages/operation', methods=['POST'])
@login_required
def publish_message():
    """ 运营发送系统消息 """
    if not request.json:
        return jsonify(msg='请求中缺失参数！')

    # 要求前端传来的参数
    content = request.json.get('content', None)
    title = request.json.get('title', None)

    if not content:
        return jsonify(msg="发送内容不能为空!")
    if not title:
        return jsonify(msg="发送标题不能为空!")
    
    message = OperateMessage(content=content, title=title)
    message.save()
    return jsonify(
        title = message.title,
        content = message.content,
        mid = message.mid
            )


@msg.route('/messages/box', methods=['POST'])
@login_required
def recieve_message():
    """ 消息盒子接受消息 """
    # 找出当前用户最近收到的信息
    newest_system = Message.objects(is_system=True, 
            to_user=current_user.id).order_by("-created_at").first()
    # mark是一个时间标尺
    if not newest_system:
        mark = 0
    else:
        mark = newest_system.mid

    # 在运营消息里只要是时间比mark要新的就是都要发送的,将其存入Message数据库
    to_be_sent = OperateMessage.objects(mid__gt=mark).all()
    if to_be_sent:
        for msg in to_be_sent:
            message = Message(
                title = msg['title'],
                content = msg['content'],
                is_system = True,
                mid = msg['mid'],
                created_at = msg['created_at'],
                to_user = current_user.id,
                from_user = "2018071800000003"
            )
            message.save()
    
    # 从数据库里找到当前用户的所有没被禁用的消息
    messages = Message.objects(to_user=current_user.id, is_enable=True)).order_by(
                        '-created_at').all()
    res = {}
    for message in messages.items:
        info = dict(
            title = message['title'] or '',
            content = message['content'] or '',
            created_at =  message['created_at'].strftime("%Y-%m-%d %H:%M%S"),
            is_read = message['is_read'],
            from_user = message['from_user']
        )
        if message.is_system:
            info['from_user'] = 'system'
        res[str(message['mid'])] = info

    # 统计出未读信息的数目,将其传给前端,前端可根据其来做一些标示
    noread_count = len(Message.objects(is_read=False, to_user=current_user.id,
                            is_enable=True).all())
    
    return jsonify(msg='消息接受成功', data=res, count=noread_count)


@msg.route('/messages/detail', methods=['POST'])
@login_required
def read_message():
    """ 读取消息详情 """
    if not request.json:
        return jsonify(msg='请求中缺失参数！')
    mid = request.json.get('mid', None)

    if not mid:
        return jsonify(msg="消息ID不能为空!")
    to_be_read = Message.objects(mid=mid, to_user=current_user.id).first()
    if not to_be_read:
        return jsonify(msg="消息ID有误!")

    info = dict(
        title = to_be_read['title'] or '',
        content = to_be_read['content'] or '',
        created_at =  message['created_at'].strftime("%Y-%m-%d %H:%M%S"),
        from_user = to_be_read['from_user'] or ''
    )
    if to_be_read.is_system:
            info['from_user'] = 'system'
    # 读取完消息详情就将is_read设为True
    to_be_read.is_read = True
    to_be_read.save()
    return jsonify(msg="消息读取成功", data=info)


@msg.route('/messages/delete', methods=['POST'])
@login_required
def delete_message():
    """ 删除消息 """
    if not request.json:
        return jsonify(msg='请求中缺失参数！')

    mid = request.json.get('mid', None)

    if not mid:
        return jsonify(msg="消息ID不能为空!")
    to_be_delete = Message.objects(mid=mid, to_user=current_user.id).first()
    if not to_be_delete:
        return jsonify(msg="消息ID有误!")
    if not to_be_delete.is_enable:
        return jsonify(msg="该消息已经删除!")
    to_be_delete.is_enable = False
    to_be_delete.save()
    return jsonify(msg="删除消息成功!")
```

#### 4.心得总结

这种方法简单易行,主要是通过数据库表格的设置来达成目的,利用中间表同步的思想.但是有一个很大的不足之处在于,如果在用户量很大的情况下,运营每发送一条消息,每个用户就都会有一条这样的消息存储在库表了,一旦系统消息频繁发送,那么数据很快就会变得庞大不堪.这也是此种方法的最大的不足之处.

