
数据就绪
========
- 读 
  1. 读缓冲字节数**大于***读低水位*（低水位为了给客户控制**至少有多少字节可读**时候返回，减少不完整数据消耗客户进程陷入内核处理次数）  
    
- 异常， 返回-1
- 半读关闭，读socket接受到一个FIN，另一端标示关闭它不再写数据了，此时read 返回0 
- 监听套接字，已完成连接数不为0

写
- 写，写缓冲`可用字节数` **大于于** `写低水位` （水位保证内核写缓冲**一次最少可写字节数**）
- 写半关闭，自己主动发送了FIN 表示我不会再写数据  
- connect 函数非阻塞的fd连接建立，通知客户进程可写

半关闭
=====
- 半关闭状态，我的理解就是发送或者接受了FIN  
- 从主动方讲，发送FIN表示为我告诉对方我不在写了，表示写半关闭（至于需不需要接受ACK,我理解不需要，这相当标示要关闭不在写，不管对方有没有收到）  
- 从被动方将，接受到FIN表示我知道对方不会再有数据发送过来了，所以读已经关闭了，但我自己还想可以继续写

shutDown
========
```
int shutdown(int sockfd, int howto);//成功返回0， 失败反回-1
```
- 正常close，引用减1，为0时关闭套接字，读写都关闭
- shutDown 可以忽略引用计数，立即发送FIN 
- SHUT_RD 关闭读一半，所有接收到的字节返回ACK，数据被丢弃，读操作会出错
- SHUT_WR 进入半关闭状态， 所有缓冲区的字节都发送完毕，后接正常的种种序列（FIN?) ,对套接字写报错
- SHUT_RDWR 相当于先调SHUT_RD, SHUT_WR

poll
===
```
int poll(stuct pollfd *fdarray, unsigned long nfds, int timeout);// 返回就绪个数，超时返回0，失败范湖-1

struct pollfd{
  int fd; // fd
  int events; // 关心的events
  int revents; // 出现的事件
};
```
### 事件类型
poll的事件总结为三类，normal， 带外， 异常
按照读写分又可分为 读，写 ，异常
#### normal：
POLLIN（定义为POLLRDNORMAL|POLLRDBAND  
POLLRDNORMAL  
POLLOUT  
POLLWRNORMAL  

#### 带外
POLLRDBAND, POLLWRBAND, POLLPIR(带外优先级读数据)  

#### 异常
POLLERR  // 错误
POLLHUP //发生挂起？
POLLNVAL //fd 未打开

正常不要随便使用带外



