# 第4章 LInux网络编程

## socket介绍

### 所谓socket(套接字)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。

## 字节序

### 字节的顺序

### 小端字节序

- 高位放高地址

### 大端字节序

- 高位放低地址

### 网络字节序

- 均是大端

### 主机字节序

- 可能是大端 可能是小端

## socket函数

### int socket(int domain,int type,int protocol)

- 功能：创建一个套接字
- 参数：

	- - domain：协议族

		- AF_INET：ipv4
		- AF_INET6:ipv6
		- AF_UNIX,AF_LOCAL: 本地套接字通信(进程间通信)

	- - type：通信过程中使用的协议类型

		- SOCK_STREAM:流式协议
		- SOCK_DGRAM:报式协议

	- - protocol：具体的一个协议。一般写0

		- - SOCK_STREAM：流式协议默认使用TCP
		- - SOCK_DGRAM：报式协议默认使用UDP

- 返回值

	- - 成功，返回文件描述符，操作的就是内核缓冲区
	- - 失败：-1

### int bind(int socketfd,const struct sockaddr *addr,socklen_t addrlen);

- 功能：绑定，将fd和本地IP+端口进行绑定（socket 命名）
- 参数

	- - sockfd

		- 通过socket函数得到的文件描述符

	- - addr

		- 需要绑定的socket地址，这个地址封装了ip和端口号的信息

	- -addrlen

		- 第二个参数结构体占的内存大小

### int listen(int sockfd,int backlog);

- 功能：监听这个socket上的连接
- 参数

	- - sockfd:通过socket()函数得到的文件描述符
	- - backlog: 未连接的和已连接的和的最大值，5

- 底层创建两个队列

	- 未连接队列
	- 已连接队列

### int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen);

- 功能：接收客户端连接，默认是一个阻塞的函数，阻塞等待客户端连接
- - 参数

	- - socketfd：通过socket函数得到的文件描述符
	- - addr：需要绑定的socket地址，这个地址封装了ip和端口号的信息
	- - addrlen：第二个参数结构体占的内存大小

- - 返回值

	- - 成功：用于通信的文件描述符
	- -  -1：失败

### int connect(int sockfd, const struct sockaddr *addr,socklen_t addrlen);

- - 功能 ：客户端连接服务器
- - 参数 ：

	- - sockfd：用于通信的文件描述符
	- - addr：客户端要连接的服务器的地址信息
	- - addrlen：第二个参数的内存大小

- - 返回值

	- 成功：0
	- 失败：-1

## TCP三次握手

### 第一次握手

- 1.客户端将SYN标志位置为1
- 2.生成一个随机的32位的序号 seq = J，这个序号后边是可以携带数据（数据的大小）

### 第二次握手

- 1.服务器端接收客户端的连接：ACK = 1
- 2.服务器会回发一个确认序号： ack = 客户端的序号 + 数据长度 + SYN/FIN(按一个字节算)
- 3.服务器端会向客户端发起连接请求：SYN = 1
- 4.服务器会生成一个随机序号：seq = K

### 第三次握手

- 1.客户端应答服务器的连接请求：ACK = 1
- 2.客户端回复收到了服务器端的数据：ack = 服务端的序号 + 数据长度 + SYN/FIN(按一个字节算)

### 第一次握手不能带通信数据!

### 第三次握手可以携带数据!

## 滑动窗口

### 滑动窗口为缓冲区的大小

### 滑动窗口的大小会随着发送数据和接收数据而变化

### 通信的双方都有发送缓冲区和接收数据的缓冲区

## TCP四次挥手

### 四次挥手发生在断开连接的时候，在程序中当调用了close()会使用TCP协议进行四次挥手。

### 客户端和服务器端都可以主动发起断开连接，谁先调用close()谁就是发起。

### 因为在TCP连接的是时候，采用三次挥手建立的连接是双向的，在断开的时候需要双向断开

### 主动断开的一方有 2MSL(最大报文生存时间)

## 半关闭

### 当TCP链接中A向B发送FIN请求关闭，另一端B回应ACK之后(A端进入FIN_WAIT_2状态)，并没有立即发送FIN给A,A方处于半连接状态(半开关)，此时A可以接收B发送的数据，但是A已经不能再向B发送数据

### int shutdown(int sockfd,int how);

## 端口复用

### 用途

- 1.防止服务器重启时之前绑定的端口还未释放
- 2.程序突然退出而系统没有释放端口

### netstat

- 参数

	- -a:所有的socket
	- -p:显示正在使用socket的程序的名称
	- -n:直接使用ip地址，而不通过域名服务器

### 端口复用，设置的时机是在服务器绑定端口之前

## I/O多路复用(I/O多路转接)

### I/O多路复用使得程序能同时监听多个文件描述符，能够提高程序的性能，Linux下实现I/O多路复用的系统调用有select、poll和epoll

### 阻塞等待

- 好处

	- 不占用CPU宝贵的时间片

- 缺点

	- 同一时刻只能处理一个操作，效率低

- 多线程或者多进程解决

	- 缺点

		- 1.线程或者进程会消耗资源
		- 2.线程或进程调度消耗CPU资源

### 非阻塞，忙轮询

- 每隔一分钟催一次
- 优点

	- 提高了程序的执行效率

- 缺点

	- 需要占用更多的CPU和系统资源

### NIO模型

- 1W Client 每循环内O(n)系统调用

### 第一种：select/poll

- select代收员比较懒，她只会告诉你有几个快递到了，但是哪个快递，你需要挨个遍历一遍

### 第二种：epoll

- epoll代收快递员很勤快，她不仅会告诉你有几个快递到了，还会告诉你是哪个快递公司的快递

### select

- 主旨思想

	- 1.首先要构造一个关于文件描述符的列表，将要监听的文件描述符添加到该列表中。
	- 2.调用一个系统函数，监听该列表中的文件描述符，直到这些描述符中的一个或者多个进行I/O操作时，该函数才返回

		- a.这个函数是阻塞
		- b.函数对文件描述符的检测的操作是由内核完成的

	- 3.在返回时，它会告诉进程有多少(哪些)描述符要进行I/O操作

- int select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,struct timeval *timeout);

	- 参数

		- - nfds：委托内核检测的最大文件描述符的值+1
		- - readfds：要检测的文件描述符的读的集合，委托内核检测哪些文件描述符的读的属性

			- 一般检测读操作
			- 对应的是对方发送过来的数据，因为读是被动的接收数据，检测的就是读缓冲区
			- 是一个传入传出参数

		- - writefds：要检测的文件描述符的写的集合，委托内核检测哪些文件描述符的写的属性

			- 委托内核检测写缓冲区是不是还可以写数据(不满的就可以写)

		- - exceptfds：检测发生异常的文件描述符的集合
		- - timeout：设置的超时时间

			- struct timeval{
long tv_sec;
long tv_usec;
};

				- -NULL：永久阻塞，直到检测到了文件描述符有变化
				- 两个=0：不阻塞
				- 两个>0：阻塞对应的时间

	- 返回值

		- -1：失败
		- >0(n):检测的集合中有n个文件描述符发生了变化

- select()工作过程分析

	- 客户端A,B,C,D连接到服务器分别对应文件描述符3,4,100,101
	- fd_set reads;

		- 将这4个标志位置为1

	- select(101+1,&reads,NULL,NULL,NULL);

		- +1因为从0开始遍历的

	- A,B发送了数据

		- 检测到3和4为 1

			- 拷贝回来

				- 只有3和4置为1

- select缺点

	- 1.每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
	- 2.同时每次调用select都需要在内核遍历传递进来的所有fd,这个开销在fd很多时也很大
	- 3.select支持的文件描述符数量太小了，默认是1024
	- 4.fds集合不能重用，每次都需要重置

### poll

- int poll(struct pollfd *fds,nfds_t nfds,int timeout);

	- - 参数

		- - fds：是一个struct pollfd结构体数组，这是一个需要检测的文件描述符的集合
		- - nfds: 这是第一个参数数组中最后一个有效元素的下标+1
		- - timeout：阻塞时长

			- 0：不阻塞
			- -1：阻塞，当检测到需要检测的文件描述符有变化，解除阻塞
			- >0：阻塞时长

	- - 返回值

		- -1：失败
		- >0(n)：成功，n表示检测到集合中有n个文件描述符发生变化

- struct pollfd{
      int fd;
      short events;
      short revents;
}

	- fd:委托内核检测的文件描述符
	- events:委托内核检测文件描述符的什么事件
	- revents:文件描述符实际发生的事件

- 与select区别

	- 不需要重用数组，直接改revents
	- 没有1024限制，select只有128个字节，代表1024位
	- 其他缺点都有

### epoll

- struct eventpoll{
     ...
     struct rb_root rbr;
     struct list_head rdlist;
     ...
};
- int epoll_create(int size);

	- 创建一个新的epoll实例。在内核中创建了一个数据，这个数据中有两个比较重要的数据，一个是需要检测的文件描述符的信息(红黑树)，还有一个是就绪列表，存放检测到数据发生改变的文件描述符信息(双向链表)
	- - 参数

		- size:目前没有意义了，必须大于0

	- - 返回值

		- -1：失败
		- >0：文件描述符，操作epoll实例的

- int epoll_ctl(int epfd,int op,int fd,struct epoll_event *event)

	- 作用：对epoll实例进行管理，添加文件描述符信息，删除信息，修改信息
	- - 参数

		- - epfd：epoll实例对应的文件描述符
		- - op：要进行什么操作

			- EPOLL_CTL_ADD

				- 添加

			- EPOLL_CTL_MOD

				- 修改

			- EPOLL_CTL_DEL

				- 删除

		- - fd :要检测的文件描述符
		- - events：检测文件描述符什么事情

- int epoll_wait(int epfd,struct epoll_event *events,int maxevents,int timeout);

	- - 参数

		- - epfd：epoll实例对应的文件描述符
		- - events：传出参数，保存了发生了变化的文件描述符的信息
		- - maxevents：第二个参数结构体数组的大小
		- -timeout：阻塞时间

			- -0：不阻塞
			- - -1：阻塞，直到检测到fd数据发生变化，解除阻塞
			- - >0：阻塞的时长(毫秒)

	- - 返回值

		- -成功,返回发生变化的文件描述符的个数 > 0
		- - 失败，返回-1

- Epoll的工作模式

	- LT模式(level - triggered)  水平触发

		- 缺省的工作方式，并且同时支持block和no-block socket。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的
		- 假设委托内核检测读事件 -> 检测fd的读缓冲区

			- 读缓冲区有数据 -> epoll 检测到了会给用户通知
			- a.用户不读数据，数据一直在缓冲区，epoll会一直通知
			- b.用户只读了一部分数据，epoll会通知
			- c.缓冲区数据读完了，不通知

	- ET模式(edge - triggered) 边沿触发

		- 只支持非阻塞socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了。（only-once）
		- 非阻塞是为了 read多读时候 防止 阻塞了！！！
		- 假设委托内核检测读事件->检测fd的读缓冲区

			- 读缓冲区有数据 -> epoll检测到了会给用户通知
			- a.用户不读数据，数据一直在缓冲区中，epoll下次检测的时候就不通知了
			- b.用户只读了一部分数据，epoll不通知
			- c.缓冲区的数据读完了，不通知

## UDP通信

### ssize_t sendto(int sockfd,const void *buf,size_t len,int flags,const struct sockaddr *dest_addr,socklen_t addrlen);

- 参数

	- - sockfd：通信的fd
	- -buf：要发送的数据
	- -len：发送数据的长度
	- -flags:0
	- - dest_addr：通信的另外一端的地址信息
	- - addrlen：地址的内存大小

### ssize_t recvfrom(int sockfd,void *buf,size_t len,int flags,struct sockaddr *src_addr,socklen_t *addrlen);

- -参数

	- - sockfd：通信的fd
	- - buf：接收数据的数组
	- - len：数组的大小
	- - flags：0
	- - src_addr：用来保存另外一端的地址信息，不需要可以指定为NULL
	- - addrlen：地址的内存大小

### 广播

- 向子网中多台计算机发送消息，并且子网中所有的计算机都可以接收到发送方的消息，每个广播消息都包含一个特殊的IP地址，这个IP地址中子网内主机标志部分的二进制全部为1

	- a.只能在局域网中使用
	- b.客户端需要绑定服务器广播使用的端口，才可以接收到广播消息

- int setsockopt(int sockfd,int level,int optname,const void *optval,socklen_t optlen);

	- - sockfd：文件描述符
	- - level：SOL_SOCKET
	- - optname:SO_BROADCAST
	- - optval：int类型的值，为1表示允许广播
	- - optlen：optval的大小

### 组播（多播）

- 单播地址标识单个IP接口，广播地址标识某个子网的所有IP接口，多播地址标识一组IP接口。
- 单播和广播是寻址方案的两个极端(要么单个要么全部)，多播则意在两者之间提供一种折中方案。
- 多播数据报只应该由对它感兴趣的接口接收，也就是说由运行相应多播会话应用系统的主机上的接口接收。另外，广播一般局限于局域网内使用，而多播则既可以用于局域网，也可以跨局域网使用

	- a.组播既可以用于局域网，也可以用于广域网
	- b.客户端需要加入多播组，才能接收到多播的数据

- int setsockopt(int sockfd,int level,int optname,const void *optval,socklen_t optlen);

	- 服务器设置多播的信息，外出接口

		- - level：IPPROTO_IP
		- - optname:IP_MULTICAST_TF
		- - optval：struct in_addr

	- 客户端加入到多播组

		- - level：IPPROTO_IP
		- - optname：IP_ADD_MEMBERSHIP
		- - optval：struct mreqn

## 本地套接字

### 作用

- 本地的进程间通信

	- 有关系的进程间的通信
	- 没有关系的进程间的通信

### 本地套接字实现流程和网络套接字类似，一般采用TCP的通信流程

### 服务器端

- 1.创建监听的套接字

	- int lfd = socket(AF_UNIX/AF_LOCAL,SOCK_STREAM,0);

- 2.监听的套接字绑定本地的套接字文件

	- struct sockaddr_un addr;
	- 绑定成功之后，指定的sun_path中的套接字文件会自动生成
	- bind(lfd,addr,len);

- 3.监听

	- listen(lfd,100);

- 4.等待并接收连接请求

	- struct sockaddr_un cliaddr;
	- int cfd = accept(lfd,&cliaddr,len);

- 5.通信

	- 接收数据：read/recv
	- 发送数据：write/send

- 6.关闭连接

	- close();

### 客户端流程

- 1.创建通信的套接字

	- int lfd = socket(AF_UNIX/AF_LOCAL,SOCK_STREAM,0);

- 2.监听的套接字绑定本地的IP端口

	- struct sockaddr_un addr;
	- 绑定成功之后，指定的sun_path中的套接字文件会自动生成

- 3.连接服务器

	- struct sockaddr_un serveraddr;
	- connect(fd,&serveraddr,sizeof(serveraddr));

- 4.通信

	- 接收数据：read/recv
	- 发送数据：write/send

- 5.关闭连接

	- close();

