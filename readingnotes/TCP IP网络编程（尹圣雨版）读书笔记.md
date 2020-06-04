### 套接字
#### 服务端处理
 * 生成socket套接字方式
    ```
    #include <sys/socket.h>
    // 成功时返回文件描述符，失败返回-1
    int socket(int domain, int type, int protocol);
    ```
* 绑定地址信息 bind
    ```
    #include <sys/socket.h>
    // 成功时返回0，失败-1
    int bind(int sockfd, struct sockaddr* addr, socklen_t addrlen);
    ```
* 监听 listen
    ```
    #include <sys/socket.h>
    // 成功时返回0，失败-1
    int listen(int sockfd, int backlog);
    ```
* 接收连接请求 accept
    ```
    #include <sys/socket.h>
    // 这里的addr和addrlen是输出参数，当accept成功并返回文件描述符时，
    // 客户端地址信息会存储在addr中
    int accept(int sockfd, struct sockaddr* addr, socklen_t* addrlen);
    ```

* #### 简言之，网络编程中服务端接受连接请求的套接字创建过程如下
    1. 调用socket函数创建套接字
    2. 调用bind函数分配监听的ip地址和端口
    3. 调用listen函数使程序进入监听状态
    4. 调用accept函数受理连接请求

#### 客户端处理，客户端连接请求相对于服务端接受连接请求要简单一些了
* 生成socket套接字方式
    ```
    #include <sys/socket.h>
    // 成功时返回文件描述符，失败返回-1
    int socket(int domain, int type, int protocol);
    ```
* 连接服务器 connect
    ```
    #include <sys/socket.h>
    // 成功时返回0，失败返回-1
    int connect(int sockfd, struct sockaddr* addr, socklen_t addrlen);
    ```
* #### 简言之，客户端发起连接就只有两个步骤
    1. 调用socket函数创建套接字
    2. 调用connect函数连接服务端


### IO复用
#### 复用含义
* 在一个通信频道中传递多个数据的技术
* 为了提高物理设备的效率，用最少的物理设备传递最多数据使用的技术

#### select 
* 方法定义：int select(int nfds, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, struct timeval *timeout)
* 使用方法和顺序：
    1. 设置文件描述符，指定监听范围，设置超时
        + 通过fd_set数组变量来表示文件描述符集合，该数组是位数组
        + FD_ZERO(fd_set* fdset): 将fd_set变量的所有位初始化为0
        + FD_SET(int fd, fd_set* fdset): 在参数fdset集合中注册文件描述符fd的信息
        + FD_CLR(int fd, fd_set* fdset): 在参数fdset集合中清除文件描述符fd的信息
        + FD_ISSET(int fd, fd_set* fdset): 在参数fdset集合中查找是否存在文件描述符fd的信息
    2. 调用select函数
    3. 查看调用结果

* 优点
    1. 兼容性高，windows和linux都支持
* 缺点
    1. 速度较慢：调用select语句后常见的针对所有文件描述符的循环语句；每次调用select函数时都需要重新向select函数传递监视描述符集合信息


#### epoll
* 优点
    1. 无需为了查看状态变化而循环遍历文件描述符
    2. 在监控套接字状态函数epoll_wait中无需每次传递套接字集合信息

* 使用方法和主要函数
    * epoll_wait: 创建保存epoll文件描述符的空间
    * epoll_ctl: 向空间注册或注销文件描述符
    * epoll_wait: 与select函数类似，等待文件描述符发生变化

* 条件触发和边缘触发
    1. 只有理解和掌握了条件触发和边缘触发才算真正掌握了epoll    
        * 条件触发中，只要输入缓冲区中有数据就会一直通知该事件；
        * 边缘触发中，只在输入缓冲区中收到数据时触发一次事件，即使输入缓冲区还有数据未被读完而留有数据，也不会再进行事件注册
    2. 边缘触发中必知的两点
        1. 通过errno变量查看错误原因
        2. 为了完成非阻塞（Non-blocking）IO, 需要更改套接字特性
    3. 更改套接字为非阻塞的方式
        ```
        int flag = fcntl(fd, F_GETFL, 0);
        fcntl(fd, F_SETFL, flag|O_NONBLOCK);
        ```
    4. 边缘触发方式中，接收数据时仅注册一次该事件，所以一旦发生输入相关事件，就必须读取输入缓冲区中的全部数据，因此需要验证输入缓冲区是否为空（read函数返回-1，变量errno中的值为EAGAIN时,说明没有数据可读）
    5. 为什么在边缘触发方式下要将套接字设置成非阻塞模式呢？
        * 因为边缘触发方式下，以阻塞方式工作的read&write函数有可能引起服务器长时间停顿，因此边缘触发方式中一定要采用非阻塞read&write函数
    6. 边缘触发方式的回声服务器代码见: echo_epetserv.c
    