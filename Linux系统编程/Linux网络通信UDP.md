## UDP通信

> UDP是一个面向无连接的(不需要connect操作)，不安全的，报式传输层协议，UDP的通信过程也是默认阻塞的
>
> UDP首部没有关于数据顺序的信息，若数据丢失，就全部丢失，不存在一半的情况，发送端不知道数据是否非正确接受，也不会重发数据。
>
> 在UDP通信过程中，哪一段是接受数据的角色，那么这个接收端就必须绑定一个固定的端口

![UDP通信流程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvmibuwoQgfcoib5NIC1Lr4VBxpddJQRVWicmLkibthzMib0vcXLricRIQmTl8OCvCl1icKoA9sK2JDxI1fQ/640?wx_fmt=jpeg "UDP通信流程")

### UDP发送端

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
int main(int argc, char *argv[])
{
	int sockfd;
	struct sockaddr_in server_addr; //用于存储服务器信息的结构体
	char buf[SIZE];
	int i;

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket()
	sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	//初始化服务器地址信息
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT_ID);
	server_addr.sin_addr.s_addr = inet_addr(argv[1]);

	for(i = 0; i < 10; i++)
	{
		sprintf(buf, "%d\n", i);
		// 发送数据
        memset(buf, 0, sizeof(buf));
		sendto(sockfd, buf, SIZE, 0, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));
		// 0： flags,套接字属性，一般使用默认属性
		// dest_addr:接受数据的一段对应的地址信息IP和端口
		// addrlen 参数dest_addr指向的内存大小
		// 函数调用成功返回实际发送的字节数，调用失败返回-1
		
		printf("Client sends to server %s: %s\n", argv[1], buf);
		sleep(1);
	}

	close(sockfd);

	return 0;
}
```

### UDP接收端

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <string.h>
//UDP通信
#define PORT_ID	8800
#define SIZE	100

int main(void)
{
	int sockfd;
	struct sockaddr_in my_addr, client_addr;
	int addr_len;
	char buf[SIZE];

	//1.socket() 绑定套接字 SOCK_DGRAM设置为UDP
	//通过指定SOCK_DGRAM来指定报式传输协议的套接字
	// 第三个参数0表示使用报式协议中的UDP
	sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	if(sockfd == -1)
    {
        perror("socket");
        exit(0);
    }

	//2.bind() 将套接字绑定端口与IP，
	my_addr.sin_family = AF_INET;
	my_addr.sin_port = htons(PORT_ID);
	my_addr.sin_addr.s_addr = INADDR_ANY;
	int ret = bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr));
  	if(ret == -1)
    {
        perror("bind");
        exit(0);
    }
	//绑定后可直接接受数据UDP传输
	addr_len = sizeof(struct sockaddr);
	while(1)
	{
		printf("Server is waiting for client to connect:\n");
		//4.recv() 接受数据
		memset(buf, 0, sizeof(buf));
		int rlen = recvfrom(sockfd, buf, SIZE, 0, (struct sockaddr *)&client_addr, &addr_len);
		//接受数据，如果没有数据，该函数阻塞
		//sockfd:基于UDP通信的文件描述符
		//buf指针指向地址用来存储接受的数据
		//len:buf指针指向的内存容量
		//flags：设置套接字属性，一般使用默认属性，指定为0
		//src_addr：发送数据端的地址信息，IP和端口都存储在这里边,大端存储
		//addrlen：传入src_addr参数指向的内存大小，传出的也是
		printf("客户端的IP地址: %s, 端口: %d\n",
		inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr, ipbuf, sizeof(ipbuf)),
		ntohs(cliaddr.sin_port));
		printf("Server receive from client: %s\n", buf);
	}

	close(sockfd);

	return 0;
}
```

## UDP广播特性



## UDP组播