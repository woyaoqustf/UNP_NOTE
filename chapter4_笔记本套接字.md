socket 函数
===========
```
int socket(int family, int type, int protocol); // 成功返回fd，失败 -1
```
* protocol 写0 ，系统根据family type 设置默认协议
* SOCKET_STREAM * AF_INET AF_INET = TCP
* SOCKET_DGRM *  AF_INET AF_INET = UDP
* SOCKET_RAW *  AF_INET AF_INET = IPV4,IPV6

connect
=======
```
int connect(int fd, sockaddr * serveraddr, sock_len_t addrlen);// 成功返回 0，失败-1
```
* 客户端想server发起连接，在三次握手成功/失败后才会返回(NOBLOCK??)
* 内核自动选择端口
* 失败的原因有三种：
  失败的话，就是三次握手失败  
  1. SYN 没有ACK,则分4，24 最多75s ETIMEOUT  
  2. RST 主机找到了，没有监听端口，我自己理解叫端口不可达  
  3. ICPM 路由路径不通， ENETUNREACH,EHOSTUNREACH , `OR connect不等待就返回 （NOBLOCK??）`  
* fd在连接失败后，不能重用必须关闭**（why 待究）**
