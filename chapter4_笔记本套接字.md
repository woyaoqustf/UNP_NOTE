socket 函数
===========
```
int socket(int family, int type, int protocol); // 成功返回fd，失败 -1
```
* protocol 写0 ，系统根据family type 设置默认协议
* SOCKET_STREAM * AF_INET AF_INET = TCP
* SOCKET_DGRM *  AF_INET AF_INET = UDP
* SOCKET_RAW *  AF_INET AF_INET = IPV4,IPV6
