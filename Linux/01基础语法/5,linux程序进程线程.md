程序：（静态的概念）源代码，指令

进程：运行着的程序，一个程序可以创建多个进程

线程：线程从属于进程，一个进程可以有多个线程，线程之间共享进程的资源

任务：具体要做的事情（进程/线程）

#### 获取进程ID


```sh

//linux 编程环境
man getpid //得到linux帮助文档，Linux的编程手册
#include <sys/types.h>
#include <unistd.h>
pid_t getpid(void)
pid_t getppid(void)
//pid_t类型用于存放进程的ID号

getpid() //返回当前正在运行的进程ID号
getppid()  //返回当前进程额父进程的ID号
```

```sh
su    //进入管理权限
cd /     //进入根目录
mkdir C_test  //创建C_test文件夹
cd C_test    //进入此文件夹
nano pid.c   //创建pid.c的文件
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs5CI8nJRCM8iasJSRBKkWayRZWiaOa5a4kz7iaNypnib8uGED9oRCVu2hkbOibXntOB4TnogiadRJHjFtA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```sh
//写完代码后输入  ctrl+x   保存并退出
//首次使用Linux环境编译，错误多多，完全没有任何提升和自动语句
gcc pid.c  //编译运行gid.c文件,生成执行文件名a.out
./a.out     //执行pid.c得出执行结果 ctrl+z 停止运行
man pstree  //获得关于pstree 的帮助文档
pstree -p  //得出进程树，谁创建的谁，查看进程树
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs5CI8nJRCM8iasJSRBKkWayic1KZs8iapu8Gfu1TPTr8ybic8ricpaPNyFQ6hHc2UHDOMOpuBn2dWPYiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 创建进程

```sh
man fork   //查看fork的Linux开发手
```

#### fork函数创建一个孩子进程

> 需要引用的库


```sh
#include <sys/types.h>
#include <unistd.h>
#include <unistd.h>
pid_t fork(void);  //创建进程通过复制调用进程，新创建的进程称为子进程
//子进程和父进程运行在不同的区间，相互独立，但父进程和子进程有相同的内容
```
运行：
```
nano 2_fork_1.c    //创键c文件
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv3kia1l6wwtmbQETqNgJibrYf3ULiaE93LNntbML2oje60fz3ettmh1RMYibmiaWtrajzOf6KY96IvRTg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
gcc 2_fork_1.c./a.out
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv3kia1l6wwtmbQETqNgJibrYTnjpMnhWCdOUxDw6potWF2iaIibia1xVUnlPsAzW9cwnDeWNzFZrhn1bw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

输出两个Hello word!

fork函数将A设为父进程，创建子进程B，两者内容相同都会输出hello word

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvBOZfBmtPlIkD1LnnlTiawunLqDwoPAkX24vP17ZbCgzI0IicVAbuVtgOeNfhZBzt4cIl9ImWXibztA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvBOZfBmtPlIkD1LnnlTiawuh5kjDGpeagbOKabRDPcAyDR3OdVLMe64qDW0MT3Bv7H4Zv4qFXGLsQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

父进程返回子进程的id号，子进程的pid为零

失败的话，返回-1，

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvBOZfBmtPlIkD1LnnlTiawu6tVkYZ2Bkmfs3JHibhEtpfKiaLib3K4qToLudNywRsuCQ5aZlI1JuYdsw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvBOZfBmtPlIkD1LnnlTiawuCl6EH49AyKncf1ocJticxl9OlVibBQibKwWMuZX4HTnaquk6BdbLfAibicA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


```sh

cp 2_fork_1.c fork_2.c   //复制2_fork_1.c的代码到fork_2.c

nano for_2.c

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
        pid_t pid;
        pid = fork();
        printf("pid=%d\n",pid);
        if(pid > 0)
        {
                while(1)
                {
                        printf("hello world\n");
                        sleep(1);
                }
        }
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
        pid_t pid;
        pid = fork();
        printf("pid=%d\n",pid);
        if(pid > 0)
        {
                while(1)
                {
                        printf("hello world\n");
                        sleep(1);
                }
        }
       else if(pid == 0)
            {
                    while(1)
                    {
                            printf("Good morning\n");
                            sleep(1);
                    }
            }
        return 0;
}
//输出结果：交替运行
//宏观上子进程父进程并发执行，微观上父进程先调度
```
```sh

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
        pid_t pid;
        int count =0;
        pid = fork();
        printf("pid=%d\n",pid);
        if(pid > 0)
        {
                while(1)
                {
                        printf("hello world count = %d\n",count++);
                        sleep(1);
                }
        }
        else if(pid == 0)
        {
                while(1)
                {
                        printf("Good morning count = %d\n",count++);
                        sleep(1);
                }
        }
        return 0;
}

```
> 父进程和子进程内存是相互独立的。

> 父进程和子进程的运行没有相互依赖的关系

#### 监控子进程wait函数

```sh
man wait  //等待任意子进程终止，会有返回值，返回那个进程终止，返回其ID号
//引用库
#include <sys/types.h>
#include <unistd.h>

pid_t wait(int *wstatus)  //地址变量,不希望用就设置为空指针
```
> 创建三个子进程，五秒钟，10秒钟，15秒结束，父进程等所有子进程结束再结束
```sh

nano  5_fork.c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>

//run model:./a.out 10 5 15 (there child process ,after 15s they are over
int main(int argc,char *argv[])
{
        pid_t child_pid;
        int numDead;
        int i;
        for(i =1 ; i < argc; i++)
        {
                switch(fork())
                {
                        case -1: 
                                perror("fork()");
                                exit(0);
                       case 0:
                                printf("child %d started $ = %d, sleeping %s seconds\n", i , getpid(),argv[i]);
                                sleep(atoi(argv[i]));
                                exit(0);
                        default:
                                break;
                }
        }
        numDead = 0;
        while(1)//当前有几个子进程结束了
        {
                child_pid = wait(NULL);  //返回子进程结束的ID号
                if(child_pid==-1)
                {
                        printf("No morre children,Byebye Byebye\n");
                        exit(0);
                }

                numDead++;
                printf("wait() returned child  PID : %d(numDead = %d)\n", child_pid,numDead );
        }
}


//指向
gcc 5_fork.c
./a.out 10 5 15   //运行参数
```
![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvBOZfBmtPlIkD1LnnlTiawudbwh0nficOkWuDDpajFjbq3Yia0r46ziaxNR6z8ica3DyeS9PARcbBiaEMQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 线程函数pthread_create


```sh

//引入库函数
#include <pthread.h>
//创建新的线程
int pthread_create(pthread_t *thread, const pthread_attr_t,void *(*start_routne) (void*),void *arq);)
//第一个参数线程Id号，第二个参数线程结构体指针类型空就是NULL
//第三个函数指针，第四个参数，也可为NULL
//成功返回值为0，错误返回错误数字
//编译的时候末位要加  -pthread
```
#### 等待线程结束

```sh
int pthread_join(pthread_t thread,void **retval);
//编译并且链接给 pthread
//调用成功返回零
//第一个为线程参数。等待的线程
//第二个为
```

编写：
```sh

#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>
void *thread_function(void *arg);
int main(void)
{
        pthread_t pthread;
        int ret;
        int count=5;//传递子针，传递给线程arg
        //线程用到所有资源都依赖于进程，
        ret = pthread_create(&pthread,NULL,*thread_function,&count);
        if(ret != 0)
        {
                perror("pthread_create");
                exit(1);
        }
        //while(1);//让进程永远不结束，否则线程资源也会被回收
        //使用函数等线程来结束 pthread_join，才会接着运行
        pthread_join(pthread,NULL);
        printf("The thread is over ,process is over too.\n");
        return  0;
}
void *thread_function(void *arg)  //arg为count指针
{       //线程每隔几秒打印
        int i;
        printf("Thread begins running\n");
        for(i = 0; i <= *(int*)arg; i++)
        {
                printf("Hello world\n");
                sleep(1);
        }
}

//编译:
gcc 6_thread.c -pthread  //编译
./a.out  //运行
```

#### 创建两个线程/多个线程

```sh
//创建多个线程，多次调用pthread即可
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>
void *thread1_function(void *arg);
void *thread2_function(void *arg);
int count =0;
int main(void)
{
        pthread_t pthread2,pthread1;
        int ret;
        //int count=5;//传递子针，传递给线程arg
        //线程用到所有资源都依赖于进程，
        ret = pthread_create(&pthread1,NULL,*thread1_function,NULL);

        if(ret != 0)
        {
                perror("pthread1_create");
                exit(1);
        }
        //创建第二个线程
        ret = pthread_create(&pthread2,NULL,thread2_function,NULL);
        if(ret !=0)
        {
                perror("pthread_create");
                exit(1);
        }

        //while(1);//让进程永远不结束，否则线程资源也会被回收
        //使用函数等线程来结束 pthread_join，才会接着运行
        pthread_join(pthread1,NULL);
        pthread_join(pthread2,NULL);
        printf("The thread is over ,process is over too.\n");
        return  0;
}
void *thread1_function(void *arg)  //arg为count指针
{       //线程每隔几秒打印
        int i;
        printf("Thread begins running\n");
        //for(i = 0; i <= *(int*)arg; i++)
        while(1)
        {
                printf("Hello world count = %d\n",count++);
                sleep(1);
        }
}
void *thread2_function(void *arg)
{
        printf("Thread2 begins runnings\n");
        while(1)
        {
                printf("Good morning count = %d\n",count++);
                sleep(1);
        }
}

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv3lprxuaVmgdia0Lo9JSzQltLHM6ojyZV6FxkQOms5shg8r9qeF1xM0Py5dZNoNwBR0WWgibBp9t5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> 总：父子进程分别打印count，父进程和子进程会分别独立打印，独立的全局count值

  进程和线程不同，会使用同一个count值是累加的

补：

```sh
exit()函数//里面参数为0表示正常退出，为1/-1表示程序异常退出
```

> 参考：“嵌入式开发” Linux简明教程01～06