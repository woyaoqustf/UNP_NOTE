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
 * sockaddr 前两位和sockaddr_in_ 的属性和长度是一致的，在强转后可以更具 sa_family 判断family类型然后转换成对应family的类型
