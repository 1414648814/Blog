# 1 主机子节序和网络子节序

什么是字节序，指的是多字节的数据在内存中的存放顺序，根据整数在连续的4byte内存中的存储顺序，字节序被分为大端序和小端序；

* 大端字节序：一个整数的高位字节（23～41）存储在内存的低地址处，而低位字节（0～7）则存储在内存的高地址处；
* 小端字节序：一个整数的高位字节存储在内存的高地址处，低位字节存储在内存的低位子节；
*

很多机器默认使用的是小端。


IP地址转化函数

in\_addr\_t inet_addr(const char* strptr);

将点分十进制字符串表示的IPv4地址转化成用网络字节序整数表示的IPv4地址。它失败时返回INADDR_NONE。

int inet\_aton(const char\* cp, struct in_addr* inp);

函数完成和inet_addr同样的功能，但是将转化结果存储于参数inp指向的地址结构中。它成功时返回1，失败则返回0。

char* inet\_ntoa(struct in_addr in);

将网络字节序整数表示的IPv4地址转化为点分十进制字符串表示的IPv4地址。但是需要注意的是，该函数内部用一个静态变量存储转化结果，函数的返回值指向该静态内存，因此inet_ntoa是不可重入的。

---

下面的函数同样可以。

int inet\_pton(int af, const char\* src，char* dst，sockslen_t cnt);

将字符串表示的IP地址src（IPv4和IPv6都可以）转换成网络字节序整数表示的IP地址，并把转换结果存储于dst指向的内存中。

const char\* inet\_ntop(int af, const void\* src, char* dst, socklen_t cnt);

和上面相反的转换。成功时返回目标存储单元的地址。


# 2 Socket

## 2.1 创建Socket

int socket(int domain, int type, int protocol);

创建一个socket

## 2.2 命名Socket

int bind(int socktfd, const sruct sockaddr* my_addr, socklen_t addrlen);

将my_addr所指向的socket地址分配给未命名的sockfd文件描述符。

## 2.3 监听Socket

int listen(int sockfd, int backlog);

创建一个监听队列以存放处理的客户连接。其中backlog参数提示内核队列的最大长度。如果超出的话，服务器将不会受理新的客户连接。

## 2.4 接受连接

int accept(int sockfd, struct sockaddr *addr, socklen_t *addlen);

accept成功时返回一个新的连接socket，该socket唯一的标识了被接受的这个连接，服务器可通过读写该socket来和被接受连接对应的客户端通信。

## 2.5 发起连接

int connect(int sockfd, const struct sockaddr* serv\_addr, socklen_t addlen);

客户端通过connect来主动和服务器建立连接。


## 2.6 关闭连接

int close(int fd);

两个端都可以使用。但是这并非总是立即关闭一个连接，而是将fd的引用计数减1，只有当fd的引用计数为0的时候，才会真正关闭连接。

多线程程序中，一次fork系统调用默认将使父进程中打开的socket的引用计数加一，因此我们必须在父进程和子进程中都对该socket执行close调用才能将连接关闭。

如果无论如何都要立即终止连接，可以使用

int shutdown(int sockfd, int howto);

shutdown能够分别关闭socket上的读或者写，或者都关闭。但是close在关闭连接时只能将socket上的读和写同时关闭。

# 3 数据读写

## 3.1 TCP数据读写

ssize\_t recv(int sockfd, void* buf, size_t len, int flags);

tcp接受数据

ssize\_t send(int sockfd, const void* buf, size_t len, int flags);

tcp写入数据

## 3.2 UDP数据读写

ssize\_t recvfrom(int sockfd, void\* buf, size\_t len, int flags, struct sockaddr* src\_addr, socklen_t addrlen);

ssize\_t sendto(int sockfd, const void\* buf, size\_t len, int flags, const struct sockaddr* dest\_addr, socklen_t addrlen);

同上，但是UDP通信因为没有连接的概念，所以我们每次读取数据都需要获取发送端的socket地址。写入数据同样如此需要指定接收端的socket地址。

这两个函数同时还可以用于面向连接（STREAM）的socket的数据读写，只需要将最后两个参数设置成NULL。

## 3.3 通用数据读写

既可以用于TCP流数据，也能用于UDP数据报：

ssize_t recvmsg(int sockfd, struct msghdr* msg, int flags);

ssize_t sendmsg(int sockfd, struct msghdr* msg, int flags);

# 4 带外标记

Linux内核通知应用程序带外数据到达的两种常见方式：I/O复用产生的异常事件和SIGURG信号。

int sockatmark(int sockfd);

判断是否处于带外标记，即下一个被读取到的数据是否是带外数据。

# 5 地址信息函数

int getsockname(int sockfd, struct sockaddr\* address, socklen\_t* address_len);

int getpeername(int sockfd, struct sockaddr\* address, socklen\_t* address_len);

获取本端和远端socket地址，并且将其存储于address中。

# 6 socket选项

int getsockopt(int sockfd, int level, int option\_name, void\* option\_value, socklen\_t* restrict option_len);

int setsockopt(int sockfd, int level, int option\_name, void\* option\_value, socklen\_t* restrict option_len);

设置socket属性。

# 7 其他

## 7.1 SO_REUSEADDR选项

强制使用被处于TIME_WAIT状态的连接占用的socket地址。

## 7.2 SO_RCVBUF和SO_SNDBUF选项

设置TCP接受数据和发送数据大小。

## 7.3 其他

TCP客户端

1. socket() 
2. connect()
3. write()
4. read()
5. close()

TCP服务端

1. socket()
2. bind()
3. listen()
4. accept()
5. read()
6. write()
7. close()


在[这篇文章](http://blog.csdn.net/tennysonsky/article/details/45621341)中，**作者质疑了unp中TCP连接队列满了以后，不再接收新的客户端的connect，而是在满了以后，会延时连接，并不会不接收**。


## 7.4 I/O多路复用

I/O多路复用是一种并发服务器开发技术（处理多个客户端的连接）。通过该技术，系统内核缓冲I/O数据，当某个I/O准备好后，系统就会通知应用程序该I/O可读或者可写，这样应用程序就可以马上完成相应的I/O操作，而不需要等待系统完成相应的I/O操作，从而应用程序不必因为I/O操作而阻塞。

### 7.4.1 Linux

select

poll

epoll

### 7.4.2 Windows

事件选择模型

重叠I/O模型（异步I/O模型）

完成端口模型（IOCP）

### 7.4.3 FreeBSD（iOS/mac）

kqueue


---


[复用模型](http://www.cnblogs.com/liyux/p/5603826.html)

https://www.zhihu.com/question/19732473

http://www.jianshu.com/p/f671d3895d13


## 7.5 阻塞和非阻塞，同步和异步

### 7.5.1 例子

故事：老王烧开水。

出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。

老王想了想，有好几种等待方式

1.老王用水壶煮水，并且*站在那里*，*不管水开没开，每隔一定时间看看水开了没*。－**同步阻塞**

老王想了想，这种方法不够聪明。

2.老王还是用水壶煮水，不再傻傻的站在那里看水开，*跑去寝室上网*，*但是还是会每隔一段时间过来看看水开了没有，水没有开就走人*。－**同步非阻塞**

老王想了想，现在的方法聪明了些，但是还是不够好。

3.老王这次使用高大上的响水壶来煮水，*站在那里*，*但是不会再每隔一段时间去看水开，而是等水开了，水壶会自动的通知他*。－**异步阻塞**

老王想了想，不会呀，既然水壶可以通知我，那我为什么还要傻傻的站在那里等呢，嗯，得换个方法。

4.老王还是使用响水壶煮水，*跑到客厅上网去*，等着响水壶*自己把水煮熟了以后通知他*。－**异步非阻塞**

老王豁然，这下感觉轻松了很多。

---

* **同步和异步**

    同步就是烧开水，需要自己去轮询（每隔一段时间去看看水开了没），异步就是水开了，然后水壶会通知你水已经开了，你可以回来处理这些开水了。
同步和异步是相对于操作结果来说，会不会等待结果返回。

* **阻塞和非阻塞**

    阻塞就是说在煮水的过程中，你不可以去干其他的事情，非阻塞就是在同样的情况下，可以同时去干其他的事情。阻塞和非阻塞是相对于线程是否被阻塞。

其实，这两者存在本质的区别，它们的修饰对象是不同的。阻塞和非阻塞是指进程访问的数据如果尚未就绪，进程是否需要等待，简单说这相当于函数内部的实现区别，也就是未就绪时是直接返回还是等待就绪。
而同步和异步是指访问数据的机制,同步一般指主动请求并等待I/O操作完毕的方式,当数据就绪后在读写的时候必须阻塞,异步则指主动请求数据后便可以继续处理其它任务,随后等待I/O,操作完毕的通知,这可以使进程在数据读写时也不阻塞。

### 7.5.2 详细介绍

网络IO的模型大致包括下面几种

* 同步模型（synchronous IO）
    * 阻塞IO（bloking IO）
    * 非阻塞IO（non-blocking IO）
    * 多路复用IO（multiplexing IO）
    * 信号驱动式IO（signal-driven IO）
* 异步IO（asynchronous IO）
    * 异步IO

网络IO的本质是socket的读取，socket在linux系统被抽象为流，IO可以理解为对流的操作。对于一次IO访问，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间，所以一般会经历两个阶段：
1. 等待所有数据都准备好或者一直在等待数据，有数据的时候将数据拷贝到系统内核；
2. 将内核缓存中数据拷贝到用户进程中；

对于socket流而言：
1. 等待网络上的数据分组到达，然后被复制到内核的某个缓冲区；
2. 把数据从内核缓冲区复制到应用进程缓冲区中；

#### 7.5.2.1 阻塞IO

##### 介绍

这也是最常用的模型，默认情况下所有的套接字都是 `阻塞` 的；

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/26414081-file_1491994457423_13e6d.png)

我们把recvfrom函数视为系统调用，因为我们正区分进程和内核，系统调用一般都会从在应用进程空间中运行切换到内核空间中运行，一段时间后又再切换回来；

我们可以从图中看到，应用进程从 `进行系统调用` 到 `复制数据报到应用进程的缓冲区完成` 的整段时间内是被阻塞的；在这个过程中，要么正确到达，要么系统调用被信号打断；直到数据报被复制到用户进程完成后，用户进程才解除阻塞的状态，当然，这是用户进程自己进行的阻塞；

##### 优点和缺点

* 优点：能够及时返回数据，无延迟；方便调试；
* 缺点：需要付出等待的代价；

#### 7.5.2.2 非阻塞IO

##### 介绍

非阻塞，当所请求的I/O操作非得把当前进程设置成睡眠才能完成时，不要把当前进程设置成睡眠，而是返回一个错误信息（数据报没有准备好的情况下），此时当前进程可以做其它的事情，不用阻塞；

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/27967427-file_1491997152954_10575.png)

从图中可以得知，前三次系统调用时都没有数据可以返回，内核均返回一个  `EWOULDBLOCK`，并且不会阻塞当前进程，直到第四次询问内核缓冲区是否有数据的时候，此时内核缓冲区中已经有一个准备好的数据，因此将内核数据复制到用户空间，此时系统调用则返回成功；

> 当一个应用进程像这样对一个非阻塞socket循环调用 `recv/recvfrom` 时，则称为轮询；应用进程持续轮询内核，以查看某个操作是否就绪，这么做往往消耗大量的CPU时间。

##### 优点和缺点

* 优点：相较于阻塞模型，非阻塞不用再等待任务，而是把时间花费到其它任务上，也就是这个当前线程同时处理多个任务；

* 缺点：导致任务完成的响应延迟增大了，因为每隔一段时间才去执行询问的动作，但是任务可能在两个询问动作的时间间隔内完成，这会导致整体数据吞吐量的降低。

#### 7.5.2.3 IO多路复用

有了I/O复用，我们就可以调用 `select或poll`，让其阻塞在两个系统调用（1.询问数据是否准备好并且直到数据准备好才返回；2.内核是否把数据全部复制完成到用户进程）中的某一个之上

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/2193057-file_1491998838696_160a3.png)

图中阻塞于 `select` 调用，等待数据报套接字变为可读。当select返回套接字可读这一条件的时候，则调用 `recvfrom` 把所读数据报复制到应用进程缓冲区；

之前的同步非阻塞方式需要用户进程不停的轮询，但是IO多路复用不需要不停的轮询，而是派别人去帮忙循环查询多个任务的完成状态，UNIX/Linux 下的 `select、poll、epoll` 就是干这个的；select调用是内核级别的，select轮询相对非阻塞的轮询的区别在于---前者可以等待多个socket，能实现同时对多个IO端口进行监听，当其中任何一个socket的数据准好了，就能返回进行可读，然后进程再进行recvform系统调用，将数据由内核拷贝到用户进程，当然这个过程是阻塞的。select或poll调用之后，会阻塞进程，与blocking IO阻塞不同在于，此时的select不是等到socket数据全部到达再处理, 而是有了一部分数据（网络上的数据是分组到达的）就会调用用户进程来处理。如何知道有一部分数据到达了呢？监视的事情交给了内核，内核负责数据到达的处理。

我认为上面那句话中存在两个重要点：1.对多个socket进行监听，只要任何一个socket数据准备好就返回可读；2.不等一个socket数据全部到达再处理，而是一部分socket的数据到达了就通知用户进程；

其实 `select、poll、epoll` 的原理就是不断的遍历所负责的所有的socket完成状态，当某个socket有数据到达了，就返回可读并通知用户进程来处理；

##### 优点和缺点

* 优点：能够同时处理多个连接，系统开销小，系统不需要创建新的额外进程或者线程，也不需要维护这些进程和线程的运行，降低了系统的维护工作量，节省了系统资源。
* 缺点：如果处理的连结数目不高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。（因为阻塞可以保证没有延迟，但是多路复用是处理先存在的数据，所以数据的顺序则不管，导致处理一个完整的任务的时间上有延迟）

##### 同步非阻塞和多线程＋同步阻塞

高并发的程序一般使用同步非阻塞方式而非多线程 + 同步阻塞方式。要理解这一点，首先要扯到并发和并行的区别。比如去某部门办事需要依次去几个窗口，办事大厅里的人数就是并发数，而窗口个数就是并行度。也就是说并发数是指同时进行的任务数（如同时服务的 HTTP 请求），而并行数是可以同时工作的物理资源数量（如 CPU 核数）。通过合理调度任务的不同阶段，并发数可以远远大于并行度，这就是区区几个 CPU 可以支持上万个用户并发请求的奥秘。在这种高并发的情况下，为每个任务（用户请求）创建一个进程或线程的开销非常大。而同步非阻塞方式可以把多个 IO 请求丢到后台去，这就可以在一个进程里服务大量的并发 IO 请求。

#### 7.5.2.4 信号驱动式I/O模型

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/66043283-file_1492002950669_22bd.png)

首先开启套接字的信号驱动式IO功能，并且通过 `sigaction` 系统调用安装一个信号处理函数，该函数调用将立即返回，当前进程没有被阻塞，继续工作；当数据报准备好的时候，内核则为该进程产生 `SIGIO` 的信号，随后既可以在信号处理函数中调用 `recvfrom` 读取数据报，并且通知主循环数据已经准备好等待处理，也可以通知主循环让它读取数据报；（其实就是一个待读取的通知和待处理的通知）；


#### 7.5.2.5 异步式I/O模型

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/75136497-file_1492004375006_bcbe.png)

我们调用 `aio_read` 函数，给内核传递描述符、缓冲区指针、缓冲区大小和文件偏移，并且告诉内核当整个操作完成时如何通知我们。该函数调用后立即返回，不被阻塞；

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/185128-file_1491989742145_181eb.png)


#### 7.5.2.6 比较

![](http://odwv9d2u8.bkt.clouddn.com/17-4-12/48821302-file_1492005018721_d9f5.png)



> 推荐文章
[IO多路复用机制详解](http://xuding.blog.51cto.com/4890434/1739649)
[I/O模型与多路复用](http://rainybowe.com/blog/2016/09/12/IO%E6%A8%A1%E5%9E%8B%E4%B8%8E%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8/index.html)
[聊聊Linux 五种IO模型](http://www.jianshu.com/p/486b0965c296)
[聊聊IO多路复用之select、poll、epoll详解](http://www.jianshu.com/p/dfd940e7fca2)


### 7.5.3 实现


