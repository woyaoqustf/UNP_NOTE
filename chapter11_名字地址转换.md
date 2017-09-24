

gethostbyname
=============
查找ipv4的地址
```
struct hostent* gethostbybane(const char *name);// 成功返回 指针，失败NULL,h_errno置为相应值

struct hostent {
  char *h_name; //正式规范名称
  char **h_aliases;//别名列表，字符串，最后一个名称后为NULL
  int h_addrtype;// AF_INET
  int h_length;// 地址长度4
  char **h_addr_list;// iplist，最后一个IP后为NULL
};

错误码，可用hstrerror 函数转换为字符串

```

