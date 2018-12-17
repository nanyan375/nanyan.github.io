---
title: Python实现ping工具(二)
date: 2018-12-15 22:32:50
tags: python ping ICMP struct select
---

第一部分阐述了ping工具的原理和涉及到的协议方面的知识，接下来我们需要使用python语言去实现它们，主要涉及到的模块有struct,select,socket。socket我想大家应该都比较熟悉，所以重点说一下前两个模块。

### struct模块

在网络通信当中，大多传递的数据是以二进制流（binary data）存在的。当传递字符串时，不必担心太多的问题，而当传递诸如int、char之类的基本数据的时候，就需要有一种机制将某些特定的结构体类型打包成二进制流的字符串然后再网络传输，而接收端也应该可以通过某种机制进行解包还原出原始的结构体数据。python中的struct模块就提供了这样的机制，该模块的主要作用就是对python基本类型值与用python字符串格式表示的C struct类型间的转化（This module performs conversions between Python values and C structs represented as Python strings.）。在我们这里主要用到的函数是pack和unpack。下面我们举例来实践它：<!--more-->

```python
>>> import struct
>>> t = (1, b'abc', 2.1)
>>> s = struct.Struct('I3sf')
>>> s.size
12
>>> p = s.pack(*t)
>>> p
b'\x01\x00\x00\x00abc\x00ff\x06@'
>>> s.unpack(p)
(1, b'abc', 2.0999999046325684)
```

代码中，首先定义了一个元组数据，包含int、string、float三种数据类型，然后定义了struct对象，并制定了format‘I3sf’，I 表示int，3s表示三个字符长度的字符串，f 表示 float。最后通过struct的pack和unpack进行打包和解包。通过输出结果可以发现，value被pack之后，转化为了一段二进制字节串，而unpack可以把该字节串再转换回一个元组，但是值得注意的是对于float的精度发生了改变，这是由一些比如操作系统等客观因素所决定的。打包之后的数据所占用的字节数与C语言中的struct十分相似。但需要注意的是，字符串的长度是以４个长度为单位的，只能是４的倍数，这也是上述代码中ｓ的size有12的原因。定义format可以参照官方api提供的对照表：
![FagN6S.png](https://s1.ax1x.com/2018/12/15/FagN6S.png)
另一方面，打包的后的字节顺序默认上是由操作系统的决定的，当然struct模块也提供了自定义字节顺序的功能，可以指定大端存储、小端存储等特定的字节顺序，对于底层通信的字节顺序是十分重要的，不同的字节顺序和存储方式也会导致字节大小的不同。在format字符串前面加上特定的符号即可以表示不同的字节顺序存储方式，例如采用小端存储 s = struct.Struct(‘<I3sf’)就可以了。官方api library 也提供了相应的对照列表：
![FagclT.png](https://s1.ax1x.com/2018/12/15/FagclT.png)

### select模块

This module provides access to the select() and poll() functions available in most operating systems, devpoll() available on Solaris and derivatives, epoll() available on Linux 2.5+ and kqueue() available on most BSD. Note that on Windows, it only works for sockets; on other operating systems, it also works for other file types (in particular, on Unix, it works on pipes). It cannot be used on regular files to determine whether a file has grown since it was last read.
在我们这里只涉及select.select()函数。理解select.select模块其实主要就是要理解它的参数, 以及其三个返回值。select()方法接收并监控3个通信列表， 第一个是所有的输入的data,就是指外部发过来的数据，第2个是监控和接收所有要发出去的data(outgoing data),第3个监控错误信息。

### 最终实现

综合以上所有的论述，通过对ping原理的解释和将要用到的模块的详细阐述，我们就可以实现ping的所有代码了。代码如下:

```python
# -*- coding: utf-8 -*-
import sys
import os
import time
import socket
import struct
import select

def checksum(data):
    """
    定义校验和函数
    """
    n = len(data)
    m = n % 2
    sum = 0  # 将校验和初始化为0
    # 以16bit为单位，依次进行求和
    for i in range(0, n-m, 2):
        sum += data[i] + (data[i+1] << 8)

    if m:
        sum += data[-1]

    # 对校验和本身进行16bit求和
    sum = (sum >> 16) + (sum & 0xffff)
    sum += sum >> 16  # 如果还有高于16位的继续求和
    # 对结果求反
    answer = ~sum & 0xffff
    # 主机字节序转网络字节序
    answer = ((answer << 8) & 0xff00) | (answer >> 8)
    return answer

def ping_socket(dst_addr,imcp_packet):
    """
    建立套接字。
    """
    # 建立ICMP的套接字
    ping_socket = socket.socket(socket.AF_INET,socket.SOCK_RAW, socket.getprotobyname("icmp"))
    send_request_ping_time = time.time()
    ping_socket.sendto(imcp_packet, (dst_addr, 80))
    return send_request_ping_time, ping_socket, dst_time

def request_ping(data_type,data_code,data_checksum,
    data_ID,data_Sequence,payload_body):
    """
    将发送数据打包成二进制。其中:
    data_type: 数据类型，0或8
    data_code: 类型代码, 0
    data_checksum: 校验和
    data_ID: 报文的标识符, 当前进程id
    data_Sequence: 报文的序号
    payload_body: 正文
    """
    imcp_packet = struct.pack(
        '>BBHHH32s',
        data_type,
        data_code,
        data_checksum,
        data_ID,
        data_Sequence,
        payload_body
    )
    # 获取校验和
    icmp_chesksum = chesksum(imcp_packet)
    # 重新再打包
    imcp_packet = struct.pack(
        '>BBHHH32s',
        data_type,
        data_code,
        icmp_chesksum,
        data_ID,
        data_Sequence,
        payload_body
    )
    return imcp_packet

def response_ping(send_request_ping_time, pingsocket,
    data_Sequence,timeout = 2):
    """
    回应
    """
    while True:
        started_time = time.time()
        what_ready = select.select([pingsocket], [], [], timeout)
        wait_for_time = time.time() - started_time
        if what_ready[0] == []:  # 连接超时
            return -1
        recieved_time = time.time()
        recieved_packet, addr = pingsocket.recvfrom(1024)
        # IMCP报文是通过IP协议发送的，所以前面有ip首部，长度为20个字节
        # imcp首部长度为8个字节
        imcpHeader = recieved_packet[20:28]
        type, code, checksum, packet_id, sequence = struct.unpack(
            '>BBHHH', imcpHeader
        )
        # 如果报文类型，序号正确，则返回所用时间
        if type == 0 and sequence == data_Sequence:
            return recieved_time - send_request_ping_time
        time_out = timeout - wait_for_time
        if time_out <= 0:
            return -1

def ping(host):
    data_type = 0
    data_code = 0
    data_checksum = 0
    data_ID = os.getpid()
    data_Sequence = 1
    payload_body = b'abcdefghigkmlnopqrstuvwxyz'
    # 根据域名获取ip
    dst_addr = socket.gethostbyname(host)
    print("正在ping {0} [{1}]具有{2}字节的数据".format(host, dst_addr, len(payload_body)))
    for i in range(0, 4):
        imcp_packet = request_ping(data_type, data_code, data_checksum, 
                    data_ID, data_Sequence+i, payload_body)
        send_request_ping_time, ping_socket, addr = ping_socket(dst_addr, imcp_packet)
        times = response_ping(send_request_ping_time, ping_socket, data_Sequence+i)
        if times > 0:
            print("来自{0}的回复: 字节={1}, 时间={2}ms".format(addr, len(payload_body), times*1000))
            time.sleep(0.7)
        else:
            print("请求超时")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        sys.exit("Usage: python3 ping.py <host>")
    ping(sys.argv[1])
```