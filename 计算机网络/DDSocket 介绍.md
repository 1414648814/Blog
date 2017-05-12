# Tickle

一个socket的网络库

需要包括以下内容：

* 原生api的使用示例（首先需要完成）
    * linux : epoll,select,poll
    * macos : kqueue
    * windows : iocp
* 封装库
* 封装库的使用示例
* 封装库的相关的简单测试
* 进行千万并发连接测试
* 使用openssl
* 使用protobuf


## 功能和优点：

* 使用智能指针，管理内存更加安全；
* 支持`ipv4和ipv6`；
* 支持TCP，UDP，Unix sockets (DGRAM and STREAM)；
* 跨平台：Linux, FreeBSD, Solaris，Windows；
* IPv4/IPv6 多播，节省消息复制造成的低效；
* 错误处理更加安全；
* socket使用更加方便快捷；
* 使用protobuf，openssl；
* 提供shell脚本编程；
* UDP转发；
* 动态端口转发；
* http连接；
* 易于扩展，并且提供高扩展性的代码规范检查工具；
* 多线程服务；
* 提供守护进程的使用；
* 提供谷歌测试
* 提供多种连接方案（reactor模式／proactor模式）


ZMQ是一个可伸缩的分布式或并发应用程序设计的高性能异步消息库。提供一个消息队列，但不需要专门的消息代理。

基本的ZMQ模式有：

* 请求相应模式
* 发布订阅模式
* 管道模式
* 排它对模式


# 遇见的问题

1. inet_ntop和inet_pton；
2. ip转字符串
3. [htons(), htonl(), ntohs(), ntohl()](http://beej.us/guide/bgnet/output/html/multipage/htonsman.html)
4. 全连接队列和半连接队列
5. connect和accept的底层实现（结合三次握手）
6. send和recv中的套接字是客户端还是服务器的
7. 半同步半异步I/O的设计模式(half sync/half async)
8. 各种乱七八糟的设计模式
    * 同步阻塞
    * 同步非阻塞
    * 异步非阻塞
    * 半同步半异步（同步模仿异步）
    * IO多路复用
    * 异步
    


epoll就是IO多路复用（select/poll）＋非阻塞

Reactor（epoll＋线程池）

    LT模式：不停的提醒用户进程
    ET模式：只提醒用户进程一次
        
Proactor（纯异步或者多进程+同步阻塞模拟异步或者Reactor＋线程池模拟异步）


## 原生API

### select

```cpp
int select(int numfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

```

函数参数

* numfds：文件描述符的最大值＋1（为了限制检测文件描述符的范围）
* readfds：包含所有因为状态变为可读而触发select函数返回文件描述符
* writefds：包含所有因为状态变为可写而触发select函数返回文件描述符
* exceptfds：包含所有因为状态发生特殊异常而触发select函数返回文件描述符
* timeout：表示阻塞超时时限

返回值

* 当为-1的时候表示出错
* 当为0的时候表示超时
* 当大于0则成功

```cpp
// 新增fd到set中
FD_SET(int fd, fd_set *set); 

// 从set中移除fd
FD_CLR(int fd, fd_set *set);

// 判断fd是否在set中
FD_ISSET(int fd, fd_set *set);

// 将set整个清0
FD_ZERO(fd_set *set);

```

**基本思路**，把要检测的文件描述符加载到 `fd_set` 类型的集合中，然后调用 `select` 函数检测加载到集合中的文件描述符；

`select` 函数监视的文件描述符分为3类，分别是 `writefds, readfds, exceptfds`，调用之后`select`函数就会阻塞，直到有文件描述符就绪（有数据可读，可写或者except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回；当select函数返回之后，可以通过遍历 `fdset`来找到就绪的描述符。

```cpp

#include <iostream>
#include <sys/select.h>
#include <unistd.h>
#include <netinet/in.h>
#include <unistd.h>
#include <assert.h>

const int MAXSIZE = 1024;

int main() {

    int sockfd = ::socket(AF_INET, SOCK_STREAM, IPPROTO_TCP); //sockfd为服务器的套接字
    sockaddr_in sin;
    sin.sin_family = AF_INET;
    sin.sin_port = htons(4567);  //1024 ~ 49151：普通用户注册的端口号
    sin.sin_addr.s_addr = INADDR_ANY;
    sockaddr_in client_addr;

    // ...bind 和 listen操作

    socklen_t clen = sizeof(sockaddr_in);

    struct timeval tv;
    int fds[MAXSIZE];
    memset(fds,-1,sizeof(fds));
    fd_set fdset;

    fds[0] = sockfd;

    while( 1 ) {
        FD_ZERO(&fdset);
        int i = 0;
        int fdmax = fds[0];
        for (; i < MAXSIZE; i++) {
            if (fds[i] != -1) {
                FD_SET(fds[i], &fdset);
                if (fdmax < fds[i]) {
                    fdmax = fds[i];
                }
            }
        }
        tv.tv_sec = 2;
        tv.tv_usec = 0;
        int res = select(fdmax + 1, &fdset, NULL, NULL, &tv);
        assert(res != -1);
        if (res == 0) {
            printf("timeout\n");
        } else {
            int i = 0;
            for (; i < MAXSIZE; i++) {
                if (fds[i] == -1) {
                    continue;
                }
                if (FD_ISSET(fds[i], &fdset)) {

                    if (fds[i] == sockfd) {
                        int c = accept(sockfd, (struct sockaddr *)&client_addr, &clen);
                        if (c >= 0) {
                            // 找到一个空的设置成新的套接字
                            for (int k = 0; k < MAXSIZE; k++) {
                                if (fds[i] == 0) {
                                    fds[i] = c;
                                    break;
                                }
                            }
                        }
                    } else {
                        char buff[256] = {0};
                        int n = read(fds[i], buff, 255);
                        if (n > 0) {
                            printf("read:%s\n", buff);
                            write(fds[i], "OK", 2);
                        } else if (n == 0) {
                            // 删除套接字
                            fds[i] = 0;
                        }

                    }
                }
            }

        }
    }
}

```

这个代码中有不完善的地方：使用数组保存套接字，建议以链表的形式保存链表会更好一些；


优点：跨平台

缺点：

* 单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这个限制，但是这样也会造成效率的降低；
* 每次都要调用 select ，都需要把 `fd` 集合从用户态拷贝到内核态，在fd很多时开销会很大；
* 每次调用 select 都需要在内核遍历传递进来的所有fd，在fd很多时开销也很大；

> 注意，每次调用select之前都要对fdset集合进行 FD_ZERO(&fdset) 操作，即清空。

> 参考文章
[linux的I/O复用技术](http://wangfakang.github.io/linux0/)


### poll

```cpp
int poll(struct pollfd *fds, unsigned int nfds, int timesout);

```

函数参数：

1. 表示一个`pollfd`结构的数组。用来保存想要监听的文件描述符及其注册（绑定）的相应事件
2. 表示监听事件集合的大小
3. 指定poll的超时值。当timeout为-1时，就会一直阻塞，直到某个事件发生；当timeout为0时，表示立即返回。


返回值：

当为-1的时候表示失败，当为0的时候表示超时，当为大于0的整数的时候表示执行成功，表示文件描述符的个数。


不同与select使用三个位图来表示三个fdset的方式，poll使用一个 pollfd的指针实现。

```cpp
struct pollfd {
    int fd; /* file descriptor */
    short events; /* requested events to watch */
    short revents; /* returned events witnessed */
};

```

该结构里包含了要监视等待的event和实际发生的event；

经常检测的事件标记：

* POLLIN/POLLRDNORM：可读
* POLLOUT/POLLWRNORM：可写
* POLLERR：出错


合法的事件标记如下：

* POLLIN：              有数据可读
* POLLRDNORM：       有普通数据可读
* POLLRDBAND：        有优先数据可读
* POLLPRI：              有紧迫数据可读
* POLLOUT：             写数据不会导致阻塞
* POLLWRNORM：       写普通数据不会导致阻塞
* POLLWRBAND：        写优先数据不会导致阻塞
* POLLMSG SIGPOLL：    消息可用

`POLLIN | POLLPRI`等价于select()的读事件，`POLLOUT |POLLWRBAND`等价于select()的写事件。`POLLIN`等价于`POLLRDNORM |POLLRDBAND`，而`POLLOUT`则等价于`POLLWRNORM`。

从原理上看，`select` 和 `poll` 都需要在返回以后，通过遍历文件描述符来获取已经就绪的socket。但是和select不同的是，调用这个函数后，系统不用清空它所检测的socket描述符集合；


因此`select函数`适合于只检测少量socket描述符的情况，而`poll函数`适合于大量socket描述符的情况；

```cpp
#include <unistd.h>
#include <sys/poll.h>
#include <sys/time.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <poll.h>
#define OPEN_MAX 100

int main(int argc, char *argv[])
{
    //1.创建tcp监听套接字
    int sockfd = ::socket(AF_INET, SOCK_STREAM, 0);

    //2.绑定sockfd
    struct sockaddr_in my_addr;
    bzero(&my_addr, sizeof(my_addr));
    my_addr.sin_family = AF_INET;
    my_addr.sin_port = htons(8000);
    my_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    bind(sockfd, (struct sockaddr *)&my_addr, sizeof(my_addr));

    //3.监听listen
    listen(sockfd, 10);

    //4.poll相应参数准备
    struct pollfd client[OPEN_MAX];
    int i = 0, maxi = 0;
    for(;i<OPEN_MAX; i++)
        client[i].fd = -1;//初始化poll结构中的文件描述符fd

    client[0].fd = sockfd;//需要监测的描述符
    client[0].events = POLLIN;//普通或优先级带数据可读

    //5.对已连接的客户端的数据处理
    while(1)
    {
        int ret = ::poll(client, maxi+1, -1);//对加入poll结构体数组所有元素进行监测

        if (ret == -1) {
            cout << "poll failed" << endl;
            continue;
        }

        //5.1监测sockfd(监听套接字)是否存在连接
        if((client[0].revents & POLLIN) == POLLIN )
        {
            struct sockaddr_in cli_addr;
            int clilen = sizeof(cli_addr);
            int connfd = 0;
            //5.1.1 从tcp完成连接中提取客户端
            connfd = ::accept(sockfd, (struct sockaddr *)&cli_addr, &clilen);

            //5.1.2 将提取到的connfd放入poll结构体数组中，以便于poll函数监测
            for(i=1; i<OPEN_MAX; i++)
            {
                if(client[i].fd < 0)
                {
                    client[i].fd = connfd;
                    client[i].events = POLLIN;
                    break;
                }
            }

            //5.1.3 maxi更新
            if(i > maxi)
                maxi = i;
        }

        //5.2继续响应就绪的描述符
        for(i=1; i<=maxi; i++)
        {
            if(client[i].fd < 0)
                continue;

            if(client[i].revents & (POLLIN | POLLERR))
            {
                int len = 0;
                char buf[128] = "";

                //5.2.1接受客户端数据
                if((len = recv(client[i].fd, buf, sizeof(buf), 0)) < 0)
                {
                    if(errno == ECONNRESET)//tcp连接超时、RST
                    {
                        close(client[i].fd);
                        client[i].fd = -1;
                    }
                    else
                        cout << "read error:" << endl;

                }
                else if(len == 0)//客户端关闭连接
                {
                    close(client[i].fd);
                    client[i].fd = -1;
                }
                else {//正常接收到服务器的数据
                    ::send(client[i].fd, buf, len, 0);
                }

                //5.2.2所有的就绪描述符处理完了，就退出当前的for循环，继续poll监测
                if(--ret <= 0)
                    break;

            }
        }
    }
}

```

### kqueue

```cpp
int kqueue(void);

```

生成一个内核事件队列，返回该队列的文件描述符，其它API通过这个描述符操作这个 `kqueue`，结构如下：

![](http://odwv9d2u8.bkt.clouddn.com/17-4-17/86926431-file_1492429406045_11cc.gif)


```cpp
struct kevent {
    uintptr_t ident; //事件ID，一般为文件描述符
    short filter; //事件过滤器
    u_short flags; //行为标示
    u_int fflags; //过滤器标识值
    intptr_t data; //过滤器数据
    void *udata; //应用透传数据
};


int kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);

```

提供向内核注册／反注册事件和返回就绪事件或错误事件；在一个kqueue中，｛ident，filter｝确定一个唯一的事件；

函数参数：

1. kq：kqueue的文件描述符
2. changelist：注册／反注册的事件数组
3. nchanges：changelist的元素个数
4. eventlist：满足条件的通知事件数组
5. nevents：eventlist的元素个数
6. timeout：等待事件到来时的超时时间

返回值为**可用事件的个数**

kqueue不光能够处理socket的事件，同时还能处理异步io，信号，文件变化等等；

kqueue有两个部分，分别是kqueue和kevent；kqueue主要是用来描述event的队列，而kevent则是监听的事件；

通过`kevent`提供三个主要的行为功能，分别是

* 注册／反注册
    
    注意`kevent`中的`neventlist`这个输入参数，当其设为0，且传入合法的`changelist和nchanges`，就会将 `changelist` 中的事件注册到 kqueue 中；

* 允许／禁止过滤器事件
    
    通过`flags EV_ENABLE 和 EV_DISABLE` 使过滤器事件有效或者无效，这个功能在使用 `EVFILT_WRITE` 发送数据时非常有用；

* 等待事件通知

    将 `nchangelist 和 nchanges` 设置成 `null和0` ，当kevent非错误和超时返回时，在 `eventlist和nevents` 中保存可用事件集合。


#### 实现

```cpp
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/event.h>
#include <sys/time.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <errno.h>

#define PORT 5001
#define MAX_EVENT_COUNT 64

int createSocket()
{
    int sock = socket(PF_INET, SOCK_STREAM, 0);
    if (sock == -1)
    {
        printf("socket() failed:%d\n",errno);
        return -1;
    }
    
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(PORT);
    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
    
    int optval = 1;
    setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval));
    optval = 1;
    setsockopt(sock, SOL_SOCKET, SO_NOSIGPIPE, &optval, sizeof(optval));
    
    if (bind(sock, (struct sockaddr*)&addr, sizeof(struct sockaddr)) == -1)
    {
        printf("bind() failed:%d\n",errno);
        return -1;
    }
    
    if (listen(sock, 5) == -1)
    {
        printf("listen() failed:%d\n",errno);
        return -1;
    }
    
    return sock;
}

int main(int argc, const char * argv[])
{    
    int listenfd = createSocket();
    if (listenfd == -1)
        return -1;
    
    int kq = kqueue();
    if (kq == -1)
    {
        printf("kqueue failed:%d",errno);
        return -1;
    }
    
    struct kevent event = {listenfd,EVFILT_READ,EV_ADD,0,0,NULL};
    int ret = kevent(kq, &event, 1, NULL, 0, NULL);
    if (ret == -1)
    {
        printf("kevent failed:%d",errno);
        return -1;
    }
    
    while (true)
    {
        struct kevent eventlist[MAX_EVENT_COUNT];
        struct timespec timeout = {5,0};
        int ret = kevent(kq, NULL, 0, eventlist, MAX_EVENT_COUNT, &timeout);
        if (ret <= 0)
            continue;
        
        for (int i=0; i<ret; i++)
        {
            struct kevent event = eventlist[i];
            int sock = (int)event.ident;
            int16_t filter = event.filter;
            uint32_t flags = event.flags;
            intptr_t data = event.data;
            
            //有新的客户端链接
            if (sock == listenfd)
            {
                socklen_t client_addrlen = 4;
                struct sockaddr client_addrlist[client_addrlen];
                int clientfd = accept(listenfd, client_addrlist, &client_addrlen);
                if (clientfd > 0)
                {
                    struct kevent changelist[2];
                    EV_SET(&changelist[0], clientfd, EVFILT_READ, EV_ADD, 0, 0, NULL);
                    EV_SET(&changelist[1], clientfd, EVFILT_WRITE, EV_ADD, 0, 0, NULL);
                    kevent(kq, changelist, 1, NULL, 0, NULL);
                }
                continue;
            }
            
            //异常事件
            if (flags & EV_ERROR)
            {
                close(sock);
                struct kevent event = {sock,EVFILT_READ,EV_DELETE,0,0,NULL};
                kevent(kq, &event, 1, NULL, 0, NULL);
                printf("socket broken,error:%ld\n",data);
                continue;
            }
            
            //数据可读
            if (filter == EVFILT_READ)
            {
                char buffer[data];
                memset(buffer, '\0', data);
                ssize_t recvlen = recv(sock, buffer, data, 0);
                if (recvlen <= 0)
                {
                    //链接断开
                    close(sock);
                    struct kevent event = {sock,EVFILT_READ,EV_DELETE,0,0,NULL};
                    kevent(kq, &event, 1, NULL, 0, NULL);
                    printf("socket broken!\n");
                    continue;
                }
                
                printf("%s\n",buffer);
            }
            
            //数据可写
            if (filter == EVFILT_WRITE)
            {
                char buffer[data];
                memset(buffer, 'a', data);
                ssize_t sendlen = send(sock, buffer, data, 0);
                if (sendlen <= 0)
                {
                    //链接断开
                    close(sock);
                    struct kevent event = {sock,EVFILT_READ,EV_DELETE,0,0,NULL};
                    kevent(kq, &event, 1, NULL, 0, NULL);
                    printf("socket broken!\n");
                    continue;
                }
            }
            
        }
    }
    
    return 0;
}

```

#### 不同

和前面不同的是，kqueue不会像select或者poll一样每隔一段事件就去轮询所有的socket，当socket数量很多，但是很多socket都不活跃的时候，性能是有影响的，而kqueue只会关注事件发生的socket；

### epoll

#### 函数

* 创建事件表

```cpp
int epoll_create(int size);

```

创建一个epoll的句柄，**参数 size 并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的建议**，不同于select中的给出最大监听的`fd+1`。

* 操作事件表

```cpp
int epoll_ctl(int epfd, int op, int fd, struct epoll_event* event);

```

函数参数

1. epfd：事件表的文件描述符
2. op：何种操作，包括 `EPOLL_CTL_ADD，EPOLL_CTL_DEL，EPOLL_CTL_MOD`，分别实现对fd的监听事件进行添加、删除、修改
3. fd：需要监听的文件描述符
4. event：告诉内核需要监听什么事
    
    epoll_event 结构如下：

    ```cpp
    struct epoll_event {
      __uint32_t events;  /* Epoll events */
      epoll_data_t data;  /* User data variable */
    };
    
    //events可以是以下几个宏的集合：
    EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
    EPOLLOUT：表示对应的文件描述符可以写；
    EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
    EPOLLERR：表示对应的文件描述符发生错误；
    EPOLLHUP：表示对应的文件描述符被挂断；
    EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
    EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
    
    ```

* 监听相应事件

```    
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)

```

函数参数：

1. epfd：事件表的文件描述符
2. events：从内核得到事件的集合
3. maxevents：事件集合的大小（不能大于创建时的`size`）
4. timeout：超时时间


#### 工作模式

　epoll对文件描述符的操作有两种模式：LT（level trigger）和ET（edge trigger）。LT模式是默认模式，LT模式与ET模式的区别如下：
　
* LT模式：当epoll\_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

* ET模式：当epoll\_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

> ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用**非阻塞套**接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。


当使用epoll的ET模型来工作时，当产生了一个`EPOLLIN`事件后， 读数据的时候需要考虑的是当recv()返回的大小如果等于请求的大小，那么很有可能是缓冲区还有数据未读完，也意味着该次事件还没有处理完，所以还需要再次读取：

```cpp
void handle_rev() {
    while(rs){
        buflen = ::recv(activeevents[i].data.fd, buf, sizeof(buf), 0);
        if(buflen < 0){
            // 由于是非阻塞的模式,所以当errno为EAGAIN时,表示当前缓冲区已无数据可读
            // 在这里就当作是该次事件已处理处.
            if(errno == EAGAIN){ //EAGAIN经常出现在当应用程序进行一些非阻塞(non-blocking)操作(对文件或socket)的时候
                break;
            }
            else{
                return;
            }
        }
        else if(buflen == 0){
            // 这里表示对端的socket已正常关闭.
        }

        if(buflen == sizeof(buf){
            rs = 1;   // 需要再次读取
        }
                else{
            rs = 0;
        }
    }
}

```

有时候epoll不一定比select和poll的效率高，比如这样的场景下：当活动连接数比较高的时候此时epoll会经常触发回调函数 ，此时在性能上还是有一定的损失．epoll适用于连接数量多，但是活跃的连接少．

#### 实现

```

epollserver::epollserver(int af, int type, int protocol) : norserver(af, type, protocol) {
    this->_epollfd = ::epoll_create(MAX_SIZE);
    if (this->_epollfd == INVALID_SOCKTE) {
        cout << "epoll create failed" << endl;
    }
}

epollserver::~epollserver() {
    this->close(this->socket());
}

void epollserver::wait_events() {
    struct epoll_event _events[EPOLL_EVENTS_NUM];
    this->add_event(this->socket(), EPOLLIN);

    while (true) {
        int ret = ::epoll_wait(this->_epollfd, _events, EPOLLEVENTS, -1);
        this->handle_events(_events, ret);
    }
}

void epollserver::handle_events(struct epoll_event* events, int num) {
    for (int i = 0; i < num; i++) {
        int socket = events[i].data.fd;
        // 服务器本身
        if (socket == this->socket()) {
            this->handle_accept();
        }
        else if (events[i].events & EPOLLIN) {
            this->handle_read(socket);
        }
        else if (events[i].events & EPOLLOUT) {
            this->handle_write(socket);
        }
    }
}

void epollserver::handle_accept() {
    this->accept();
}

void epollserver::handle_read(int socket) {
    int nread;
    char buf[MAX_SIZE];
    nread = ::read(socket, buf, MAX_SIZE);
    if (nread == SOCKET_ERROR)     {
        cout << "read error:" << endl;
        this->close(socket); //记住close fd
        delete_event(socket, EPOLLIN); //删除监听
    }
    else if (nread == 0)     {
        fprintf(stderr,"client close.\n");
        this->close(socket); //记住close fd
        delete_event(socket, EPOLLIN); //删除监听
    }
    else {
        cout << "read message is :" << buf;
        //修改描述符对应的事件，由读改为写
        modify_event(socket, EPOLLOUT);
    }
}

void epollserver::handle_write(int socket) {
    int nwrite;
    char buf[MAX_SIZE];
    nwrite = ::write(socket, buf, strlen(buf));
    if (nwrite == -1){
        cout << "write error:" << endl;
        this->close(socket);   //记住close fd
        delete_event(socket, EPOLLOUT);  //删除监听
    }else{
        modify_event(socket, EPOLLIN);
    }
    memset(buf,0, MAX_SIZE);
}

bool epollserver::add_event(int socket, int state) {
    struct epoll_event ev;
    ev.events = state;
    ev.data.fd = socket;
    if (!epoll_ctl(this->_epollfd, EPOLL_CTL_ADD, socket, fd, &ev)) {
        cout << "epoll add event failed" << endl;
        return false;
    }
    return true;
}

bool epollserver::delete_event(int socket, int state) {
    struct epoll_event ev;
    ev.events = state;
    ev.data.fd = socket;
    if (!epoll_ctl(this->_epollfd, EPOLL_CTL_DEL, socket, fd, &ev)) {
        cout << "epoll delete event failed" << endl;
        return false;
    }
    return true;
}

bool epollserver::modify_event(int socket, int state) {
    struct epoll_event ev;
    ev.events = state;
    ev.data.fd = socket;
    if (!epoll_ctl(this->_epollfd, EPOLL_CTL_MOD, socket, fd, &ev)) {
        cout << "epoll modify event failed" << endl;
        return false;
    }
    return true;
}

```

> 参考文章
[Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859#articleHeader17)