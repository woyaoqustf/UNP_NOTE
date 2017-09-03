
Tips
  * TCP (Transmission Control Protocol, 传输控制协议)  
  * bzero , 所有支持socket的OS都支持bzero函数  
  * 任何现实世界的程序，都必须检查函数的返回值是否返回错误  
  * 线程函数，错误码errno都是以返回值得形式返回，即`pthread_开头的函数，都需以返回值判断错误类型`   
  * Unix函数，发生错误，全局errno置为相应的值，错误码都已E开头，ERROR的定义在 <sys/errno.h> 中

## 接口  
* socket  
  * 协议族  
  * 协议类型  
* sockaddr_in(socket_in6)  
  * sin_family(sin6_family)  
  * sin_port(sin6_port) 
  
## server
* socket --> sockfd
* sockaddr_in  
  * family
  * socketaddr_in.sin_addr.s_addr
  * socketaddr_in.sin_port
  * bind: sock_fd, addr
  * accept
  * W/R

  
