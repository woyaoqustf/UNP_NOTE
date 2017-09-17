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
