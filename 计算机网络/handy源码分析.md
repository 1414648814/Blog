# handy 源码分析

## poller.h/poller.cc

封装了linux中的 `epoll` 和mac中的`kqueue` 中的函数；

## net.h/net.cc

封装了网络的函数，比如端口，ip地址；

## conn.h/conn.cc

封装了tcp连接的函数

## conf.h/con.c

封装配置文件

## daemon.h/daemon.cc

封装守护进程

## event_base.h/event_base.cc

事件分发器，可以管理定时器、连接、超时连接

## file.h/file.cc

封装了文件的获取操作

## handy.h

核心库文件

## http.h/http.cc

封装了http的相关实现

## logging.h/logging.cc

封装了生成日志

## net.h/net.cc

封装了网络的相关，比如ipv4

## silce

？？？

## stat-sver.h/stat_ser.cc

服务器状态

## status.h

封装状态信息

## threads.h/threadss.cc

封装线程

## udp.h/udp.cc

封装udp

## util.h/util.c

？？？


---

先讲TCP的：

handy 封装了一个叫做`PollerBase`的类，当作所有系统的基类，然后根据系统去实现它，如果是`Linux`则为`PollerEpoll`，如果是`Mac os`则为`PollerKqueue`，这个自然是用来封装`epoll和kqueue`，封装了其的函数，用来管理通道。

实现了一个基类的`EventBase`事件分发器，用来管理定时器和连接、超时连接，用来管理。

实现了`Channel`，意为一个通道，封装了`PollerBase和EventBase`，实现了发送和接收函数。

`TcpConn`为最外层封装。



**一摸一样的啊，亲！**

moduo | handy | 描述
---|---|----
EventLoop | EventBase | 封装事件循环，也是事件分派的中心，关注一个IO事件
Channel | Channel | 负责注册和响应IO事件
Poller | PollerBase | 封装epoll和kevent的应用


