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
int connect(int fd, const struct sockaddr *serveraddr, socklen_t addrlen);// 成功返回 0，失败-1
```
* 客户端想server发起连接，~~在三次握手成功/失败后才会返回(NOBLOCK??)~~ 在收到server的ACK即返回
* 内核自动选择端口
* 失败的原因有三种：
  失败的话，就是三次握手失败  
  1. SYN 没有ACK,则分4，24 最多75s ETIMEOUT  
  2. RST 主机找到了，没有监听端口，我自己理解叫端口不可达  
  3. ICPM 路由路径不通， ENETUNREACH,EHOSTUNREACH , `OR connect不等待就返回 （NOBLOCK??）`  
* fd在连接失败后，不能重用必须关闭**(why 待究)**

accept
======
```
int accept(int fd, sockaddt *cliaddr, socklen_t *sock_len); // 成功返回新连接的fd，客户号地址，长度在cliaddr,sock_len中， 失败返回-1
```
* 从已完成队列头返回完成的连接，若`完成队列为空进程睡眠`

exec
====
* 当前进程内存镜像，替换成新的程序文件，原进程的内存数据、数据结果没了
* 将控制权交个新程序起点 一般main


close
=====
* 将文件标记为已关闭，立即返回进程
* 在进程中将不可用read，write不可读
* TCP 继续尝试发送数据到对端，发送完毕，按正常流程关闭连接
* **TODO** SO_LINGER
* 要是想真的`发送FIN可调用shutDown` ，**场景待究**

getsockname、getpeername
========================
```
int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *sock_len);
int getpeername(int sockfd, struct scokaddr *peername, socklen_t *sock_len);
```
* getsockname 获取fd 本地的地址，server ADDR_ANY、PORT_ANY 绑定的fd，在accept后 可以此函数获取连接的本地地址
* getpeername 获取对端的地址
* 对fork exec的进程，由于内存镜像呗替换，原进程的sock数据结果被替换，用上述两个函数可获取相关的地址信息


sockaddr_storage
================
* 在获取的地址不止长度时，用次结构做为结果存储结构，能存储任意类型的sockaddr类型
* 对fork exec的进程，由于内存镜像呗替换，原进程的sock数据结果被替换，用上述两个函数可获取相关的地址信息
* 对fork exec的进程，由于内存镜像呗替换，原进程的sock数据结果被替换，用上述两个函数可获取相关的地址信息

SIG
===
* 设置SIG_IGN胡烈之
* 设置SIG_DFL使用默认内核处理函数
* **sig_mask** 设置`阻塞`的信号，相应的信号不会递交给进程
* SIG_KILL SIG_STOP无法 被忽略，无法设置回调
```
struct sigacion{
 sa_flags;//要设置的信号
 sa_handler;// 对应的信号处理函数
}
sigaction(int signo, struct sigaction* actions, struct sigaction *outactions)
```
 * 
