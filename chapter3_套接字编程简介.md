 socket结构可以在两个方向上传递：进程到内核，内核到进程
 
 ipv4 套接字结构
 ==============
 * 不同family的socket，都已sockaddr_ 作为名字的前缀
 * ipv4 的socket 结构体为 sockaddr_in
 
 ```
 struct in_addr {
   in_addr_t s_addr; //32位 网络字节序
 };
 
 sockaddr_in {
   uint8_t sin_len;
   sa_family_t    sin_family;
   in_port_t      sin_port;
   struct in_addr sin_addr;
   char sin_zero[8];
 };
 ```
 
 * 通用套接字，由于socket接口在 ANSI C之前就设计好，为了兼容各种family的socket 定义了通用套接字 sockaddr
 
 ```
 struct sockaddr {
   uint8_t sa_len;
   sa_family_t sa_family;
   char sa_data[14];
 };
 ```
 * bind,accept,listen等函数在使用socket_in_ 时需要强转成 sockaddr 类型指针
 * sockaddr 前两位和sockaddr_in_ 的属性和长度是一致的，在强转后可以更具 sa_family 判断family类型然后转换成对应family的类型
 
 套接字类型
 =========
 * 相关的套接字类型有 sockaddr_sin, sockaddr_in6, sockaddr_un, sockaddr_dl, sockaddr_storage
 * sockaddr_un,sockaddr_dl 是长度可变的，so在sock的接口函数中，有socket长度的参数，用以处理socket的变长特点

值-结果参数
==========
* 即做参数传值，有做函数的返回
* accept recvfrom， getsockname 这些从内核向进程传递socket结构的函数，接收一个socket_len参数，作为socket的总大小，返回是若是可变长socket，返回边长结构的实际长度

字节序
======
* 判定字节序
```
union {
  short s;
  char c[2];
}un;

un.s=0x0102;
if (un.c[0] == 2 && un.c[1] == 1) 小尾端；
if (un.c[0] == 1 && un.c[1] == 2) 大尾端；
if (un.c[0] == 2 && un.c[1] == 2) 小尾端；
```
* htons, htonl, ntohs,ntohl, h:host, n:net, s:short,l:long


