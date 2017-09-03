
Tips
  * TCP (Transmission Control Protocol, 传输控制协议)  
  * bzero , 所有支持socket的OS都支持bzero函数  
  * 任何现实世界的程序，都必须检查函数的返回值是否返回错误
  * 线程函数，错误码errono都是以返回值得形式返回，即pthread_开头的函数，都需以返回值判断错误类型

## 接口  
* socket  
  * 协议族  
  * 协议类型  
* sockaddr_in  
  * family  
  * port  
  
