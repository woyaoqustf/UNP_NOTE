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
* 在系统终端**EINT,或者ECONNABORT之后，EAGAIN，accept是可以重试的**，libevent就是这么处理的
* ECONNABORT，这个错是在三次握手之后，连接已经进入完成队列，而在还没有accept返回之前，`客户端发送了RST要断开连接`，这个时候accept会返回错误（有些实OS实现是会直接将连接从完成队列删除），errno为ECONNABORT，此时accept是非致命错误，是可以重试的,

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

中断的系统调用
=============
* 在执行accept，read等函数时线程被阻塞，当收到信号时，正常应该处理中断，so线程需要从慢系统调用中苏醒，这是系统调用会返回错误，errno设置为EINT

SIG
===
### 处理EINT,处理多子进程的 SIGCHLD,用wait_pid WNOHANG
* 设置SIG_IGN胡烈之
* 设置SIG_DFL使用默认内核处理函数
* **sig_mask** 设置`阻塞`的信号，相应的信号不会递交给进程
* SIG_KILL SIG_STOP无法 被忽略，无法设置回调
* SA_RESTART有些OS能自动重启中断的系统调用，有的不能需要处理EINT errno,为可移植性应该处理EINT
* SIG_ALARM，通常是用作IO超时，应当 主动设置 中断系统调用**TODO???**

```
struct sigaction {
        void     (*sa_handler)(int);
        void     (*sa_sigaction)(int, siginfo_t *, void *);
        sigset_t   sa_mask;
        int        sa_flags;
        void     (*sa_restorer)(void);
    };
sigaction(int signo, struct sigaction* actions, struct sigaction *oldactions)
```
 * 信号不排队，正在执行的信号被阻塞
 
 被中断的系统调用
 ===============
 
 wait,wait_pid
 =============
 ```
 int wait(int *statloc);
 int wait_pid(pid_t pid, int *statloc, int options);
 ```
 * 在有多个子进程时，使用wait_pid, pid设置为-1（等待第一个结束的子进程），option设置 WNOHANG(不阻塞进程)
 * **没有任何子进程时 返回-1**
 * 没有状态变化的子进程返回0
 * 父进程应对在SIGCHLD的信号处理函数中调用wait_pid,且有多少child可以wait就掉多少次
 ```
 void sig_child(int signo) {
    int stat;
    while(wait_pid(-1, &stat, WNOHANG) > 0)do_someting();
    return;
 }
 ```
 
 服务进程终止
 ==========
 * 在服务主机进程终止，收到数据是，返回RST，read时返回EOF 返回0 （server TIME_WAIT2, client CLOSE_WAIT）
 * 客户端收到RST的fd，再次写操作，内核发送SIGPIPE，写操作但会PIPE错误码
 * SIGPIPE默认终止进程，客户程序应当处理改信号(可能 client多次写一个server 关闭的放fd，在次之前RST数据没有被读到)
 
 服务器异常的几种情况
 ==================
 * 服务器进程结束，主机&网络都可达，但是监听该端口的进程退出了，这是服务器会返回RST, 客户端对已收到RST的fd写时发送SIGPIPE,返回PIPE错误
 * 服务器崩溃，服务器的网络端口，机器关闭，或者路由器ICPM返回不可达，客户端会不断重试，到一定时间客户进程返回错误（what错）， 此时阻塞的read的函数  返回 ETIMEOUT OR (EHOSTUNREACH , ENETUNREACH (ICPM返回错误))
 * 服务器重启，服务器重启后所有连接信息都丢失，在接受到没连接的数据是返回RST
 
 RST
 ==
 * 服务端，没有端口监听之
 * 接受到没有建立的数据，如服务端重启
 
