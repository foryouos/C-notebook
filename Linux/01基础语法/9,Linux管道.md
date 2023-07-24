# 进程

##  进程命令
```sh
pa aux
```
* `a`:查看所有`终端`的信息
* `u`:查看所有`用户相关`的信息
* `x`:查看和`终端无关`的进程信息
## 杀死进程
> `kill `
```sh
# 无条件杀死进程，进程ID通过PS aux查看
kill -9 进程ID
kill -SIGKLL 进程ID
```

## 进程函数
* 获取当前进程的`ID`
```c
#include <sys/types.h>
#include <unistd.h>
pid_t getpid(void);
```
* 获取当前进程的`父进程`
```c
#include <sys/types.h>
#include <unistd.h>
pid_t getppid(void);
```
* `创建`新的进程
> 启动磁盘上的应用程序，得到一个进程，调用`fork()`函数，得到一个`子进程`，子进程的地址空间是基于`父进程`地址空间拷贝出。
> 拷贝完成之后，两个地址空间中`用户区数据`是`相同`的。
> 代码区，父子进程`源代码始终相同`，全局数据区，堆区，动态库加载区(内存映射区),栈区，环境变量，文件描述表都始终相同
> 区别:父子进程各自的虚拟空间是相互独立的，不会相互干扰和影响，父子进程ID不同，fork的函数返回值不同，`父进程返回值``大于`0，子进程返回值等于`0`
```c
#include <unistd.h>
pid_t fork(void);
```
> 两个进程是不能通过`全局变量`实现`数据交互`的，每个进程都有自己的地址空间。
> 进程通信:`管道`，`共享内存`，`本地套接字`，`内存映射区`，`信息队列`等

### 通过一个进程启动另一个进程
####  `execl()`

```c
#include <unistd.h>
// 变参函数
int execl(const char *path, const char *arg, ...);
```
* `path:`要启动的可执行程序路径，推荐绝对路径
* `arg`：` pa aux`查看进程，启动的进程名字，一般和要启动的可执行程序名相同
* `... `要执行的命令参数，可多个，最后以`NULL`结尾表示参数`指定完毕`
* `返回值`：成功没有返回值，失败` -1`

#### `execlp()`

> 该函数执行已经设置了`环境变量`的`可执行程序`，不需要指定路径
```c
// p == path
int execlp(const char *file, const char *arg, ...);
```

#### 函数使用
> 一般在子进程中调用`exec族函数`，子进程的用户区数据被替换掉开始执行新的程序中的`代码逻辑`，但是父进程不受任何影响，仍然可以继续工作

## 进程控制
> 进程退出，进程的回收，和进程特殊状态：`孤儿进程`和`僵尸进程`

### 结束进程
> `status`为进程退出参数的`退出码`，`参数值是0`，退出之后`返回0`
```c
// 专门退出进程的函数, 在任何位置调用都可以
// 标准C库函数
#include <stdlib.h>
void exit(int status);

// Linux的系统函数
// 可以这么理解, 在linux中 exit() 函数 封装了 _exit()
#include <unistd.h>
void _exit(int status);
```
### 孤儿进程
> 父进程`先与子进程`结束，这时候子进程就成为`孤儿进程`。当子进程编程孤儿进程，就会有一个固定的进程领养这个孤儿进程,若`linux`没有桌面端，就是`进程init进程(PID =1)`.如果有桌面终端，就是`桌面进程`
> 在子进程退出时，用户去可以自己释放，但进程内核去`PCB`需要父进程来释放，领养的进程将`代劳`释放进程退出时的PCB资源，避免系统资源的浪费。

### 僵尸进程
> 子进程和父进程结束，子进程无法释放自己的PCB资源，父进程不作为，此时子进程编程僵尸进程。
> PS aux查看进程信息， `Z+`表示僵尸进程
> `kill -9 僵尸进程PID`

### 进程回收
#### `wait`

> `阻塞函数`，当检测到子进程退出，该函数阻塞`回收子进程`资源，该函数`被调用一次`，如`多个子进程`回收资源，需要`调用多次`
```c
// man 2 wait
#include <sys/wait.h>

pid_t wait(int *status);
```
> 参数为传出参数，若不需要指定为NULL，
> 返回值:成功返回被回收的`子进程ID`，`即大于0`
> 失败`-1`，没有子进程回收，或者回收进程出现异常

#### `waitpid`

> 该函数可以`精准指定回收某个`或者`某一类或者全部子进程`资源
```c
// man 2 wait
// man 2 waitpid
#include <sys/wait.h>
// 这个函数可以设置阻塞, 也可以设置为非阻塞
// 这个函数可以指定回收哪些子进程的资源
pid_t waitpid(pid_t pid, int *status, int options);
//参数
//pid ： -1:无差别回收，大于0：指定某个进程的资源，0回收当前进程组所有子进程ID，小于-1:pid的绝对值代表进程组ID，便是要回收这个进程组的所有子进程资源，
//status:NULL
//options:控制函数是阻塞还是非阻塞 0函数阻塞和wait一样，WNOHANG函数行为是非阻塞
//返回值
//如果函数是非阻塞的，并且进程还在运行，返回0
//成功:得到子进程的进程ID
//失败:-1，没有子进程资源可以回收，如果是阻塞，阻塞会解除，直接返回-1.回收子进程资源的时候出现异常
```
## 线程数据资源共享

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
void* thread1_function(void* arg);
void* thread2_function(void* arg);
int count = 5;  //全局，两个线程使用同一个count
int main(void)
{
	pthread_t pthread1,pthread2; //创建新的线程
	int ret;
	//线程用到的所有资源都依赖进程
	ret = pthread_create(&pthread1, NULL, thread1_function,NULL); //创建一个新的线程
	if (ret != 0)
	{
		perror("pthread1_creator");
		exit(1);
	}
	//创建第二个进程
	ret = pthread_create(&pthread2, NULL, thread2_function, NULL);
	if (ret != 0)
	{
		perror("pthread2_creator");
		exit(1);
	}
	//while (1);
	//等线程pthread1结束pthread_join才会接着运行
    //等待两个线程结束
	pthread_join(pthread1, NULL);
	pthread_join(pthread2, NULL);
	printf("Thre thread is over ,process is over too\n");
	return 0;
} 
void *thread1_function(void* arg) //arg为count指针
{
	//int i;
	printf("Thread1 begin running\n");
	//for (int i = 0; i < *(int*)arg; i++)
	while(1)
	{
		printf("Hello world %d \n",count++);
		sleep(1);
	}
	return NULL;
}
void* thread2_function(void* arg) //arg为count指针
{
	printf("Thread2 begin running \n");
	while (1)
	//for (int i = 0; i < *(int*)arg; i++)
	{
		printf("Good Morning  %d \n",count ++);
		sleep(1);
	}
	return NULL;
}
//输出:
Thread1 begin running
Hello world 5 
Thread2 begin running
Good Morning6 
Hello world 7 
Good Morning8 
Hello world 9 
Good Morning10
.....
```
# 管道
> 进程之间`通信`（`IPC`)：管道的`本质`就是`内核中的一块内存`(或者内存缓冲区，这个缓冲区的数据存储在一个`环形队列`中，管道对应缓冲区大小`固定`位`4K`，管道中的数据`只能读一次`，读一次操作之后就没有，管道是`单工的`，只能`单向流动`，对管道的`读写操作`，默认是`阻塞的`。)
```c
// 读管道
ssize_t read(int fd, void *buf, size_t count);
// 写管道的函数
ssize_t write(int fd, const void *buf, size_t count);
```
## 无名管道
> `管道`没有名字，匿名管道只能实现`有血源关系`的进程间通信。
> `#include <unistd.h>`
> 创建一个`匿名`的管道, 得到`两个可用的文件描述符`
> `int pipe(int pipefd[2]);`
> 创建管道的函数`pipe`,`双向数据传输，`
> `pipefd[0]`：对应`读`端的文件描述符
> `pipefd[1]` ：对应`写`端的文件描述符
> 返回值: 成功返回`0`，失败返回`-1`

### 发送与容量
> 父进程:`写管道`
> 子进程:`读管道`
> 测试管道能够容纳的容量 :管道的`最终容量`为 `65536`
```c
/*
    parent process:read pipe
    child process : write pipe

    parent 不断地写数据，
    Child 
*/

#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <wait.h>
int main(void)
{
    int fd[2];  //0读端，1写端
    pid_t pid; //进程输出
    if(pipe(fd) == -1) //创建管道
    {
        perror("pipe");
    }
    pid = fork(); //创建进程
    //根据pid的不同，判断父进程还是子进程，进行进程的不同处理
    if(pid > 0)
    {
        //接受数据代码
        // char ch[2]; //接受数据
        // //父进程
        // printf("Child process is waiting for data :\n");
        // close(fd[1]);
        // while(1)
        // read(fd[0],ch,2);  //如果没有等到，管道内部是阻塞的，就会一直阻塞在此处
        // printf("read from the pid : %s \n",ch);

        //测试管道容量代码
       waitpid(pid,NULL,0); //等待子进程的结束
        
    }
    else if(pid ==0)
    {
        //进入子进程
        printf("子进程开始发数据\n");
        close(fd[0]);
        sleep(5);  //延迟屋面，将数据发送出去
        int n = 0;  //判断可以管道的容量
        
        //管道写到65536个字节后，停止
        while (1)
        {
            /* code */
            write(fd[1],"ad",2);   //要相互对应，避免被阻塞
            n= n+2;
            printf("count = %d \n",n);
        }
    }

    return 0;
}
```
## 单向管道通信
```c
/*
    父进程:读数据
    子进程:写数据
*/

#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <wait.h>
int main(void)
{
    int fd[2];  //0读端，1写端
    pid_t pid; //进程输出
    if(pipe(fd) == -1) //创建管道
    {
        perror("pipe");
    }
    pid = fork(); //创建进程
    //根据pid的不同，判断父进程还是子进程，进行进程的不同处理
 
    if(pid ==0)
    {
        printf("您进入子进程\n");
        //进入子进程
        char tmp[100]; //往里面写数据的缓存区
        close(fd[0]);
        //成一直写数据
        while (1)
        {
            printf("输入你要写入的数据:");
            scanf("%s",tmp); //往里面写数据
            /* code */
            write(fd[1],tmp,sizeof(tmp));   //要相互对应，避免被阻塞
            sleep(1);
        }
    }
    else if(pid > 0)
    {
        printf("您进入父进程\n");
        //父进程
        char tmprec[100]; //用于接受数据的缓存区
        close(fd[1]); //关闭写
        //父进程一直读数据
        while(1)
        {
            printf("waiting the pipe write\n");
            read(fd[0],tmprec,sizeof(tmprec)); //没有读到会阻塞
            printf("read from child pipe: %s\n",tmprec);
            sleep(1);
        }
        
    }
    return 0;
}
```
## 双向管道通信
> 父进程和子进程`双向通信`，需要`两条管道`
```c
/*
    双向管道通信
    需要两个管道
*/

#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <wait.h>
#include <string.h>
#include <ctype.h>
int main(void)
{
    int fd[2];  //0读端，1写端
    int fd2[2]; //第二个管道
    pid_t pid; //进程输出
    if(pipe(fd) == -1) //创建管道
    {
        perror("pipe1");
    }
    if(pipe(fd2) == -1)
    {
        perror("pipe2");
    }

    //创建进程
    pid = fork(); 
    //根据pid的不同，判断父进程还是子进程，进行进程的不同处理
 
    if(pid ==0)
    {
        printf("您进入子进程\n");
        //管道1负责读父进程的数据
        //管道2负责向父进程写数据
        close(fd[1]); //关闭管道1的写端
        close(fd2[0]); //关闭管道2的读端
        //进入子进程
        char tmp[100]; //往里面写数据的缓存区
        //成一直写数据
        while (1)
        {
            //现将清空
            memset(tmp,0,sizeof(tmp));
            read(fd[0],tmp,sizeof(tmp));

            //对读到的字符都转成大写
            for(int i=0;i<sizeof(tmp);i++)
            {
                tmp[i] = toupper(tmp[i]);
            }
            //将大写的数据通过管道2发送过去
            write(fd2[1],tmp,sizeof(tmp));   //要相互对应，避免被阻塞
            
        }
    }
    else if(pid > 0)
    {
        printf("您进入父进程\n");
        close(fd[0]); //关闭管道1的读端
        close(fd2[1]); //关闭管道2的写端
  
        char tmprec[100]; //用于接受数据的缓存区
       
        //父进程一直读数据
        while(1)
        {
            memset(tmprec,0,sizeof(tmprec));
            gets(tmprec);
            write(fd[1],tmprec,sizeof(tmprec));
            //通过管道2读
            memset(tmprec,0,sizeof(tmprec));
            read(fd2[0],tmprec,sizeof(tmprec)); //没有读到会阻塞
            printf("After Changed: %s\n",tmprec);
        }
        
    }
    return 0;
}/*
    双向管道通信
    需要两个管道
*/

#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <wait.h>
#include <string.h>
#include <ctype.h>
int main(void)
{
    int fd[2];  //0读端，1写端
    int fd2[2]; //第二个管道
    pid_t pid; //进程输出
    if(pipe(fd) == -1) //创建管道
    {
        perror("pipe1");
    }
    if(pipe(fd2) == -1)
    {
        perror("pipe2");
    }

    //创建进程
    pid = fork(); 
    //根据pid的不同，判断父进程还是子进程，进行进程的不同处理
 
    if(pid ==0)
    {
        printf("您进入子进程\n");
        //管道1负责读父进程的数据
        //管道2负责向父进程写数据
        close(fd[1]); //关闭管道1的写端
        close(fd2[0]); //关闭管道2的读端
        //进入子进程
        char tmp[100]; //往里面写数据的缓存区
        //成一直写数据
        while (1)
        {
            //现将清空
            memset(tmp,0,sizeof(tmp));
            read(fd[0],tmp,sizeof(tmp));

            //对读到的字符都转成大写
            for(int i=0;i<sizeof(tmp);i++)
            {
                tmp[i] = toupper(tmp[i]);
            }
            //将大写的数据通过管道2发送过去
            write(fd2[1],tmp,sizeof(tmp));   //要相互对应，避免被阻塞
            
        }
    }
    else if(pid > 0)
    {
        printf("您进入父进程\n");
        close(fd[0]); //关闭管道1的读端
        close(fd2[1]); //关闭管道2的写端
  
        char tmprec[100]; //用于接受数据的缓存区
       
        //父进程一直读数据
        while(1)
        {
            memset(tmprec,0,sizeof(tmprec));
            gets(tmprec);
            write(fd[1],tmprec,sizeof(tmprec));
            //通过管道2读
            memset(tmprec,0,sizeof(tmprec));
            read(fd2[0],tmprec,sizeof(tmprec)); //没有读到会阻塞
            printf("After Changed: %s\n",tmprec);
        }
        
    }
    return 0;
}
```
## 有名管道
> `有名管道`拥有管道的`所有特性`，有名是因为管道在磁盘中有`实体文件`，有名管道的大小永远为0，有名管道的数据也是存储到内存的缓冲区，打开管道文件可以操作有名管道的`文件描述符`，通过文件描述符读写管道存储在内核中的数据
> 通过指令的方式`mkfifo`
> 函数的方式
> `mkfifo`函数创建一个`先进先出`的特殊文件 有名管道
```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);  //mode : 666所有可读写

#include <fcntl.h>           /* Definition of AT_* constants */
#include <sys/stat.h>

int mkfifoat(int dirfd, const char *pathname, mode_t mode);
```
## 有名管道通信
### `write`

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>
int main(int argc,char*argv[]) 
{
    int ret;  //创建有名管道的ret
    int fd;  //打开文件的fd
    char buf[100]; //数据缓存区
   
    ret = mkfifo("./myfifo",0664);
    if(ret!=-1)
    {
        perror("mkfifo");
    }
    //main带参数，要写入的内容
    //发送的数据，在argv里面
    printf("进入写管道\n");

    fd = open("./myfifo",O_WRONLY); //以写的当时打开
    if(fd == -1)
    {
        printf("打开文件失败");
        perror("open");
    }
    printf("打开文件成功");
    if(argc == 1)
    {
        printf("Please send something to the named pipe:\n");
        exit(EXIT_FAILURE);
    }
    
    strcpy(buf,argv[1]);
    printf("您输入的数据: %s\n",buf);
    write(fd,buf,sizeof(buf));
    printf("将数据写进去了");
    printf("Write to the pipe : %s\n",buf);
    close(fd);
    return 0;
}
```

### `read`

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
int main(void)
{
    int fd;  //打开文件的fd
    char buf[100]; //数据缓存区
    printf("Prepare reading from named pipe:\n");
    fd = open("./myfifo",O_RDWR);
    if(fd ==-1)
    {
        perror("open");
    }
    while (1)
    {
        /* code */
        //读管道,若没有数据，或阻塞在此处，直到有数据
        memset(buf,0,sizeof(buf));
        int len = read(fd,buf,sizeof(buf));
        
        printf("Read from named pipe:%s \n",buf);
        if(len == 0)
        {
            printf("管道的写端已经关闭....\n");
            break;
        }
        sleep(1);
    }
    close(fd);
    return 0;
}
```
# 内存映射
> 内存映射对应的`内存空间`在进程的用户(用于加载动态库的区域)。进程通信使用的内存映射区`不是一块`，而是每个进程内部都有一块。
> 由于每个进程的地址空间是独立的，各个进程之间也不能直接访问对方的内存映射区，需要通信的进程需要将各自的内存映射区和同意磁盘文件进行映射，这样进程之间就可以通过磁盘文件这一个唯一桥梁完成数据交互可实现`有无血源关系`的`双向通信`

## 创建
```c
#include <sys/mman.h>
// 创建内存映射区
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```
参数:
	* `addr`：从动态库加载到什么位置创建内存映射区，一般指定为`NULL，`委托内核分配
	* `length`:创建的内存映射区大小,实际按照`4K`整数倍去分配
	* `por`t:对内存映射区的操作权限
		* `PORT_READ`：`读`内存映射区
		* `PORT_WRITE`：`写`内存映射区
		* 有`读写`权限: `PROT_READ | PROT_WRITE`
	* `flags`：
		* `MAP_SHARED`:`多个`进程可以共享数据，进行映射区数据同步，
		* `MAP_PRIVATE:`映射区数据`私有`，不能同步给其他进程
		*` fd `：`文件描述符`，对应一个打开的`磁盘文件`，内存映射区通过这个文件描述符和磁盘文件建立关联
		*` offset`:磁盘文件的`偏移量`，文件从偏移的位置开始进行数据映射
		* 偏移量必须是`4K`的`整数倍`，写`0`代表`不偏移`，参数必须是`大于0`
		返回值:
	* 成功:返回一个`内存映射区的起始地址`
	* 失败:`MAP_FALED`(that is, (void *) -1)

## 释放
```c
int munmap(void *addr, size_t length);
```
参数:
	* `addr：mmap()`的返回值，创建的内存映射区的`起始地址`
	* `length`和`mmap()`第二个参数相同即可
	返回值:函数调用`成功返回0`，`失败返回-1`

# 共享内存
> 共享内存效率最高，不同于内存映射区，不属于任何进程，通过调用Linux提供的系统函数得到这块内存。
> 先开辟一个内存空间，把此内存空间`映射`到不同的进程中去，创建映射，解除映射，删除映射。
```c
#include <sys/ipc.h>
#include <sys/shm.h>
int shmget(key_t key, size_t size, int shmflg);
//key值是内核标识的排他性的共性内存描述符
```
参数:
	*` key`: 类型` key_t` 是个整形数，通过这个`key`可以创建或者打开一块共享内存，该参数的值一定要大于0
	* 指定共享`内存的大小`
	* `shmflg`：创建共享内存指定的属性
		* `IPC_CREAT`：`创建`新的共享内存，如果徐璈指定操作权限:`IPC_CREAT | 0664`
		*` IPC_EXCL`：检测`共享内存`是否已经`存在`，要和 `IPC_CREAT `一起使用
		返回值:
	* 共享内存创建或者打开成功返回标识共享内存的`唯一ID`，`失败`返回`-1`

```c
// 	如果共享内存已经存在, 共享内存创建失败, 返回-1, 可以perror() 打印错误信息
shmget(100, 4096, IPC_CREAT|0664|IPC_EXCL);
```

### `ftock`

> `shmget`第一个参数是`大于0`的正整数，尅通过`ftok()`函数直接生成这个`key`值
```c
// ftok函数原型
#include <sys/types.h>
#include <sys/ipc.h>

// 将两个参数作为种子, 生成一个 key_t 类型的数值
key_t ftok(const char *pathname, int proj_id);
//pathname:当前操作系统中一个存在的路径
//proj_id:只用到int中的一个字节，传参的时候将其作为char进行操作，取直返回为:1-255
//返回值:函数调用成功返回一个可用于创建，打开共享内存的key值，调用失败返回-1
// 根据路径生成一个key_t
key_t key = ftok("/home/robin", 'a');
// 创建或打开共享内存
shmget(key, 4096, IPC_CREATE|0664);
```

### 练习

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
char *msg = "hello";

int main(void)
{
    int shmid;
    pid_t pid;
    //让两个进程使用共享内存通信IPC_PRIVATE表示创建一个新的共享内存区0666表示指定共享内存的权限。
    shmid = shmget(IPC_PRIVATE,2048,IPC_CREAT | 0666); //IPC_CREAT
    //创建进程
    pid = fork();
    //根据pid范围值不同，判断子进程或者父进程
    if(pid > 0)
    {
        //父进程
        //将共享内存映射到当前父进程中
        //第一个参数:映射那个共享内存，第二个参数:映射到本进程为止，NULL交给内核处理,第三个参数为0,拥有读写权限
        //返回映射共享内存的首地址
        char * p_addr; //所有对共享内存的操作，相当于对p_addr
        p_addr = shmat(shmid,NULL,0);

        memset(p_addr,'\0',strlen(msg)+1); //清空，

        memcpy(p_addr,msg,strlen(msg)+1); //写入msg

        //解除绑定
        shmdt(p_addr);
        //父进程等待子进程结束
        waitpid(pid,NULL,0); //第二个参数指针
        
    }
    else if(pid ==0)
    {
        //子进程
        //先绑定
        char * c_addr; //所有对共享内存的操作，相当于对c_addr
        //创建关联
        c_addr = shmat(shmid,NULL,0);
        //子进程读
        printf("Child Process waits a short time: \n");
        sleep(3);
        //读共享内存c_addr数据数据
        printf("Child Process reads from shared memory :%s\n",c_addr); //从c_addr读取数据，并打印
        //解除与共享内存的绑定
        shmdt(c_addr);
    }
    else
    {
        perror("fork");
    }
    return 0;
}
//运行结果
//foryouos@bottle:~/project/共享内存$ gcc shm.c -o shm
//foryouos@bottle:~/project/共享内存$ ./shm
//Child Process waits a short time: 
//Child Process reads from shared memory :hello
```

### 非亲缘内存通信

> 删除共享内存
> `int shmctl(int shmid, int cmd, struct shmid_ds *buf);`

* read.c
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#define MY_KEY 9527
int main(void)
{
    int shmid;
    //打开的作用
    //MY_KEY已经存在，起到一个读的作用，编号要和创建时的编号相同
    shmid = shmget(MY_KEY,2048,IPC_CREAT | 0666); //IPC_CREAT
    //先绑定
    char * c_addr; //所有对共享内存的操作，相当于对c_addr
    c_addr = shmat(shmid,NULL,0);
    // 读共享内存c_addr数据数据
    printf("Read from shared memory :%s\n",c_addr); //从c_addr读取数据，并打印
    // 解除与共享内存的绑定
    shmdt(c_addr);
    //删除功能内存
    shmctl(shmid,IPC_RMID,NULL);
    return 0;
}
```
* write.c
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#define MY_KEY 9527
char *msg = "hello";

int main(void)
{
    int shmid;
    //让两个进程使用共享内存通信IPC_PRIVATE表示创建一个新的共享内存区0666表示指定共享内存的权限。
    //
    shmid = shmget(MY_KEY,2048,IPC_CREAT | 0666); //IPC_CREAT
    //第一个参数:映射那个共享内存，第二个参数:映射到本进程为止，NULL交给内核处理,第三个参数为0,拥有读写权限
    char * p_addr; //所有对共享内存的操作，相当于对p_addr
    p_addr = shmat(shmid,NULL,0); //绑定当前进程来
    memset(p_addr,'\0',strlen(msg)+1); //清空，
    memcpy(p_addr,msg,strlen(msg)+1); //写入msg
    //解除绑定
    shmdt(p_addr);
    return 0;
}
```
# 共享内存和内存映射区区别
* 实现通信方式:
	* `shm`: 多个进程只需要`一块共享内存`就够了，共享内存不属于进程，需要和进程关联才能使用
	* `内存映射区`：位于每个进程的虚拟地址空间中，并且需要`关联同一个磁盘文件`才能实现进程间数据通信
* 效率:
	* `shm`: 直接对`内存操作，效率高`
	* 内存映射区：需要内存和`文件之间的数据同步，效率低`
* 生命周期
	* 内存映射区：进程退出，内存映射区也就没有了
	* `shm`：进程`退出对共享内存没有影响`，调用相关函数 / 命令 / 关机才能删除共享内存
* 数据的完整性 -> 突发状态下数据能不能被保存下来（比如：突然断电）
	* 内存映射区：可以`完整`的保存数据，内存映射区数据会`同步到磁盘文件`
	* `shm`：数据存储在`物理内存中`，断电之后系统关闭，内存数据也就丢失了




## 参考资料:

* Linux 系统编程`7~13`
* 爱编程的大丙