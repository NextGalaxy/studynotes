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
    