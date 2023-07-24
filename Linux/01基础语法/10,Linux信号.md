# 信号

> `信号`本质是一个`整数`，不同的信号对应不同`的值`，由于信号的结构简单，不能携带大量信息量，在系统中的`优先级非常高`

## 信号编号

![信号编号](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVekAeF40TOFSQZEJriaz5jNYmwCS5ZTVZSRWQkYpbTu3M7aQUTbMALeBGg/640?wx_fmt=png "信号编号")

* `SIGHUP `: 用户`退出shell`时，由shell启动的`所有进程都收到此信号`
* `SIGINT` ： 用户按`下Ctrl + C` 组合键，用户终端向所有`正在运行`的由该终端启动的程序发出`此信号`

```sh
// 查看man文档的信号描述
man 7 signal
```

## 信号状态

* `产生`：输入，函数`调用`，执行设立了`指令`
* `未决`：信号残生，但此信号还没有被`处理掉`
* `抵达:` 信号`被处理了`

## 信号函数

```sh
#inlcude <signal.h>
//给某个进程发送信号
int kill(pid_t pid,int sig);
//pid 进程ID
//sig 要发送的信号

//杀死自己
kill(getpid(),9);
// 子进程杀死自己父进程
kill(getppid(),10);

//raise给当前进程发送指定的信号
int raise(int sig); //参数就是要给当前进程发送信号
// abort给当前进程发送一个固定信号(SIGABRT)
#include <stdlib.h>
void abort(void);

//定时器
unsigned int alarm(unsigned int seconds);
// 倒计时seconds秒，倒计时完成发送一个信号SIGALRM,当前进程收到这个信号，默认处理中断当前进程
// 大于0 表示倒计时剩余多少秒，返回0表示倒计时完成
```

### 计算一秒数数

> 文件`IO`操作需要进行`用户区到核心区`的切换，处理方式不同，两者`切换频率`不同。对`文件IO`操作进行优化是可以`提供程序`的执行效率。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main()
{
    // 1. 设置一个定时器, 定时1s
    alarm(1);	// 1s之后会发出一个信号, 这个信号将中断当前进程
    int i = 0;
    while(1)
    {
        printf("输出信号 : %d\n", i++);
    }
    return 0;
}
// 运行数据 直接通过终端数据
time ./time 
// 将数据重定向到磁盘文件中 ,用户实际时间增长
time ./time > a.txt
```

![输出信号](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVekCmSfyqTdFH9RsS3hxBsjHibaryFwSS2ibyMniarkicH508TGib0GxUzkQBA/640?wx_fmt=png "输出信号")

## 周期性信号函数

> `setitimer()`函数可以进行周期性定时，`每`触发`一次`就会`发射出`一个`信号`

```c
#include <sys/time.h>

struct itimerval {
	struct timeval it_interval; /* 时间间隔 */
	struct timeval it_value;    /* 第一次触发定时器的时长 */
};
// 举例: 
// 早晨5点起床, 第一次闹钟响起时可能起不来, 之后每隔5分钟再响一次
//  - it_value: 当前设置闹钟的时间点 到 明天早晨7点 对应的总秒数
//  - it_interval: 闹钟第一次响过之后, 每隔5分钟响一次

// 这个结构体表示的是一个时间段: tv_sec + tv_usec
struct timeval {
	time_t      tv_sec;         /* 秒 */
	suseconds_t tv_usec;        /* 微妙 */
};

int setitimer(int which, const struct itimerval *new_value, 
              struct itimerval *old_value);
// which 定时器使用什么样的计时法
    * ITIMER_REAL:自然计数法，发出的信号为SIGALRM,自然计时法时间=用户区+内核+消耗的时间(从进程的用户区到内核区切换的总时间)
    * ITIMER_VIRTUAL:只计算用户区运行使用的时间，发射的信号为SIGVTALRM
    * ITIMER_PROF: 只计算内核运行使用的时间，发出的信号为SIGPROF
// new_value :定时器设置的定时信息，传入参数
// old_value :上一次给定时器设置的定时信息，传出参数，如果不需要指定为NULL
```

## 信号集

* `信号`的`未决`是一种状态，指的是信号的产生到信号`被处理前`的这`一段时间`
* 信号的`阻塞`是一个`开关动作`，值得使阻止信号被处理，但不是`阻止信号产生`。信号的阻塞就是让系统`暂时保留`信号待以后发送。

> 阻塞信号集和未决信号集在内核中`结构相同`，都是`整型数组`，一共`128字节`，`1024`个标志位，前`31`个标志位。
>
> 默认`没有信号被阻塞`，信号标识为为`0`，`阻塞`状态为`1`. 
>
> 如果信号`被阻塞`，不能处理则`标志位`被设置为`1`
>
> 阻塞`被解除`，未决信号`马上被处理`，信号标志位为`0`



### 信号集函数

#### `sigprocmask`读写信号集

```c
int sigprocmask(int how,const sigset_t *set,sigset_t *oldset);
//参数
// how
	* SIG_BLOCK:将参数set集合中数据添加到阻塞信号集中
    * SIG_UNBLOCK:将参数set集合中在阻塞信号中解除阻塞
    * SIG_SETMASK : 使参数set集合中的数据覆盖内核的阻塞信号集数据
//oldset:通过此参数将之前的阻塞数据集传出，若不需要指定NULL
//返回值：调用成功返回0，调用失败返回-1
```

#### `sigset_t`s初始化

> 在程序中读写`sigset_t`类型的变量，阻塞信号和未决信号集都存储在`sigset_t`类型的`变量中`，这个变量对应一块`内存`，则是`信号集`和`未决信号集`，对应内存`1024bit = 128`字节

```c
//将set集合中所有的标志位设置为0
int sigemptyset(sigset_t *set);
// 将set集合中所有的标志为设置为1
int sigfillset(sigset_t *set);
// 将set集合中的某一个信号(signum)对应的标志位设置为1
int sigaddset(sigset_t *set,int signum);
//将set集合中某一个信号(signum)对应的标志位设置为0``
int sigdelset(sigset_t *set,int signum);
//判断某个信号在集合中对应的标志位是0还是1，返回对应标志位
int sigismember(const sigset_t *set,int signum);
//读内核未决信号集
int sigpending (sigset_t *set);
```

## 信号捕捉

> 程序中每个信号产生之后都会有对应的`默认处理动作`。程序中`信号捕捉`，需要`提前注册`，告诉应用程序信号产生之后做什么样的处理。

### `signal`

> 可以`捕捉`进程中产生的`信号`，并且`修改捕`捉到的`函数行为`，信号自定义动作时回调函数，内核通过`signal()`得到这个回调函数地址，在信号产生之后会被内核调用，`信号产生之前，提前注册`

```c
#include <signal.h>
sighandler_t signal(int signum,sighandler_t handler);
// sighandler_t 需要捕捉的信号
// handler 信号捕捉到之后的处理动作，函数指针
typedef void(*sighandler_t)(int);
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/time.h>
#include <signal.h>

// 定时器信号的处理动作
void doing(int arg)
{
    printf("当前捕捉到的信号是: %d\n", arg);
    // 打印当前的时间
}

int main()
{
    // 注册要捕捉哪一个信号, 执行什么样的处理动作
    signal(SIGALRM, doing);
    // 1. 调用定时器函数设置定时器函数
    struct itimerval newact;
    // 3s之后发出第一个定时器信号, 之后每隔1s发出一个定时器信号
    newact.it_value.tv_sec = 3;
    newact.it_value.tv_usec = 0;
    newact.it_interval.tv_sec = 1;
    newact.it_interval.tv_usec = 0;
    // 这个函数也不是阻塞函数, 函数调用成功, 倒计时开始
    // 倒计时过程中程序是继续运行的
    setitimer(ITIMER_REAL, &newact, NULL);

    // 编写一个业务处理, 阻止当前进程自己结束, 让当前进程被发出的信号杀死
    while(1)
    {
        sleep(1000000);
    }

    return 0;
}
```

### sigaction

> 同`signal`信号捕捉

```c
#include <signal.h>
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);

struct sigaction {
	void     (*sa_handler)(int);    // 指向一个函数(回调函数)
	void     (*sa_sigaction)(int, siginfo_t *, void *);
	sigset_t   sa_mask;             // 初始化为空即可, 处理函数执行期间不屏蔽任何信号
	int        sa_flags;	        // 0
	void     (*sa_restorer)(void);  //不用
};
//sa_handler:函数指针，指向捕捉到信号的处理动作
//sa_sigaction: 上同
//sa_mask: 在信号处理函数执行期间，临时屏蔽某些信号
//sa_flags:使用那个指针函数 0 使用sa_handler, SA_SIGINFO使用sa_sigaction 使用信号传递数据 == 进程间通信

```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <signal.h>

// 信号的处理动作
void callback(int num)
{
    printf("捕捉信号");
    printf("当前捕捉的信号: %d\n", num);
}

int main()
{
    // 1. 初始化信号集
    sigset_t myset;
    sigemptyset(&myset);
    // 设置阻塞的信号
    sigaddset(&myset, SIGINT);  // 2
    sigaddset(&myset, SIGQUIT); // 3
    sigaddset(&myset, SIGKILL); // 9 测试不能被阻塞

    // 当阻塞的信号被解除阻塞, 该信号就可以被捕捉到了
    // 如果信号被捕捉到之后, 马上就被处理掉了 --> 递达状态
    struct sigaction act;
    act.sa_handler = callback; //回调函数
    act.sa_flags = 0; // 0对应sa_handler
    // 将set集合中所有标志位设置为0
    sigemptyset(&act.sa_mask); //初始化为空，不屏蔽任何信号
    //设置信号捕捉
    sigaction(SIGINT, &act, NULL);  //CTRL +C
    // 和sigint的处理动作相同
    sigaction(SIGQUIT, &act, NULL);
    sigaction(SIGKILL, &act, NULL);

    // 2. 将初始化的信号集中的数据设置给内核
    sigset_t old;
    //使用此函数修改内核中阻塞信号集,
    sigprocmask(SIG_BLOCK, &myset, &old);

    // 3. 让进程一直运行, 在当前进程中产生对应的信号
    int i = 0;
    while(1)
    {
        // 4. 读内核的未决信号集
        sigset_t curset;
        //读取集合中哪个信号是未决状态
        sigpending(&curset);
        // 遍历这个信号集
        for(int i=1; i<32; ++i)
        {
            //判断某个信号在集合中对应的标志位是0还是1，
            int ret = sigismember(&curset, i);
            printf("%d", ret);
        }
        printf("\n");
        sleep(1);
        i++;
        if(i==10)
        {
            // 解除阻塞, 重新设置阻塞信号集
            //sigprocmask(SIG_UNBLOCK, &myset, NULL);
            sigprocmask(SIG_SETMASK, &old, NULL);
        }
    }
    return 0;
}
```

![捕捉信号](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVekpsRPgBL2lUzicqdiaxhcf1OSmKbM60NFzwtCrX9DDLRR0b1dnB8APFgw/640?wx_fmt=png "捕捉信号")

### `SIGCHLD`

> 当子进程`退出,暂停，从暂停恢复运行`，在子进程中会产生一个`SIFGCHLD`信号，并将其发送给`父进程`。父进程可以捕捉子进程`发送过来的信号来回收子进程的资源`。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#include <signal.h>

// 回收子进程处理函数
void recycle(int num)
{
    printf("捕捉到的信号是: %d\n", num);
    // 子进程的资源回收, 非阻塞
    // SIGCHLD信号17号信号, 1-31号信号不支持排队 17子进程中执行完毕
    // 如果这些信号同时产生多个, 最终处理的时候只处理一次
    // 假设多个子进程同时退出, 父进程同时收到了多个sigchld信号
    // 父进程只会处理一次这个信号, 因此当前函数被调用了一次, waitpid被调用一次
    // 相当于只回收了一个子进程, 但是是同时死了多个子进程, 因此就出现了僵尸进程
    // 解决方案: 循环回收即可
    while(1)
    {
        // 如果是阻塞回收, 就回不到另外一个处理逻辑上去了
        pid_t pid = waitpid(-1, NULL, WNOHANG);
        if(pid > 0)
        {
            printf("child died, pid = %d\n", pid);
        }
        else if(pid == 0)
        {
            // 没有死亡的子进程, 直接退出当前循环
            break;
        }
        else if(pid == -1)
        {
            printf("所有子进程都回收完毕了, 拜拜...\n");
            break;
        }
    }
}


int main()
{
    // 设置sigchld信号阻塞
    sigset_t myset;
    sigemptyset(&myset); //将集合中的所有标志位设置为0
    // 将myset集合中所有标志位设置为0
    sigaddset(&myset, SIGCHLD); 
    // SIG_BLOCK 将参数set集合中的数据追加到阻塞信号集中
    sigprocmask(SIG_BLOCK, &myset, NULL); 

    // 循环创建多个子进程 - 20
    pid_t pid;
    for(int i=0; i<20; ++i)
    {
        //循环创建20个进程
        pid = fork();
        if(pid == 0) //子进程退出，父进程继续创建进程
        {
            break;
        }
    }

    if(pid == 0)
    {
        printf("我是子进程, pid = %d\n", getpid());
    }
    else if(pid > 0)
    {
        printf("我是父进程, pid = %d\n", getpid());
        // 注册信号捕捉, 捕捉sigchld
        struct sigaction act;
        act.sa_flags  =0;
        act.sa_handler = recycle;
        sigemptyset(&act.sa_mask);
        // 注册信号捕捉, 委托内核处理将来产生的信号
        // 当信号产生之后, 当前进程优先处理信号, 之前的处理动作会暂停
        // 信号处理完毕之后, 回到原来的暂停的位置继续运行
        sigaction(SIGCHLD, &act, NULL);

        // 解除sigcld信号的阻塞
        // 信号被阻塞之后,就捕捉不到了, 解除阻塞之后才能捕捉到这个信号
        sigprocmask(SIG_UNBLOCK, &myset, NULL);

        // 父进程执行其他业务逻辑就可以了
        // 默认父进程执行这个while循环, 但是信号产生了, 这个执行逻辑或强迫暂停
        // 	父进程去处理信号的处理函数
        while(1)
        {
            sleep(100);
        }
    }
    return 0;
}
```

![检测子进程](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuAZEWbsPKptycVwDzZkVek3scegkoZ6EFD8W2l1QLLYvxGUIKz8fe0UdmamKs8kbQ1gRCXPsjQuQ/640?wx_fmt=png "检测子进程")