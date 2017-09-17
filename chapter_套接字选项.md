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
    5. read，有还有数据未处理，将先返回未处理的数据，无返回errorno置为so_error ,then重置为0  
    6. write 有错errno 置为 so_error,then 重置为0  
    7. 通过getsockopt获取，返回相应值，重置0  
    8. 在异步多线程的情况先，使用SO_ERROR的方式合理，应为errno可能被多个线程覆盖  
