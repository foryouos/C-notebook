# Linux多线程

> 线程时轻量级的进程(`LWP: light weight process`)，在`linux`环境下线程的本质仍是`进程`，进程时资源分配的`最小单位`，线程时操作系统调度执行的`最小单位`
>
> 上下文切换：进程/线程`分时复用CPU时间片`，在切换之前会将上一个任务的状态进行保存，下次切换回这个任务的时候，加载这个状态继续运行，任务从保存到再次加载这个进程就是一次`上下文切换`，线程更加廉价，启动速度更快，退出也快，对系统资源`冲击小`。线程并不是越多越好，
>
> 文件IO操作 ： 线程的个数 = `2* CPU核心数`(效率最高)
>
> 处理`复杂算法`（主要CPU进行运算，压力大) `线程个数 =CPU核心数`
>
> 多线程的编程功力直接决定服务器性能的优异
> 充分利用内核：使用到所有的内核，内核不空闲

## `POSIX`多线程

> 需要依赖头文件`pthread.h` 在编译的时候要加上库`pthread`表示包含多线程库文件
![常见的API函数](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbshWwiapqz7vzD3cbGKibtuwOReBOO82NbC1FtNBCqsswFqef8Q9ugZEPIuKRL6u0Q5IDsYCNP7PcQg/640?wx_fmt=jpeg "常见的API函数")

### 线程ID

> 每个线程都有唯一的`线程ID`，ID类型为`pthread_t`

```cpp
//返回当前线程的线程ID
pthread_t pthread_self(void); 
```

### 线程的创建

```cpp
#include <pthread.h>
pthread_t tid;
int pthread_create(pthread_t *pid,const pthread_attr_t *attr,void *(*start_routine)(void *),void *arg)
//pid为线程创建后的id
//attr为指向线程属性结构的pthread_attr_t的指针，NULL使用默认属性
//start_routine指向线程函数的地址，要执行的函数
//arg指向线程函数的参数
//如果成功，函数返回0
```
### `pthread_join`

> 函数`pthread_join`来`等待子线程结束`，函数会让主线程挂起(即休眠，`让出CPU`)，指导子线程退出，同时`pthread_join`能让子线程所占资源也释放
```cpp
int pthread_join(pthread_t pid,void **value_ptr);
//pid是所等待线程的ID，value_ptr通常设为NULL。
//如果不为NULL，pthread_join复制一份线程退出值到一个内存区域，因此pthread_join还有一个重要功能就是获得子线程的返回值。如果函数成功，返回0,否则返回错误码
```

### 创建线程(不传参数)

```cpp
#include <pthread.h>
#include <stdio.h>
#include <unistd.h> //sleep
void* thfunc(void* arg)
{
    printf("in thfunc\n");
    return (void*)0;
}

int main(int argc,char *argv[])
{
    pthread_t tidp;
    int ret;
    ret = pthread_create(&tidp, NULL, thfunc, NULL); //创建线程 ,如果成功返回0
    if (ret) //0不会 执行
    {
        printf("pthread_create fail:%d\n", ret);
        return -1;
    }
    sleep(1); //main线程挂起 一秒钟
    printf("in main:thread is created!\n");
    return 0;
}
```
### 线程退出
> 调用线程退出函数，当前线程会马上退出，不会影响其他线程的正常运行，不管是在子线程和主线程都可以使用
```cpp
#include <pthread.h>
void pthread_exit(void* retval);
//参数:线程退出时携带的数据，当前子线程会得到该数据，若不需要指定为NULL
```

### 回收子线程数据
> 1，在子线程退出的时候可以使用`pthread_exit()`的参数将数据传出，在回收这个子线程的时候通过`pthread_join()`的第二个参数来接受子线程传递出的数据，
> 2，使用全局变量,在主函数上定义`struct Persion p;`	// 定义全局变量
> 3，使用传给子线程参数的形式，在结束时，再传递给主线程的形式
```cpp
// pthread_join.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 定义结构
struct Persion
{
    int id;
    char name[36];
    int age;
};

// 子线程的处理代码
void* working(void* arg)
{
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        printf("child == i: = %d\n", i);
        if(i == 6)
        {
            struct Persion p;
            p.age  =12;
            strcpy(p.name, "tom");
            p.id = 100;
            // 该函数的参数将这个地址传递给了主线程的pthread_join()
            pthread_exit(&p);
        }
    }
    return NULL;	// 代码执行不到这个位置就退出了
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;
    pthread_create(&tid, NULL, working, NULL);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 阻塞等待子线程退出
    void* ptr = NULL;
    // ptr是一个传出参数, 在函数内部让这个指针指向一块有效内存
    // 这个内存地址就是pthread_exit() 参数指向的内存
    pthread_join(tid, &ptr);
    // 打印信息
    struct Persion* pp = (struct Persion*)ptr;
    printf("子线程返回数据: name: %s, age: %d, id: %d\n", pp->name, pp->age, pp->id);
    printf("子线程资源被成功回收...\n");
    
    return 0;
}
```
### 线程分离
> 实现子线程和主线程分离，当子线程退出的时候，其占用的内核资源就被系统的其他进程接管并回收。
```cpp
// 参数就子线程的线程ID, 主线程就可以和这个子线程分离了
int pthread_detach(pthread_t thread);
```

### 线程取消
> 线程取消，主线程调用取消函数`pthread_Cancel`指定杀死线程，线程B需要进行一次系统调用(从用户区切换到内核去，否则子线程会一直运行 `hread_self()`是一次`接的系统调用`
```cpp
#include <pthread.h>
// 参数是子线程的线程ID
int pthread_cancel(pthread_t thread);
```

### 线程ID比较

> 在Linux中线程ID时一个无符号长整型，
```cpp
#include <pthread.h>
//t1,t2要比较的线程ID，如相等返回非0，如果不相等返回0
int pthread_equal(pthread_t t1, pthread_t t2);
```

## 线程同步

> CPU对应寄存器、一级缓存、二级缓存、三级缓存是独占的，用于存储处理的数据和线程的状态信息，数据被 CPU 处理完成需要再次被写入到物理内存中，物理内存数据也可以通过文件 IO 操作写入到磁盘中。
> `共享资源`就是多个线程共同访问的变量，这些变量通常为全局数据区变量或者堆区变量，这些变量对应的共享资源也被称为`临界资源`
> 找到临界资源，确认临界区的上下文代码，临界区越小越好，之后进行进程同步，临界区上面加锁，下面解锁。

###  互斥锁
> 被锁定的代码块顺序执行(不能并行处理)，执行效率低
> 一个互斥锁变量智能被一个线程锁定，被锁定之后其他线程再对互斥锁变量加锁就会阻塞，直到互斥锁被解锁。

```cpp
//互斥锁类型
pthread_mutex_t  mutex;
// 初始化互斥锁
// restrict: 是一个关键字, 用来修饰指针, 只有这个关键字修饰的指针可以访问指向的内存地址, 其他指针是不行的，attr一般默认属性，指定为NULL
//mutex 为互斥锁变量的地址
int pthread_mutex_init(pthread_mutex_t *restrict mutex,
           const pthread_mutexattr_t *restrict attr);
// 释放互斥锁资源            
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```
#### 尝试加锁
```cpp
// 尝试加锁
int pthread_mutex_trylock(pthread_mutex_t *mutex);
//如果没有被锁，线程加锁成功
//如果被锁住，调用此函数，不会被阻塞，加锁失败返回错误信号
// 对互斥锁解锁
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
#### 死锁
* 加锁忘记解锁
* 重复加锁
* 多个共享资源，注意相互阻塞

### 读写锁
> 读写锁为互斥锁的升级版，在做读操作的时候可以提高程序的执行效率，如果所有线程都是`读`的可以`并行`。`写锁死独占的`，同时`写锁`的`优先级`更高

```cpp
//创建读写锁类型
pthread_rwlock_t rwlock;
// 初始化读写锁
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
           const pthread_rwlockattr_t *restrict attr);
// 释放读写锁占用的系统资源
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
```
#### 锁
```cpp
// 在程序中对读写锁加读锁, 锁定的是读操作
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
// 这个函数可以有效的避免死锁
// 如果加读锁失败, 不会阻塞当前线程, 直接返回错误号
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

// 在程序中对读写锁加写锁, 锁定的是写操作
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
// 这个函数可以有效的避免死锁
// 如果加写锁失败, 不会阻塞当前线程, 直接返回错误号
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);

// 解锁, 不管锁定了读还是写都可用解锁
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

### 条件变量

> 条件变量配合互斥锁实现线程同步
>
> 条件变量只有在满足指定条件下才会阻塞线程，如果天剑不满足，多个线程可以同时进入临界区，同时读写临界资源。

```cpp
//条件变量类型
//被条件比那辆阻塞的线程的线程信息会被记录在这个变量中，
pthread_cond_t cond;
//条件变量的初始化与销毁释放资源
#include <pthread.h>
pthread_cond_t cond;
//初始化
int pthread_cond_init(pthread_cond_t *restrict cond,cond pthread_condattr_t *restrict  attr);
//销毁释放资源
int pthread_cond_destory(pthread_cond_t *cond);
//cond条件变量的地址
//attr：条件变量的属性，一般使用默认属性，执行为NULL

// 线程阻塞函数, 哪个线程调用这个函数, 哪个线程就会被阻塞
//参数：条件变量，互斥锁
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
// 互斥锁主要功能是线程同步，让线程顺序进入临界区，避免出现共享资源的数据混乱，
//在阻塞线程的时候，如果线程以及对互斥锁mutex上说，那么将把锁打开，当线程解除阻塞，函数内部会帮助mutex锁上，继续向下访问临界区
// 表示的时间是从1971.1.1到某个时间点的时间, 总长度使用秒/纳秒表示
struct timespec {
	time_t tv_sec;      /* Seconds */
	long   tv_nsec;     /* Nanoseconds [0 .. 999999999] */
};
// 将线程阻塞一定的时间长度, 时间到达之后, 线程就解除阻塞了
int pthread_cond_timedwait(pthread_cond_t *restrict cond,
           pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);
           
           
time_t mytim = time(NULL);	// 1970.1.1 0:0:0 到当前的总秒数
struct timespec tmsp;
tmsp.tv_nsec = 0;
tmsp.tv_sec = time(NULL) + 100;	// 线程阻塞100s
// 唤醒阻塞在条件变量上的线程, 至少有一个被解除阻塞
int pthread_cond_signal(pthread_cond_t *cond);
// 唤醒阻塞在条件变量上的线程, 被阻塞的线程全部解除阻塞
int pthread_cond_broadcast(pthread_cond_t *cond);

```


#### 条件变量实现生产者与消费者
![条件变量](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbshWwiapqz7vzD3cbGKibtuwOs6KyaQ2btRX3T6kALPB6dau2WzS4hQ0lIzLsDg8u9h6c6ricNYtwltg/640?wx_fmt=png "生产者与消费者问题")
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 链表的节点
struct Node
{
    int number;
    struct Node* next;
};

// 定义条件变量, 控制消费者线程
pthread_cond_t cond;
// 互斥锁变量
pthread_mutex_t mutex;
// 指向头结点的指针
struct Node * head = NULL;

// 生产者的回调函数
void* producer(void* arg)
{
    // 一直生产
    while(1)
    {
        //对生产者上锁
        pthread_mutex_lock(&mutex);
        // 创建一个链表的新节点
        struct Node* pnew = (struct Node*)malloc(sizeof(struct Node));
        // 节点初始化
        pnew->number = rand() % 1000;
        // 节点的连接, 添加到链表的头部, 新节点就新的头结点
        pnew->next = head;
        // head指针前移
        head = pnew;
        //输出当前线程
        printf("+++producer, number = %d, tid = %ld\n", pnew->number, pthread_self());
        //解锁
        pthread_mutex_unlock(&mutex);

        // 生产了任务, 通知消费者消费
        pthread_cond_broadcast(&cond); //通知消费者消费

        // 生产慢一点
        sleep(rand() % 3);
    }
    return NULL;
}

// 消费者的回调函数
void* consumer(void* arg)
{
    while(1)
    {
        //消费者上锁
        pthread_mutex_lock(&mutex);
        // 一直消费, 删除链表中的一个节点
//        if(head == NULL)   // 这样写有bug
        while(head == NULL) //没有数据，阻塞在此
        {
            // 任务队列, 也就是链表中已经没有节点可以消费了
            // 消费者线程需要阻塞
            // 线程加互斥锁成功, 但是线程阻塞在这行代码上, 锁还没解开
            // 其他线程在访问这把锁的时候也会阻塞, 生产者也会阻塞 ==> 死锁
            // 这函数会自动将线程拥有的锁解开
            pthread_cond_wait(&cond, &mutex); //会将mutex解锁
            // 当消费者线程解除阻塞之后, 会自动将这把锁锁上
            // 这时候当前这个线程又重新拥有了这把互斥锁
        }
        // 取出链表的头结点, 将其删除
        struct Node* pnode = head;
        printf("--consumer: number: %d, tid = %ld\n", pnode->number, pthread_self());
        head  = pnode->next; //重新指向头结点
        free(pnode);//释放表头
        pthread_mutex_unlock(&mutex);       //对互斥锁进行解锁  

        sleep(rand() % 3);
    }
    return NULL;
}

int main()
{
    // 初始化条件变量
    pthread_cond_init(&cond, NULL);
    pthread_mutex_init(&mutex, NULL);

    //单词仍然是一个生产者，消费者会被阻塞到条件变量，随机去抢生成者生产数据资源
    // 创建5个生产者, 5个消费者
    pthread_t ptid[5];
    pthread_t ctid[5];
    for(int i=0; i<5; ++i)
    {
        pthread_create(&ptid[i], NULL, producer, NULL);
    }

    for(int i=0; i<5; ++i)
    {
        pthread_create(&ctid[i], NULL, consumer, NULL);
    }

    // 释放资源
    for(int i=0; i<5; ++i)
    {
        // 阻塞等待子线程退出
        pthread_join(ptid[i], NULL);
    }

    for(int i=0; i<5; ++i)
    {
        pthread_join(ctid[i], NULL);
    }

    // 销毁条件变量
    pthread_cond_destroy(&cond);
    pthread_mutex_destroy(&mutex);

    return 0;
}
```
###  信号量
> 信号量用在多线程多任务同步时，一个线程完成了某一个动作就通过信号量告诉别的线程，别的线程再进行某些动作。
> 信号量(灯)，灯亮资源可用，灯灭则意味着不可用。信号量主要阻塞线程，不能完全保证线程安全，需要与互斥锁一起使用

```cpp
#include <semaphore.h>
sem_t sem;
//初始化信号量
int sem_init(sem_t *sem, int pshared, unsigned int value);
// 资源释放, 线程销毁之后调用这个函数即可
// 参数 sem 就是 sem_init() 的第一个参数    
//pshared: 0:线程同步，非0:进程同步
//value:初始化当前信号量拥有的资源数(>=0),如果资源数为0,线程就会被阻塞
int sem_destroy(sem_t *sem);

//函数被调用sem中的资源就会被消耗1个，资源数-1
//如果sem资源数 > 0则不会阻塞
int sem_wait(sem_t *sem);

// 函数被调用sem中的资源就会被消耗1个, 资源数-1
//当sem中的资源数减为0时，资源好近，线程不会阻塞，直接返回错误信号
int sem_trywait(sem_t *sem);

// 表示的时间是从1971.1.1到某个时间点的时间, 总长度使用秒/纳秒表示
struct timespec {
	time_t tv_sec;      /* Seconds */
	long   tv_nsec;     /* Nanoseconds [0 .. 999999999] */
};
// 调用该函数线程获取sem中的一个资源，当资源数为0时，线程阻塞，在阻塞abs_timeout对应的时长之后，解除阻塞。
// abs_timeout: 阻塞的时间长度, 单位是s, 是从1970.1.1开始计算的
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
//abs_timeout 和 pthread_cond_timedwait 的最后一个参数是一样的

// 调用该函数给sem中的资源数+1
int sem_post(sem_t *sem);
//调用之后sem_wait,sem_trywait,sem_timedwait这些线程会解除阻塞，获得资源之后继续向下运行
// 查看信号量 sem 中的整形数的当前值, 这个值会被写入到sval指针对应的内存中
// sval是一个传出参数
int sem_getvalue(sem_t *sem, int *sval);

```


#### 生产者与消费者案例

![信号量](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbshWwiapqz7vzD3cbGKibtuwOQuXflL98ZV6larh3ic8Qp65g2kCNx0FEFwtZv0VvFIcLgzKwDHrDHxA/640?wx_fmt=png "信号量生产者与消费者问题")

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <semaphore.h>
#include <pthread.h>

// 链表的节点
struct Node
{
    int number;
    struct Node* next;
};

// 生产者线程信号量
sem_t psem;
// 消费者线程信号量
sem_t csem;

// 互斥锁变量
pthread_mutex_t mutex;
// 指向头结点的指针
struct Node * head = NULL;

// 生产者的回调函数
void* producer(void* arg)
{
    // 一直生产
    while(1)
    {
        // 生产者拿一个信号灯
        sem_wait(&psem);
        // 加锁, 这句代码放到 sem_wait()上边, 有可能会造成死锁
        pthread_mutex_lock(&mutex);
        // 创建一个链表的新节点
        struct Node* pnew = (struct Node*)malloc(sizeof(struct Node));
        // 节点初始化
        pnew->number = rand() % 1000;
        // 节点的连接, 添加到链表的头部, 新节点就新的头结点
        pnew->next = head;
        // head指针前移
        head = pnew;
        printf("+++producer, number = %d, tid = %ld\n", pnew->number, pthread_self());
        pthread_mutex_unlock(&mutex);

        // 通知消费者消费
        sem_post(&csem);
        
        // 生产慢一点
        sleep(rand() % 3);
    }
    return NULL;
}

// 消费者的回调函数
void* consumer(void* arg)
{
    while(1)
    {
        sem_wait(&csem);
        pthread_mutex_lock(&mutex);
        struct Node* pnode = head;
        printf("--consumer: number: %d, tid = %ld\n", pnode->number, pthread_self());
        head  = pnode->next;
        // 取出链表的头结点, 将其删除
        free(pnode);
        pthread_mutex_unlock(&mutex);
        // 通知生产者生成, 给生产者加信号灯
        sem_post(&psem);

        sleep(rand() % 3);
    }
    return NULL;
}

int main()
{
    // 初始化信号量
    sem_init(&psem, 0, 5);  // 生成者线程一共有5个信号灯
    sem_init(&csem, 0, 0);  // 消费者线程一共有0个信号灯
    // 初始化互斥锁
    pthread_mutex_init(&mutex, NULL);

    // 创建5个生产者, 5个消费者
    pthread_t ptid[5];
    pthread_t ctid[5];
    for(int i=0; i<5; ++i)
    {
        pthread_create(&ptid[i], NULL, producer, NULL);
    }

    for(int i=0; i<5; ++i)
    {
        pthread_create(&ctid[i], NULL, consumer, NULL);
    }

    // 释放资源
    for(int i=0; i<5; ++i)
    {
        pthread_join(ptid[i], NULL);
    }

    for(int i=0; i<5; ++i)
    {
        pthread_join(ctid[i], NULL);
    }

    sem_destroy(&psem);
    sem_destroy(&csem);
    pthread_mutex_destroy(&mutex);

    return 0;
}
```
