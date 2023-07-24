# I/O多路复用(转接)

* 处理并发，
* 多进程/多线程并发，`accept`检测`客户端连接`请求和子线程和建立连接的客户端`通信`，都会`发送阻塞`，有新客户端`连接`/有`数据`到达才会`解除阻塞`。
* IO多路转接并发
> 委托`内核`帮助`检测文件描述符`的`状态`(`通信`和`监听`两类)，
> 在通信过程中监听(连接请求信息)和通信(读和写)都会放到`读写缓冲区`中，当`调用accept`就会检测`监听中`是否有连接请求，如果没有则会`一直阻塞`。通信的读写缓冲区也是阻塞的。当只有`一个线程`时，`监听和两个通信`都会处于阻塞状态，相互冲突，
> 基于内核可以同时监听多个文件描述符的读写缓冲区和`是否有剩余空间`，当内核帮我们完成所有`读写缓冲区`，则`不会再进入阻塞`。文件描述符是`线性顺序执行`，若同时则仍需要多线程。
> `select`  跨平台
> `poll`和`epoll`用于`linux`平台
> IO多路复用技术的最大优势就是系统`开销小`，系统`不`必创建`进程/线程`，也`不必`维护这些进程/线程，从而`较少`系统的`开销`。

## select
> `跨平台`通过此函数``委托内核``检测若干个文件描述符状态(`读写缓存区状态`)
> 委托检测的文件描述符被遍历检测完毕之后，已就绪的满足条件的文件描述符会通过`select()`参数传出，根据参数分情况依次处理.
> `fd_set`类型标识文件描述符集合，类型有128个字节，`1024个标志位`

```cpp
sizeof(fd_set) = 128 字节 * 8 = 1024 bit      // int [32]
```

```cpp
#include <sys/select.h>
//不用的微秒一定要初始化
struct timeval {
    time_t      tv_sec;         /* seconds */
    suseconds_t tv_usec;        /* microseconds */
};

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval * timeout);
           
```
* `nfds` : 委托内核检测的`三个集合`中`最大`文件描述符` +1` ,在`Windows`中此参数无效为`-1`
`传入传出参数`
* `readfds`:内核检测此集合文件描述符对应`读缓冲区 `
* `writefds`: 对应`写`缓存区` 不`需要可以指定`NULL`
	*` exceptfds`:检测是否有`异常状态`， `不`需要可以传入`NULL`
	`超时参数`
	*` timeout`:`超时时长，`用来强制解除`select()` 函数`阻塞`
	* NULL ：函数检测不到就绪文件就一直阻塞
	* 等待固定时间:指定时间后强制解除阻塞，函数返回0
	* 不等待:函数不阻塞，参数对应结构体阻塞为0
	函数返回值
* `大于0 `：成功，返回集合中`已就绪`文件描述符总数
* `等于-1`：函数调用失败
* `等于0`: ：超时，没有检测到就绪文件描述符

### `fd_set`类型参数

> 如果`fd_set` 标志为`0不`检测此文件描述符状态，为`1检测`文件描述符，内核在读集合的过程中，若被检测的文件描述符没有数据，修改此文件描述符对应标志位为0，若有数据，标志位不变，仍为1。当`select函数`被解除阻塞之后，被内核修改过的读集合通过参数传出，此时集合中只要标志位`为1`，那么它对应的文件描述符肯定就是`就绪的`。基于此文件描述符和客户端建立新连接或者通信。

```cpp
// 将文件描述符fd从集合set中删除，将fd对应的标志位设置为0
void FD_CLR(int fd,fd_set* set);
//判断文件描述符fd是否在set集合中， 读fd对应的标志位是0还是1
int FD_ISSET(int fd,fd_set *set);
//将文件描述符fd添加到set集合中  ，将fd对应的标志位设置为1
void FD_SET(int fd,fd_set* set);
//将set集合中，所有文件描述符对应的标志位设置为0
void FD_ZERO(fd_Set* set);
```

### 并发处理流程

* 创建监听套接字 `fd=socket`
* `bind`绑定本地`IP`和`端口`
* `listen`设置套接字`监听`
* 创建文件描述符集合`fd_set`，用于存储需要检测`事件`的所有`文件描述符`
	* `FD_ZERO`初始化
	* `FD_SET `将监听的文件描述符放入`检测`的`读集合`中
* 循环调用`select`,对所有文件描述符进行检测
* `selec`t解除阻塞返回，得到内核传出满足条件的文件描述符集合
	* 通过`FD_ISSET`判断集合中标志位是否为`1`
	* 若是监听的文件描述符调用`accept`和客户端`建立连接`，将得到的通信的`文件描述符`，通过`FD_SET`放入到`检测集合中`
	* 若果是`通信`的文件描述符，调用通信函数和客户端通信。如果客户端和服务器断开了连接，使用`FD_CLR`将文件描述符从检测集合中删除，若果没有断开连接，正常通信即可。
* 重复上一步
### 代码测试
#### 服务端
```c
#include <stdio.h>
#include <ctype.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/select.h>
#include <pthread.h>
#include <sys/select.h>

//添加互斥锁
pthread_mutex_t mutex; //针对读，和最大设置互斥



//存储文件描述符 传递给子线程的数据
typedef struct fdinfo
{
    int fd;  //文件描述符可监听/读写
    int *maxfd; //最大文件描述符
    fd_set* rdset;  //文件读地址
}FDInfo;
//子进程函数
void* acceptConn(void* arg)
{
    printf("子线程线程ID:%ld\n",pthread_self());
    FDInfo* info = (FDInfo*)arg;
    // 接受连接请求，此调用不阻塞
    struct sockaddr_in cliaddr;
    int clilen = sizeof(cliaddr);
    int cfd = accept(info->fd,(struct sockaddr*)&cliaddr,&clilen);
    //得到有效的文件描述符
    //通信的文件描述符添加到读集合
    printf("连接的客户端端口为:%d\n",cliaddr.sin_port);
    //在下一轮select检测的时候，就能得到缓冲区的状态
    
    pthread_mutex_lock(&mutex);
    FD_SET(cfd,info->rdset);  //添加文件描述符

    //重置最大文件描述符
    *info->maxfd = cfd > *info->maxfd ? cfd:*info->maxfd;
    pthread_mutex_unlock(&mutex);

    free(info);
    return NULL;
}
//发送与接受数据子线程
void* communication(void* arg)
{
    printf("子线程线程ID:%ld\n",pthread_self());
    FDInfo* info = (FDInfo*)arg;
    //接受数据
    char buf[10] = {0};
    //一次只能接受10个字节，客户端一次发送100个字节
    //一次接受不玩，文件描述符对应缓冲区还有数据
    //下一轮select检测的时候，内核还会标记这个文件描述符再读一吃
    //循环一直次序，直到缓冲区数据被读完为止
    int len = read(info->fd,buf,sizeof(buf));
    if(len ==-1)
    {
        perror("recv error");
        free(info);
        return NULL;

    }

    else if(len == 0 )
    {
        printf("客户端关闭了连接....\n");
        //将检测的文件描述符从读集合中删除
        pthread_mutex_lock(&mutex);
        FD_CLR(info->fd,info->rdset);
        pthread_mutex_unlock(&mutex);
        close(info->fd);
        free(info);
        return NULL;
    }
    else if(len > 0)
    {
        printf("read buf = %s\n",buf);
        for(int i =0;i<len;++i)
        {
            buf[i] = toupper(buf[i]);
        }
        printf("After buf = %s\n",buf);
        //收到了数据
        //发送数据
        int ret = write(info->fd,buf,strlen(buf)+1);
        if(ret == -1)
        {
            perror("send error");   
        }
    }
    else{
        //异常
        perror("read");
    }
    free(info);
    return NULL;
}

int main(void)
{
    //初始化互斥锁
    pthread_mutex_init(&mutex,NULL);

    //1，创建监听
    int lfd = socket(AF_INET,SOCK_STREAM,0);
    //2，绑定端口与IP
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999); //htons将主机字节序转换为网络字节序
    addr.sin_addr.s_addr = INADDR_ANY;
    bind(lfd, (struct sockaddr*)&addr, sizeof(addr));
    // 3，设置监听
    listen(lfd,128);

    // 4
    //将监听的fd状态委托给内核
    int maxfd = lfd;
    //初始化检测的读集合
    fd_set rdset;
    fd_set rdtemp;
    //清零
    FD_ZERO(&rdset);
    //将监听的lfd舍之道检测的读集合中
    FD_SET(lfd,&rdset);
    //通过select委托内核检测读集合中的文件描述符，检测read缓冲区有没有数据
    //若有数据select解除阻塞返回
    // 应该让内核持续检测
    while(1)
    {
        pthread_mutex_lock(&mutex);
        //默认阻塞，rdset中是委托内核检测中的所有文件描述符
        rdtemp = rdset;  //防止select对内核数据的修改
        pthread_mutex_unlock(&mutex);

        int num = select(maxfd+1,&rdtemp,NULL,NULL,NULL);
        //rdset中的数据被内核改写了，只保留了发送变化的文件标志位为1，没变化的改为0
        //只要rdset中fd对应的标志位为1，-> 缓存区有数据
        // 判断有没有新连接，是不是在监听的读集合里
        if(FD_ISSET(lfd,&rdtemp)) //判断文件描述符属于哪一类
        {
            //创建子线程
            pthread_t tid;
            FDInfo* info = (FDInfo*)malloc(sizeof(FDInfo));
            info->fd=lfd;
            info->maxfd = &maxfd;
            info->rdset = &rdset;
            pthread_create(&tid,NULL,acceptConn,info);
            
            pthread_detach(tid); //线程脱离
        }
        //没有新连接，通信
        for(int i =0;i<maxfd+1;++i)
        {
            //判断从监听的文件描述符之后到maxfd这个范围内的文件描述符是否有缓冲区数据
            if(i!=lfd &&FD_ISSET(i,&rdtemp))  //判断是不是通信文件描述符
            {
                //创建子线程接受数据并发送数据
                //创建子线程
                pthread_t tid;
                FDInfo* info = (FDInfo*)malloc(sizeof(FDInfo));
                info->fd = i;
                info->rdset = &rdset; //传递过去的原始读集合
                pthread_create(&tid,NULL,communication,info);
                
                pthread_detach(tid); //线程脱离 
            }

        }
    }
    //销毁互斥锁
    pthread_mutex_destroy(&mutex);
    close(lfd);
    return 0;
}
```
#### 客户端
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建用于通信的套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 连接服务器
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;     // ipv4
    addr.sin_port = htons(9999);   // 服务器监听的端口, 字节序应该是网络字节序
    inet_pton(AF_INET, "127.0.0.1", &addr.sin_addr.s_addr);
    int ret = connect(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("connect");
        exit(0);
    }

    // 通信
    while(1)
    {
        // 读数据
        char recvBuf[1024];
        // 写数据
        // sprintf(recvBuf, "data: %d\n", i++);
        printf("输入数据:");
        fgets(recvBuf, sizeof(recvBuf), stdin);
        write(fd, recvBuf, strlen(recvBuf)+1);
        // 如果客户端没有发送数据, 默认阻塞
        int len = read(fd, recvBuf, sizeof(recvBuf));
        if(len == -1)
        {
            perror("read error");
            exit(1);
        }
        printf("recv buf: %s\n", recvBuf);
        sleep(1);
    }

    // 释放资源
    close(fd); 

    return 0;
}
```

  ## poll

> 内核对应的文件描述符也是以`线性`的方式进行`轮训`，根据`描述符状态`进行处理，`poll`和`select`检测文件描述符集合会在检测过程中`频繁`进行`用户区`和`内核`去的`拷贝`，会随着文件描述符的增加而`线性增大`，从而`效率降低`。`select`检测文件描述符上限`1024`，`poll没有`最大文件描述符限制



### 函数

```c
#include <poll.h>
// 每个委托poll检测的fd都对应这样一个结构体
struct pollfd {
    int   fd;         /* 委托内核检测的文件描述符 */
    short events;     /* 委托内核检测文件描述符的什么事件 */
    short revents;    /* 文件描述符实际发生的事件 -> 传出 */
};

struct pollfd myfd[100];
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

### 参数

* `fds`: `struct pollfd`类型``数组``
* `events `和``events`可选参数

![event参数](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtkoJ8WITtTXibjkZ6lCabK2Yk4LaorTq6FAuF2e96QpOw4uYzUaPWpJhDCkibictkg1wKK03x7WfYUw/640?wx_fmt=png "event参数")

* nfds ：nfds参数数组中最后一个有效元素的下标+1
* timeout : 指定poll函数的阻塞时长

* 失败返回-1，成功返回一个大于0的整数，表示检测的集合中已就绪的文件描述符的总个数

### 代码服务端

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/select.h>
#include <poll.h>

int main()
{
    // 1.创建套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if(lfd == -1)
    {
        perror("socket");
        exit(0);
    }
    // 2. 绑定 ip, port
    struct sockaddr_in addr;
    addr.sin_port = htons(9999);
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(lfd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("bind");
        exit(0);
    }
    // 3. 监听
    ret = listen(lfd, 100);
    if(ret == -1)
    {
        perror("listen");
        exit(0);
    }
    
    // 4. 等待连接 -> 循环
    // 检测 -> 读缓冲区, 委托内核去处理
    // 数据初始化, 创建自定义的文件描述符集
    struct pollfd fds[1024];
    // 初始化
    for(int i=0; i<1024; ++i)
    {
        fds[i].fd = -1;  //初始化为-1
        fds[i].events = POLLIN;  //数据可读事件
    }
    fds[0].fd = lfd; 

    int maxfd = 0;
    while(1)
    {
        // 委托内核检测
        ret = poll(fds, maxfd+1, -1);
        if(ret == -1)
        {
            perror("select");
            exit(0);
        }

        // 检测的度缓冲区有变化
        // 有新连接
        if(fds[0].revents && POLLIN)
        {
            // 接收连接请求
            struct sockaddr_in sockcli;
            int len = sizeof(sockcli);
            // 这个accept是不会阻塞的
            int connfd = accept(lfd, (struct sockaddr*)&sockcli, &len);
            // 委托内核检测connfd的读缓冲区
            int i;
            for(i=0; i<1024; ++i)
            {
                if(fds[i].fd == -1) //选择一个存储
                {
                    fds[i].fd = connfd;  //连接的文件描述符
                    break;
                }
            }
            maxfd = i > maxfd ? i : maxfd;  //更新最大值
        }
        // 通信, 有客户端发送数据过来
        for(int i=1; i<=maxfd; ++i)
        {
            // 如果在集合中, 说明读缓冲区有数据
            if(fds[i].revents & POLLIN)
            {
                char buf[128];
                int ret = read(fds[i].fd, buf, sizeof(buf));
                if(ret == -1)
                {
                    perror("read");
                    exit(0);
                }
                else if(ret == 0)
                {
                    printf("对方已经关闭了连接...\n");
                    close(fds[i].fd);
                    fds[i].fd = -1;
                }
                else
                {
                    printf("客户端say: %s\n", buf);
                    write(fds[i].fd, buf, strlen(buf)+1);
                }
            }
        }
    }
    close(lfd);
    return 0;
}
```

## `epoll`

> 全称 `eventpol`l，是`linux`内核实现`IO多路转接/复用` 的一个实现。IO多路转接在一个操作里`同时`监听`多个输入输出源`，在其中一个或多个输入输出源可用的时候返回，然后对其进行读写操作。
>
> `select / pol`相对`低效`的原因之一是`添加/维护`待检测任务和阻塞进程/线程 两个步骤合二为一，在每次调用select都需要这两步，然而在大多数应用场景中，需要监视的`socket个数`相对`固定`，`epoll`将两个操作分开，`epoll_ctl`维护`等待队列`，在调用`epoll_wait`阻塞`进程`，进而提高`epoll的效率`

* `epoll`基于`红黑树`来管理`待检测集合`。
* `epoll`基于`回调机制`，而``select`和`poll`每次都会`线性``扫描整个待检测集合，集合越大速度越慢。
* 需要对select和poll的返回的`集合进行判断`才能知道那些文件描述符是`就绪的`，通过`epoll`可以`直接`得到已`就绪`的`文件描述符集`合，无需再次检测。

### 操作函数

```c
#include <sys/epoll.h>
//创建
//创建epoll实例，通过一个红黑树管理待检测集合,在lnnux内核2.6.8以后，只需指定大于0的数组
int epoll_create(int size);
// 返回值 -1 ：失败。 成功：返回一个有效的文件描述符，通过此刻访问创建的epoll实例


//管理
// 联合体, 多个变量共用同一块内存        
typedef union epoll_data {
 	void        *ptr;
	int          fd;	// 通常情况下使用这个成员, 和epoll_ctl的第三个参数相同即可
	uint32_t     u32;
	uint64_t     u64;
} epoll_data_t;

struct epoll_event {
	uint32_t     events;      /* Epoll events */
	epoll_data_t data; //用户数据变量，使用fd值，用于存储待检测的文件描述符值，在调用epoll_wait函数时值被传出
};
//evetns：epoll事件，修饰第三个fd对应文件描述符，检测指定对应的文件描述符的什么时间
//	EPOLLIN：读事件，接收数据，检测读缓冲区，如果有数据该文件描述符就绪
//	EPOLLOUT：写事件，发送数据，检测写缓冲区，如果可写该文件描述符就绪
//	EPOLLERR：异常事件
//  EPOLLET ：设置水平或边缘模式
// 管理红黑树上的文件描述符(添加,修改，删除)
int epoll_ctl(int epfd,int op,int fd,struct epool_event* event);
// epfd: epoll_create函数的返回值
// op ：枚举值，控制通过该函数执行什么操作
//      EPOLL_CTL_ADD：往 epoll 模型中添加新的节点
//      EPOLL_CTL_MOD：修改 epoll 模型中已经存在的节点
//      EPOLL_CTL_DEL：删除 epoll 模型中的指定的节点
// fd ： 文件描述符，即要添加/修改/删除的文件描述符
// 成功 返回0 
// 失败 返回-1


// 检测epoll树中是否有就绪的文件描述符
int epoll_wait(int epfd,struct epoll_event *event,int maxevents,int timeout);
// epfd : epoll_ceate函数返回值
// events ,传出参数，结构体数组地址，存储已就绪的文件描述符信息
// maxevents：结构体数组的容量(元素个数)
// timeout ：如果epoll没有已就绪的文件描述符，该函数的阻塞时长，单位ms毫秒
//			0函数不阻塞，epoll实例中没有就绪的文件描述符，函数被调用后直接返回，大于0，若epoll实例中没有已就绪的文件描述符，函数阻塞对应的毫秒数再返回，-1函数一直阻塞，直到epoll实例中有已就绪的文件描述符之后才解除阻塞

//返回值
//等于 0，函数阻塞被强制解除，没有检测到满足条件的文件描述符
//大于0  检测到的已就绪的文件描述符的总个数
//失败 返回 -1
```
### 操作步骤
* `epoll`创建监听的套接字
```c
int lfd = socket(AF_INET, SOCK_STREAM, 0);
```
* 设置`端口复用`(可选)

```c
int opt = 1;
setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
```
* `IP和端口`和`监听`的套接字进行`绑定`

```c
int ret = bind(lfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
```
* 给监听的套接字`设置监听`

```c
listen(lfd, 128);
```
* 创建`epoll实例对象`

```c
int epfd = epoll_create(100);  //一定要大于0的数
```
* 将用于监听的套接字`添加`到`epoll实例`中

```c
struct epoll_event ev;
ev.events = EPOLLIN;    // 检测lfd读读缓冲区是否有数据
ev.data.fd = lfd;  //键文件描述符添加到实例中
int ret = epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);
```
* 检测到`epoll实例`中文件描述符`是否已就绪`，并将这些已就绪的文件描述符`进行处理`

```c
int num = epoll_wait(epfd, evs, size, -1);
```
	* 如果是监听，和客户端建立`连接`，将得到的`文件描述符`添加到`epoll实例`中
```c
int cfd = accept(curfd, NULL, NULL);
ev.events = EPOLLIN;
ev.data.fd = cfd;
// 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
```

* 若是通信，和对应的客户端`通信`，如果连接已断开，将该文件描述符从`epoll实例`中删除
```c
int len = recv(curfd, buf, sizeof(buf), 0);
if(len == 0)
{
    // 将这个文件描述符从epoll模型中删除
    epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
    close(curfd);
}
else if(len > 0)
{
    send(curfd, buf, len, 0);
}
```
	* 重复最后一步

### `epoll工作模式`



#### `LT水平模式`

>`level triggered `同时`支持block和no-block socket`，内核通知使用者哪些文件描述符`已经就绪`，之后就可以对这些已就绪的文件描述符进行`IO操作`，若不进行任何操作，内核还是会继续通知使用者
>
>特点:
>
>* 读事件: 如果文件描述符对应的读缓存区还有数据，读事件就会被触发，`epoll_wait`解除阻塞。如果接受的数据`buf很小`，不能全部将缓存区全部数据读出，那么读事件会`继续`被触发，直到数据被`全部读出`，如果接受数据的内存相对较大，读数据的效率相对较高(减少了读数据次数)。读数据是被动的，必须要通过读事件才能知道有数据到达了，因此对于读事件的检测是必须的。
>* 写事件: 如果文件描述符对应的写缓存区可写，写事件就会被触发，`epoll_wait`解除阻塞。写事件的触发发生在写数据之前。`写`数据是`主动`行为，写缓存去一般情况下都是可写的(缓存区不满)，因此对于些时间的检测不是必须的。

##### 示例代码

```c

#include <stdio.h>
#include <ctype.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>

// server
int main(int argc, const char* argv[])
{
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
    
    // 设置端口复用
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 绑定端口
    int ret = bind(lfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    if(ret == -1)
    {
        perror("bind error");
        exit(1);
    }

    // 监听
    ret = listen(lfd, 64);
    if(ret == -1)
    {
        perror("listen error");
        exit(1);
    }

    // 现在只有监听的文件描述符
    // 所有的文件描述符对应读写缓冲区状态都是委托内核进行检测的epoll
    // 创建一个epoll模型
    int epfd = epoll_create(100); //参数为大于1即可，并没有实际含义
    if(epfd == -1)
    {
        perror("epoll_create");
        exit(0);
    }

    // 往epoll实例中添加需要检测的节点, 现在只有监听的文件描述符
    struct epoll_event ev;
    ev.events = EPOLLIN;    // 检测lfd读读缓冲区是否有数据
    ev.data.fd = lfd;
    ret = epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);
    if(ret == -1)
    {
        perror("epoll_ctl");
        exit(0);
    }

    struct epoll_event evs[1024];
    int size = sizeof(evs) / sizeof(struct epoll_event);
    // 持续检测
    while(1)
    {
        // 调用一次, 检测一次
        int num = epoll_wait(epfd, evs, size, -1);
        printf("num = %d \n",num);
        
        for(int i=0; i<num; ++i)
        {
            // 取出当前的文件描述符
            int curfd = evs[i].data.fd;
            // 判断这个文件描述符是不是用于监听的
            if(curfd == lfd)
            {
                // 建立新的连接
                int cfd = accept(curfd, NULL, NULL);
                // 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
                ev.events = EPOLLIN;    // 读缓冲区是否有数据
                ev.data.fd = cfd;
                ret = epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
                if(ret == -1)
                {
                    perror("epoll_ctl-accept");
                    exit(0);
                }
            }
            else
            {
                // 处理通信的文件描述符
                // 接收数据
                char buf[1024];
                memset(buf, 0, sizeof(buf));
                int len = recv(curfd, buf, sizeof(buf), 0);
                if(len == 0)
                {
                    printf("客户端已经断开了连接\n");
                    // 将这个文件描述符从epoll模型中删除
                    epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
                    close(curfd);
                }
                else if(len > 0)
                {
                    printf("客户端say: %s\n", buf);
                    send(curfd, buf, len, 0);
                }
                else
                {
                    perror("recv");
                    exit(0);
                } 
            }
        }
    }

    return 0;
}
```



#### ET边沿模式

> `edge-triggered`，只支持`no-block socket`。当文件描述符从未就绪变为就绪时，内核会通过`epoll通知使用者`，然后它会假设使用者知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知(`only one`)。ET模式在很大程度上减少`epoll事件`被重复触发的次数，`ET效率>LT模式`
>
> 特点:
>
> * 读事件: 当读缓存区有新数据进入，读事件`被触发一次`，`没有`新数据`不会`触发该事件。如果数据没有被全部读走，并且没有新数据进入，读事件不会再次触发，`只通知一次`，若有数据被全部读走或者只读走一部分，此时有新数据进入，读事件被触发，只通知一次
> * 写事件: 当写缓冲区状态可写，写事件智慧被触发一次。写缓存区从不满到被写满，从满到不满，写事件都`只会`被触发`一次`、
>
> epoll的边沿模式下epoll_wait检测到文件描述符有新事件才会通知，如果不是新的事件就不通知，`通知的次数`比水平模式`少`，`效率`比水平模式`高`。



##### ET模式的设置

```c
struct epoll_event ev;
ev.events = EPOLLIN | EPOLLET;	// 设置边沿模式 EPOLLET

//示例代码  线程安全的，不需要考虑加锁问题
int num = epoll_wait(epfd, evs, size, -1);
for(int i=0; i<num; ++i)
{
    // 取出当前的文件描述符
    int curfd = evs[i].data.fd;
    // 判断这个文件描述符是不是用于监听的
    if(curfd == lfd)
    {
        // 建立新的连接
        int cfd = accept(curfd, NULL, NULL);
        // 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
        // 读缓冲区是否有数据, 并且将文件描述符设置为边沿模式
        struct epoll_event ev;
        ev.events = EPOLLIN | EPOLLET;   
        ev.data.fd = cfd;
        ret = epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
        if(ret == -1)
        {
            perror("epoll_ctl-accept");
            exit(0);
        }
    }
}

```

##### 设置非阻塞

> 写事件一般写缓冲区有足够的空间，不需要进行检测。对于读事件的茶法就必须检测，使用epoll的边缘检测进行读事件检测，有新数据只会通知一次，就必须保证得到通知后将数据全部从读缓冲区中读出。

* 方式一:准备特大内存，用于存储缓冲区(内存无法定义，弊端大)
* 方式二：循环接受数据：将套接字设置为非阻塞模式

> 因为套接字默认是阻塞的，当读缓冲区的数据被读完之后，read/recv函数就被阻塞了，当前进程/线程就无法处理其他操作 -----将套接字默认修改为非阻塞

```c
#include <fcntl.h>
// 设置完成之后, 读写都变成了非阻塞模式
int flag = fcntl(cfd, F_GETFL); //取出cfd的flag属性
//将变量flag的标志位与O_NONBLOCK进行按位或操作，并将结果存储回flag中
flag |= O_NONBLOCK;     //设置flag为非阻塞属性         位运算符|（按位或）将O_NONBLOCK与变量的标志位进行按位或操作，将O_NONBLOCK标志位的值设置到变量的标志位中。                                          
fcntl(cfd, F_SETFL, flag); //flag属性被舍之道cfd当中

//循环接受数据
int len = 0;
while((len = recv(curfd, buf, sizeof(buf), 0)) > 0)
{
    // 数据处理...
    
    
    // 非阻塞模式下recv() / read()函数返回值 len == -1
int len = recv(curfd, buf, sizeof(buf), 0);
if(len == -1)
{
    
    
    
    //对于数据读完，仍然读取的保存，就行修改
    if(errno == EAGAIN)
    {
        printf("数据读完了...\n");
    }
    else
    {
        perror("recv");
        exit(0);
    }
}


```

##### 多线程`epoll`

```c
#include <stdio.h>
#include <ctype.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <errno.h>
#include <pthread.h>
//结构体存储要传递的数据
typedef struct socketinfo
{
    int fd;  //在子线程要操作的文件描述符可监听/读写
    int epfd; //要操作的epoll树实例
}SocketInfo;
//用于连接
void* acceptConn(void* arg)
{
    printf("子线程acceptConn线程ID:%ld\n",pthread_self());
    SocketInfo* info = (SocketInfo*)arg;
     // 建立新的连接
    int cfd = accept(info->fd, NULL, NULL);
    // 将文件描述符设置为非阻塞
    // 得到文件描述符的属性
    int flag = fcntl(cfd, F_GETFL);
    flag |= O_NONBLOCK;
    fcntl(cfd, F_SETFL, flag);
    // 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
    // 通信的文件描述符检测读缓冲区数据的时候设置为边沿模式
    struct epoll_event ev;
    
    ev.events = EPOLLIN | EPOLLET;    // 读缓冲区是否有数据
    ev.data.fd = cfd;
    int ret = epoll_ctl(info->epfd, EPOLL_CTL_ADD, cfd, &ev);
    if(ret == -1)
    {
        perror("epoll_ctl-accept");
        free(info);
        exit(0);
    }
    free(info);
    return NULL;
}

//用于通信函数
void* communication(void* arg)
{
    printf("子线程communication线程ID:%ld\n",pthread_self());
    SocketInfo* info = (SocketInfo*)arg;
    // 处理通信的文件描述符
    // 接收数据
    char buf[5];
    char temp[1024];
    bzero(temp,sizeof(temp)); //数据后面加入 /0
    memset(buf, 0, sizeof(buf));
    // 循环读数据
    while(1)
    {
        int len = recv(info->fd, buf, sizeof(buf), 0);
        if(len == 0)
        {
            // 非阻塞模式下和阻塞模式是一样的 => 判断对方是否断开连接
            printf("客户端断开了连接...\n");
            // 将这个文件描述符从epoll模型中删除  //线程安全的函数，不需要进行线程同步
            epoll_ctl(info->epfd, EPOLL_CTL_DEL, info->fd, NULL);
            close(info->fd);
            break;
        }
        else if(len > 0)
        {
            // 通信
            for(int i=0;i<len;++i)
            {
                buf[i]=toupper(buf[i]);
            }
            strncat(temp+strlen(temp),buf,len);
            // 接收的数据打印到终端  
            //修改到读取完数据之后一起发送
            // write(STDOUT_FILENO, buf, len);
            // // 发送数据
            // send(info->fd, buf, len, 0);
        }
        else
        {
            // len == -1
            if(errno == EAGAIN) //读缓存区已经没有了数据
            {
                printf("数据读完了...\n");
                send(info->fd,temp,strlen(temp)+1,0);
                break;
            }
            else
            {
                perror("recv");
                break;
            }
        }
    }
    free(info);
    return NULL;
}

// server
int main(int argc, const char* argv[])
{
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
    if(ret == -1)
    {
        perror("bind error");
        exit(1);
    }

    // 监听
    ret = listen(lfd, 64);
    if(ret == -1)
    {
        perror("listen error");
        exit(1);
    }

    // 现在只有监听的文件描述符
    // 所有的文件描述符对应读写缓冲区状态都是委托内核进行检测的epoll
    // 创建一个epoll模型
    int epfd = epoll_create(100);
    if(epfd == -1)
    {
        perror("epoll_create");
        exit(0);
    }

    // 往epoll实例中添加需要检测的节点, 现在只有监听的文件描述符
    struct epoll_event ev;
    ev.events = EPOLLIN;    // 检测lfd读读缓冲区是否有数据
    ev.data.fd = lfd;
    ret = epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);
    if(ret == -1)
    {
        perror("epoll_ctl");
        exit(0);
    }


    struct epoll_event evs[1024];
    int size = sizeof(evs) / sizeof(struct epoll_event);
    // 持续检测
    while(1)
    {
        // 调用一次, 检测一次
        int num = epoll_wait(epfd, evs, size, -1);
        printf("==== num: %d\n", num);
        
        pthread_t tid;
        for(int i=0; i<num; ++i)
        {
            // 取出当前的文件描述符
            SocketInfo* info=(SocketInfo*)malloc(sizeof(SocketInfo));
            int curfd = evs[i].data.fd;
            info->fd = curfd;
            info->epfd = epfd;
            // 判断这个文件描述符是不是用于监听的
            if(curfd == lfd)
            {

                pthread_create(&tid,NULL,acceptConn,info);
                pthread_detach(&tid); //线程分离
            }
            else
            {
                pthread_create(&tid,NULL,communication,info);
                pthread_detach(&tid); //线程分离
            }
        }
    }

    return 0;
}
```



