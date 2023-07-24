## UDP通信

> `UDP`是一个面向`无连接`的(不需要connect操作)，不安全的，报式传输层协议，`UDP`的通信过程也是`默认阻塞`的
>
> `UDP`首部没有关于数据顺序的信息，若数据丢失，就`全部丢失`，不存在一半的情况，发送端不知道数据是否非正确接受，也不会重发数据。
>
> 在`UDP通信`过程中，哪一段是接受数据的角色，那么这个接收端就`必须绑定`一个固定的`端口`

![UDP通信流程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvmibuwoQgfcoib5NIC1Lr4VBxpddJQRVWicmLkibthzMib0vcXLricRIQmTl8OCvCl1icKoA9sK2JDxI1fQ/640?wx_fmt=jpeg "UDP通信流程")

### `UDP`发送端

```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>


#define PORT_ID	18899
#define SIZE	100
//客户端发送数据
//./client IP  : ./client 192.168.217.1
int main(int argc, char *argv[])
{
	int sockfd;
	struct sockaddr_in server_addr; //用于存储服务器信息的结构体
	char buf[SIZE] = "Hello UDP";
	int i;

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket()
	sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	//初始化服务器地址信息
	server_addr.sin_family = AF_INET; //绑定的IPV4
	server_addr.sin_port = htons(PORT_ID); //将端口号转换为大端
	server_addr.sin_addr.s_addr = inet_addr(argv[1]); //获取用户传入的地址

	for(i = 0; i < 10; i++)
	{
        // 发送数据
        memset(buf, 0, sizeof(buf));
		sprintf(buf, "%d\n", i);
		sendto(sockfd, buf, SIZE, 0, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));
		// 0： flags,套接字属性，一般使用默认属性
		// dest_addr:接受数据的一段对应的地址信息IP和端口
		// addrlen 参数dest_addr指向的内存大小
		// 函数调用成功返回实际发送的字节数，调用失败返回-1
		
		printf(" UDP向端口8899发送数据 %s: %s\n", argv[1], buf);
		sleep(1);
	}

	close(sockfd);

	return 0;
}
```

### `UDP`接收端

```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>
//UDP通信
#define PORT_ID 18899
#define SIZE 100

//服务器接受数据
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
    char ipbuf[64]; 
    //一直接收
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
        inet_ntop(AF_INET, &client_addr.sin_addr.s_addr,ipbuf, sizeof(ipbuf)),
        ntohs(client_addr.sin_port));
        printf("接收到客户端发送的数据: %s\n", buf);
    }
    close(sockfd);
    return 0;
}
```

![UDP运行](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVekbc16acLM762MRyicU0jc9t558XEkOcw9JuCmOUg6Aibbc8p4WgcG9n7g/640?wx_fmt=png "UDP运行")

## `UDP`广播特性

> 通过广播可以向`子网中多台`计算机发送消息，并且子网中所有的计算机都可以`接受发送方`发送的消息
>
> 发送端可以将消息同时发送到`局域网`的多台主机上,数据发送必须要把数据发送到广播低地址上，广播只能在`局域网使用`。只要接收端在广播内，就`无法拒绝发送端的信息`。广播的`开销很小`，是局域网快速传播消息的方式。

### 设置广播属性

> 广播属性`默认是关闭的`

```c
int setsockopt(int sockfd, int level, int optname, 	const void *optval, socklen_t optlen);
//sockfd ：UDP通信文件描述符
// level 套接字级别，需要设置为SOL_SOCKET
// optname 选项名，此处设置UDP广播属性SO_BROADCAST
// optval 设置广播属性，该指针实际指向一个int类型内存
// 0 关闭广播属性
// 1 打开广播属性
//optlen ： optval 指针指向的内存大小即sizeof(int)

```

### 通信流程

> 发送端在固定端口发送数据，接收端绑定到端口，只接受广播信息、在数据接收端要绑定固定的端口，发送端可以随机端口

#### 发送端 广播端

* 创建套接字
* 设置广播属性
* `sendto` 发送数据到`端口上`
* 关闭套接字(文件描述符)

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
//发送端server
int main()
{
    // 1. 创建通信的套接字
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 设置广播属性
    int opt  = 1; //0关闭广播属性，1打开广播属性
    setsockopt(fd, SOL_SOCKET, SO_BROADCAST, &opt, sizeof(opt));

    char buf[1024];
    struct sockaddr_in cliaddr;
    int len = sizeof(cliaddr);
    cliaddr.sin_family = AF_INET; //IPV4
    cliaddr.sin_port = htons(9999); // 接收端需要绑定9999端口
    // 只要主机在217网段, 并且绑定了9999端口, 这个接收端就能收到广播消息
    inet_pton(AF_INET, "192.168.217.255", &cliaddr.sin_addr.s_addr); //此处设置成自身Linux所处的网段
    // 3. 通信
    int num = 0;
    while(1)
    {
        sprintf(buf, "hello, client...%d\n", num++);
        // 数据广播
        sendto(fd, buf, strlen(buf)+1, 0, (struct sockaddr*)&cliaddr, len);
        printf("发送的广播的数据: %s\n", buf);
        sleep(1);
    }

    close(fd);
    return 0;
}
```



#### 接收端

* 创建通信的套接字
* 绑定`固定`的端口和`本地IP地址`
* `recvfrom` 接收广播消息(接受的数据从接收端打开的时刻开始`对应数据`)
* 关闭套接字

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
//多个接收端都可以接受到数据
int main()
{
    // 1. 创建通信的套接字
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }
    //设置端口复用，以便多个客户端连接此端口进行接受数据
    int opt = 1; //1 开启端口复用 0关闭
    setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 2. 通信的套接字和本地的IP与端口绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);    // 大端 端口要与服务器相同
    // 发送端时本机的217网段，客户接收端任意IP皆可
    addr.sin_addr.s_addr = INADDR_ANY;  // 0.0.0.0 接收端的IP任意 
    int ret = bind(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("bind");
        exit(0);
    }

    char buf[1024];
    // 3. 通信
    while(1)
    {
        //客户端接受广播
        // 接收广播消息
        memset(buf, 0, sizeof(buf));
        // 阻塞等待数据达到
        recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("接收到的广播消息: %s\n", buf);
    }

    close(fd);

    return 0;
}
```

![广播](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVekOPt9V8JpXf4TYCJlv7qOdtvnSQO0eOpDbQSn7yGiaico5RciaicTicbvICw/640?wx_fmt=png "广播")

## `UDP`组播

> 组播是主机间`一对多`通讯模型，允许一个或多个组播源发送同一报文到多个接受者的技术。组播源将一份报文发送到特定的组播地址，一个`组播地址表示一个群`组，需要接受组播报文接收者都`加入`这个群组。`群聊`
>
> * 组播也可用于`广域网`
> * 组播可以控制发送端的数据被哪些接受
> * 组播使用组播地址
> * `组播地址`不属于服务器或个人，类似`QQ群号`。任何组播源往`组播IP`发送消息(组播数据)，这个组播接受者都会接收到此消息
> * `setsockopt`函数开启默认关闭的组播属性
> * 通过端口复用可以多个发送端，多个接收端，接收端可以同步接受所有发送端的数据

### `IPV4`组播地址

| `IP 地址` | 说明 |
| ------- | ---- |
|`224.0.0.0~224.0.0.255`| `局部`链接多播地址：是为路由协议和其它用途保留的地址 |
|只能用于`局域网`中|路由器是`不会转发的`地址 224.0.0.0 不能用，是保留地址|
|`224.0.1.0~224.0.1.255`	|为`用户可用的组播地址`（临时组地址），可以用于 `Internet` 上的|
|`224.0.2.0~238.255.255.255`	|用户可用的`组播地址`（临时组地址），全网范围内有效|
|`239.0.0.0~239.255.255.255`	|为`本地管理组播地址`，仅在`特定的本地范围内`有效|

### 发送端

```c
// 发送端
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建通信的套接字 SOCK_DGRAM，0使用UDP协议
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 设置组播属性
    struct in_addr opt;
    // 将组播地址初始化到这个结构体成员中即可
    inet_pton(AF_INET, "239.0.1.10", &opt.s_addr); // IP地址转换，将中间IPV4地址转为网络字节序到最后的输出参数
    //设置组播属性
    setsockopt(fd, IPPROTO_IP, IP_MULTICAST_IF, &opt, sizeof(opt));
    // fd 套接字， IPPROTO_IP 为level参数，套接字界别
    //IP_MULTICAST_IF 套接字选项名
    // opt 为指向struct in_addr{}的结构体地址，用于存储组播地址，大端存储
    char buf[1024];
    //设置绑定信息
    struct sockaddr_in cliaddr;
    int len = sizeof(cliaddr);
    cliaddr.sin_family = AF_INET; //IPV4
    cliaddr.sin_port = htons(9999); // 接收端需要绑定9999端口
    // 发送组播消息, 需要使用组播地址, 和设置组播属性使用的组播地址一致就可以
    inet_pton(AF_INET, "239.0.1.10", &cliaddr.sin_addr.s_addr); //IP地址转换
    // 3. 通信
    int num = 0;
    while(1)
    {
        //设置发送的数据
        sprintf(buf, "hello, client...%d\n", num++);
        // 数据广播 
        // 通过sendto函数发送数据到组播地址，并在程序中数据发送到接收端9999端口，接收端必须绑定此端口才能收到组播
        sendto(fd, buf, strlen(buf)+1, 0, (struct sockaddr*)&cliaddr, len);
        printf("发送的组播的数据: %s\n", buf);
        sleep(1);
    }

    close(fd);

    return 0;
}
```



### 接收端

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <net/if.h>
//组播数据接收方，需要加入组播方可
int main()
{
    // 1. 创建通信的套接字
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }
        //设置端口复用，以便多个客户端连接此端口进行接受数据
    int opts = 1; //1 开启端口复用 0关闭
    setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &opts, sizeof(opts));
    // 2. 通信的套接字和本地的IP与端口绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);    // 大端
    addr.sin_addr.s_addr = INADDR_ANY;  // 0.0.0.0
    //接收端需要绑定固定端口
    int ret = bind(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("bind");
        exit(0);
    }

    // 3. 加入到多播组，入群之后就可以接受组播信息
    struct ip_mreqn opt; // struct ip_mreqn用于接入组播机构提
    // 要加入到哪个多播组, 通过组播地址来区分
    inet_pton(AF_INET, "239.0.1.10", &opt.imr_multiaddr.s_addr); //转换为大端网络字节序
    opt.imr_address.s_addr = INADDR_ANY;   //接收任何源地址组播地址发送的组播信息
    //if_nametoindex函数将网络接口名"ens33"转换为对应的索引值，
    //并将结果存储在opt.imr_ifindex中。
    //if_nametoindex函数用于根据网络接口名获取对应的索引值。
    opt.imr_ifindex = if_nametoindex("ens33");
    //将当前主机加入到指定的多播组中IP_ADD_MEMBERSHIP表示加入多播组的
    //&opt为指向包含加入多播组参数的结构体的指针，sizeof(opt)表示结构体的大小。
    setsockopt(fd, IPPROTO_IP, IP_ADD_MEMBERSHIP, &opt, sizeof(opt));

    char buf[1024];
    // 3. 通信
    while(1)
    {
        // 接收广播消息
        memset(buf, 0, sizeof(buf));
        // 阻塞等待数据达到
        recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("接收到的组播消息: %s\n", buf);
    }

    close(fd);

    return 0;
}
```

![组播](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVek640yDZpFbnib7icUgNete0knXBYQiaI9jPoRhicz7OfOY0OBTdABqRtVUQ/640?wx_fmt=png "组播")
