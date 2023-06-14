# 套接字通信
> 设计者开发的`接口`，以便应用程序调用该通信接口进行`通信`
![网络层次](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvmibuwoQgfcoib5NIC1Lr4VBdg4IDdiaZCFfxhmZDRwLefN5W0KhhNMFlHgwWH5hLGs4MSdWp2hB7TA/640?wx_fmt=jpeg "网络层次")
## 字节序
> 字节的顺机序，大于一个字节类型的数据在内存中的存放顺序。

* `Little-Endian` 小端模式:数据的`低`位字节存在`低位`，PC机默认低位
* `Big-Endian`：大端模式(网络字节序) 数据的`低位`字节存储到`高地`址位，套接字`通信`中操作的数据都是`大端存储`，包括:`接受/发送数据，IP地址，端口`

### 字节转换函数
```c
#include <arpa/inet.h>
//将一个短整型short从主机字节序->网络字节序
uint16_t htons(uint16_t hostshort)
//整型
uint32_t htonl(uint32_t hostlong)
//将一个短整型short从网络字节序 -> 主机字节序
uint16_t ntohs(uint16_t netshort)
// 将一个整型从网络字节序 -> 主机字节序
```
### `IP`地址转换

```c
//将主机小端字节序转换为网络字节序打断
int inet_pton(int af,const char *src,void *dst);
//af:地址族(IP地址的家族包括IPV4,IPV6)
// * AF_INET : IPV4
// * AF_INET6 :IPV6
// src 传入参数  ip地址
// dst 传出参数
// 函数返回值: 成功返回0，失败返回-1
//将大端的整形数，转换为小端的点分十进制的IP地址
#include <arpa/inet.h>
const char *inet_ntop(int af,const void *src,char *dst,socklen_t size);
//上同
// size : 传出参数dst最多存储多少个字节
// 成功 返回取出转换得到的IP字符串，失败NULL
```
### `sockaddr `数据结构

```c
struct sockaddr {
	sa_family_t sa_family;       // 地址族协议, ipv4
	char        sa_data[14];     // 端口(2字节) + IP地址(4字节) + 填充(8字节)
}
// sizeof(struct sockaddr) == sizeof(struct sockaddr_in)
struct sockaddr_in
{
    sa_family_t sin_family;		/* 地址族协议: AF_INET */
    in_port_t sin_port;         /* 端口, 2字节-> 大端  */
    struct in_addr sin_addr;    /* IP地址, 4字节 -> 大端  */
    /* 填充 8字节 */
    unsigned char sin_zero[sizeof (struct sockaddr) - sizeof(sin_family) -
               sizeof (in_port_t) - sizeof (struct in_addr)];
};  
```
### 套接字函数
#### `socket` 套接字

```c
#include <arpa/inet.h>
//创建一个套接字
int socket(int domain,int type,int protocol);
// Domains地址族协议，上同
//type:
//	SOCK_STREAM:流式传输协议，默认使用TCP
//	SOCK_DGRAM:报文传输协议，默认使用UDP
//  protecol:一般为0

// 返回值；成功返回套接字通信的文件描述符，失败返回 -1
```
#### `bind` 绑定

```c
// 将文件描述符和本地的IP与端口进行绑定   
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// sockfd : 监听的文件描述符，socket的返回值
// addr ：传入参数IP和端口数据结构
// addrlen:参数addr指向的内存大小sizeof(struct sockaddr)
// 返回值：成功返回0 失败返回-1
```
#### `listen `监听

```c
// 给监听的套接字设置监听
int listen(int sockfd, int backlog);
// fd 文件描述符，socket的返回值
//backlog： 同时能处理的最大连接请求，最大值128
// 成功返回0 失败返回-1
```
#### `accept` 接受

> 阻塞函数，在没有新的客户端连接请求的时候，该函数阻塞，返回一个文件描述符，基于此文件描述符和客户端通信

```c
// 等待并接受客户端的连接请求, 建立新的连接, 会得到一个新的文件描述符(通信的)		
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
// sockfd : 监听的文件描述符
// addr 传出参数IP 和端口
// addrlen :addr指向内存大小
// 失败返回-1
```
#### 接受数据
> 如果连接没有断开，接收端接受不到数据，接受数据的函数阻塞等待。数据到达后解除阻塞，开始接受数据，当发送端断开连接，接收端无法接收到任何数据，函数直接返回0

```c
// 接收数据
ssize_t read(int sockfd, void *buf, size_t size);
ssize_t recv(int sockfd, void *buf, size_t size, int flags);
//fd accept的返回文件描述符
// buf 有效内存，存储接受的数据
// size 参数buf指向的内存容量
//flags 一般为0
// 返回值:大于0 实际接受的字节数 等于0 对方断开了连接，-1 接受数据失败
```
#### 发送数据
```c
// 发送数据的函数
ssize_t write(int fd, const void *buf, size_t len);
ssize_t send(int fd, const void *buf, size_t len, int flags);
//buf传入的字符串，
//返回值：大于0，实际发送的字节数，和参数len是相等
// -1 发送数据失败了
```
#### `connect`

```c
// 成功连接服务器之后, 客户端会自动随机绑定一个端口
// 服务器端调用accept()的函数, 客户端调用connect连接
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// sockfd文件描述符，调用socket得到
// 连接成功0 失败-1
```

## `TCP Linux`通信

> TCP协议是一个安全的，面向连接的，流式传输协议。在客户端调用connect函数，完成三次握手，close函数完成四次挥手。

### `三次握手`

![三次握手](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsj1OS0lAvlnpXy0v6sC4KPKyhWvGKXCqGytxz0QhdibFc1n5YX7GXcgk9XxpeXXdRc68ewWplGvUA/640?wx_fmt=jpeg "三次握手")

### `四次挥手`

![四次挥手](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsj1OS0lAvlnpXy0v6sC4KPN3LLdYSSCtEAWBNR4ibkdf4O9UxGydeLQJuEI2NRoVTUKRvfgic31ic6g/640?wx_fmt=jpeg "四次挥手")

### `socket`通信

![TCP通信流程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvmibuwoQgfcoib5NIC1Lr4VBZBH5hh98BHyHSy3E6KFI4ZzePtdqVpvW8uS7yIyusC7W62cKUJOWsA/640?wx_fmt=jpeg "TCP通信流程")

### `client.c`

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>

#define PORT_ID	8800
#define SIZE	100

//./client IP  : ./client 192.168.0.10
int main(int argc, char *argv[]) //在运行时需要输入参数，即IP地址
{
	int sockfd;
	struct sockaddr_in server_addr;
	char buf[SIZE];

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket() 建立套接字
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.connect()
	server_addr.sin_family = AF_INET; 
	server_addr.sin_port = htons(PORT_ID); //将端口转化为大端网络通信模式
	//inet_addr 将一个十进制的数转化为二进制数用于IPV4的IP转换
	server_addr.sin_addr.s_addr = inet_addr(argv[1]); 
	//间接IP和端口
	connect(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));

    //接受数据存入buf缓冲当中
    int re = recv(sockfd, buf, SIZE, 0);
    if(re == 0)
    {
        printf("服务器断开了连接\n");
    }
    else if(re < -1)
    {
        printf("接受数据失败\n");
    }
    printf("Client receive from server: %s\n", buf);
    sleep(1);
	


	close(sockfd);

	return 0;
}
```

### `server.c`

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>

//设置的端口号
#define PORT_ID	8800
//设置发送文件的大小
#define SIZE	100

int main(void)
{
	int sockfd, client_sockfd;
	struct sockaddr_in my_addr, client_addr; //定义结构体用户存储IP和端口号
	int addr_len;
	char welcome[SIZE] = "Welcome to connect to the sever!";

	//1.socket()创建监听的套接字 AF_INET IPV4 SOCK_STREAM TCP通信
	sockfd = socket(AF_INET, SOCK_STREAM, 0);
	if(sockfd == -1)
	{
		printf("套接字创建失败");
	}
	//2.bind()
	my_addr.sin_family = AF_INET;
	my_addr.sin_port = htons(PORT_ID); //将端口号整型转换为网络字节序打断存储
	my_addr.sin_addr.s_addr = INADDR_ANY;  //宏值，绑定任意端口号
	//绑定套接字与端口IP
	int b = bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr));
	if(b == -1)
	{
		printf("套接字与IP绑定失败");
	}
	//3.listen() 建立连接，给佳宁的套接字设置最大连接请求
	int l = listen(sockfd, 10); 
	if(l == -1)
	{
		printf("套接字设置监听失败");
	}
	addr_len = sizeof(struct sockaddr);

	while(1)
	{
		printf("Server is waiting for client to connect:\n");
		//4.accept() 阻塞函数，等待客户端连接
		client_sockfd = accept(sockfd, (struct sockaddr *)&client_addr, &addr_len);
		
		printf("Client IP address = %s\n", inet_ntoa(client_addr.sin_addr));
		//5.send()
		int sed = send(client_sockfd, welcome, SIZE, 0);
		if(sed > 0)
		{
			printf("发送了%d字节的数据",sed);
		}
		else if(sed == 0)
		{
			printf("客户端断开了连接");
		}
		else if(sed == -1)
		{
			printf("发送数据失败");
		}
		printf("Disconnect the client request.\n");
		//6.close()
		close(client_sockfd);
	}
	//关闭套接字
	close(sockfd);

	return 0;
}
```

![套接字通信](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsj1OS0lAvlnpXy0v6sC4KPjLj5pUog8sEd3pBBISrsGegareAz78rlMXDQ3LuZ4mWic7uGy6T0Tyg/640?wx_fmt=png "套接字通信")

### `TCP流量控制`

> 流量控制可以根据接收端的实际接受能力，接收端主机向发送端主机通知自己可以接受数据的大小，于是发送端发送不会超过该大小的数据，
>
> TCP首部，专门设置一个字段来通知窗口大小，当接收端的缓冲区面临数据溢出时，窗口的大小值也随之发生改变，设置一个更小的值通知发送端，从而控制数据的发送量，从而达到流量控制的目录--滑动窗口

### `TCP半关闭`

> TCP连接只有一方发送了FIN请求连接，处于半关闭状态，数据仍可以单向通信。套接字本身时双向通信

* 服务器调用了close函数，不能发送数据，只能接受数据
* 客户端没有调用close函数，可以发送数据，但是不能接受数据

```c
#include <sys/socket.h>
//可以由选择的关闭读/写，close只能关闭写操作
int shutdown(int sockfd,int how);
//sockfd要操作的文件描述符
// how
//SHUT_RD: 关闭文件描述符对应的读操作
//SHUT_WR: 关闭文件描述符对应的写操作
//SHUT_RDWR: 关闭文件描述符对应的读写操作
//返回值:函数调用成功返回0，失败返回-1
```

### `端口复用`

> 服务器进程主动断开连接，最后状态编程TIME_WAIT状态，这个进程会等待2msl(1分钟左右)才会退出，如该进程不退出，其绑定的端口就不会释放。

```c
//设置端口复用多功能函数
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
//sockfd 用于监听的文件描述符
// level 设置端口复用需要使用SOL_SOCKET宏
// optname 设置属性皆可端口复用
// optval 设置去除端口复用属性还是端口复用属性，实际应该使用int型变量0不设置，1设置
// optlen: optval指针指向的内存大小sizeof(int)
// 在绑定之前设置端口复用
    // 创建监听的套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if(lfd == -1)
    {
        perror("socket error");
        exit(1);
    }

    // 绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(9999);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 本地多有的ＩＰ
    // 127.0.0.1
    // inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr.s_addr);
    
    // 设置端口复用
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 绑定端口
    int ret = bind(lfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
```

## TCP数据`粘包`处理

> 在发送数据块之前，在数据块最前边添加一个固定大小的数据头。数据头存储当前的总字节数，接收到先接受数据头，然后根据数据头结构对应字节大小的数据块

### 发送端实现

* 发送N数据长度申请`N+4`块大小内存
* 将待发送数据总长度写入前四个字节中(需要转为大端`网络字节序`)
* 待发送的数据拷贝到包头后边的地址空间，将完整数据包发送出去
* 释放申请的`堆空间`

```c
// 发送指定的字节数
// fd:套接字的文件描述符
// msg:待发送的原始数据
// size : 待发送的原始数据总字节数
int writen(int fd, const char* msg, int size)
{
    //buf 数据缓存区
    const char* buf = msg;
    // 原始数据大小
    int count = size;
    while (count > 0)
    {
        // len返回已经发送的数据长度
        int len = send(fd, buf, count, 0);
        if (len == -1)
        {
            close(fd); 
            return -1; //发送失败
        }
        else if (len == 0)
        {
            continue;
        }
        // 缓冲区地址后移
        buf += len;
        //待发送长度减去已经发送的
        count -= len;
    }
    return size;
}
// 发送带有包头的数据包
int sendMsg(int cfd, char* msg, int len)
{
   if(msg == NULL || len <= 0 || cfd <=0)
   {   //如果发送缓存区，或长度，或套接字为空失败
       
       return -1;
   }
   // 申请内存空间: 数据长度 + 包头4字节(存储数据长度)
   char* data = (char*)malloc(len+4);
   int bigLen = htonl(len); //将数据包长度转换为网络字节序
   memcpy(data, &bigLen, 4); // 把bigLen写入data当中
   memcpy(data+4, msg, len); //把msg数据写入，data+4指针后移4位
   // 发送数据
   int ret = writen(cfd, data, len+4); //将数据发送过去
   // 释放内存
   free(data);
   return ret; //返回发送的字节数
}
```

### 接收端实现

* 将接受的`4字节`数据，从网络字节序转换为主机字节序
* 根据长度申请堆内存，用于存储`待接收的数据`
* 根据得到的数据长度固定数目保存到堆内存中
* 处理`接受的数据`
* 释放存储数据的`堆内存`

```c
// 套接字，缓存区，大小
int readn(int fd, char* buf, int size)
{
    char* pt = buf;
    int count = size;
    //直到读完所有数据
    while (count > 0)
    {
        //接受数据
        int len = recv(fd, pt, count, 0);
        if (len == -1)
        {
            return -1;
        }
        else if (len == 0)
        {
            return size - count;
        }
        pt += len;
        count -= len;
    }
    return size; //返回接受的字节数
}
// 接受带数据头的数据包
int recvMsg(int cfd, char** msg)
{
    // 接收数据
    // 1. 读数据头
    int len = 0;
    // 先读4个字节，读取其数据长度
    readn(cfd, (char*)&len, 4);
    len = ntohl(len); //大端转换为小端
    printf("数据块大小: %d\n", len);

    // 根据读出的长度分配内存，+1 -> 这个字节存储\0
    char *buf = (char*)malloc(len+1);
    //根据指定大小读取数据
    int ret = readn(cfd, buf, len);
    if(ret != len)
    {
        close(cfd);
        free(buf);
        return -1;
    }
    buf[len] = '\0'; //字符串结尾
    *msg = buf;
    return ret;
}
```
