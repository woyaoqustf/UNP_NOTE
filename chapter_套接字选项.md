getsockopt,setsockopt
=====================
```
int getsockopt(int sockfd, int level, int optname, void *optval, sock_len *optlen);//成功返回0， 失败-1
int setsockopt(int sockfd, int level, int optname, void *optval, sock_len *optlen)

```
具体的参数关系为，那一层->那个选项名称->值

- level 选项塑造的协议、代码的层级，
  SOL_SOCKET  
  IPPROTP_TCP  
  IPPROTO_IP  
  IPPROTO_IPV6  
  IPPROTO_ICPMV6  
  IPPROTO_SCTP
  
- optname 结构体名字，设置的宏
- optval 设置/获取的选项值对应的结构体
- optlen optval的字节长度
- 若果不支持该选项，errno ENOPROTOOPT

通用套接字
=========
- SO_BREADCAST
- SO_DEBUG
    仅对TCP有效，在**内核环形缓冲区**，记录fd上结束发送的**分组**的详细信息
- SO_ERROR  
    1. 只可获取的选项  
    2. 内核在各套接字上有so_error变量  
    3. 将so_error 转换成EXX （pending error）  
    4. 套接字出错时：  
      1. 套接字阻塞在select上，读写有监听，都设置为满足  
      2. 多使用信号驱动的IO, 发送SIGIO  
    5. read，so_errno 不为0，还有数据未处理，将先返回未处理的数据，无 errno置为so_error ,then重置为0  
    6. write so_errno 不为0，errno 置为 so_error,then 重置为0  
    7. 通过getsockopt获取，返回相应值，重置0  
    8. 在异步多线程的情况先，使用SO_ERROR的方式合理，应为errno可能被多个线程覆盖 
    
   
 - SO_KEEPALIVE  
1. 2小时任一方向都无数据，发送探活分节数据， 对方**必须响应**  
2. 三种情况  
    1. 结束到对方回复，说明对方存活着，客户进程无感知
    2. 收到**RST**, 对方崩溃已经重启，so_errno 设置为ECONNRESET
    3. **超时**，正常每隔75s发送一次，直到有回复，或者吵过最大等待时间（10分钟默认），返回ETIMEOUT
    4. ICPM 返回错误码，如主机不可达（HOSTUNREACH一般网络不通，或者对方机器崩溃），返回相应的错误  
3. 2小时保活时间参数可改，但**一般内核都是维护全局保活时间**，一旦修改可能影响其他设置此参数的进程

TODO SO_LINGER
==============

TODO SO_SNDBUF SO_RCVBUF
========================

TODO SO_RCVLOWAT SO_SNDLOWAT
============================


SO_REUSEADDR SO_REUSEPORT
=========================

TCP 选项
========

TCP_MSS
========
设置TCP连接分节的最大的size，通常以对端SYN设置，如果设置的值小于对端的SYN则设置为相应的值

TCP_NODELAY
===========
目的在与减少广域网上小分节的数目
- 如果缓冲区有未ACK的分节，且缓冲区大小小于MSS则延迟发送，知道有ACK活分节大小大于等于MSS
- 如果数据都ACK了可立即发送
如果对端没有放反方向的数据传递，那么发送方必须等到对方ACK才能发送数据，造成数据，会有延迟


fctl
====
F_SET O_NONBLOCK  流程，先F_GET 或上O_NONBLOCK 然后F_SET  
F_SET A_SYN fd状态任何变化发送SIGIO到**属主进程** 进程id OR pgid  
F_ETOWN 后接正数，进程id ，**负数 进程组id**，返回属主（-1 出错）信号会单个进程或者**整个进程组**接受信号


    
    
