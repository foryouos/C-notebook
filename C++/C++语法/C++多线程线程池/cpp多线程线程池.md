# 多线程

> C++ 11提供的线程类std::thread 
>
> * thread
>
> * mutex
>
> * condition_variable
>
> * atomix
>
> * future
>
>   ![多线程](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvXJRJqUjeibMeWIa6MRMpOpCx2vvZvK2Odty70EUcZm0RwySS6eunTGBYStZOmhDWcTkoMr8TZblg/640?wx_fmt=png "多线程")

## 构造函数
```cpp
//默认构造函数对象，不进行任何操作
thread() noexcept;
// 将other的线程所有权转移给新的thread对象，之后other不再表示执行线程
//thread t1(func,ref(n)) //ref(n)取n的引用
//other可以使用move(t1)，调用之后t1不再运行func函数，留给新的线程执行
thread( thread&& other ) noexcept;
// f 执行函数，args,传给函数f的餐护士，可多个
template< class Function, class... Args >
explicit thread( Function&& f, Args&&... args );
// 构造函数 =delete 显示删除拷贝构造，不允许线程对象之间拷贝
thread( const thread& ) = delete;
```
## 公共成员函数
* `get_id()` : 获取当前线程`id`
* `join()`  函数`所在线程`等待加入的`线程结束` ，子线程未结束，主线程阻塞等待
* `detach()` : 主线程与子线程`脱离`，`不等待`，子线程执行完毕后会`自行释放`系统资源
* `bool joinable()`：判断主线程与子线程`是否关联`
	* 创建线程并没有指定函数，子线程不会启动，主线程和子线程也不会连接
	* 若指定了函数，子线程启动，主线程和子线程自动连接
	* 若调用`detach函数`之后，两者分离，不再连接
	* 调用`join() 函数`之后，调用也会`返回false`, 子线程任务完毕，jion会清理当前子线程相关资源，子线程和主线程也就断开`，不再连接`
* `线程`中的`资源是不能复制`的，不可用 `=` 重载
* `thread::hardware_concurrency()` 获取当前计算机的`CPU核心数`，返回`int`

### 示例

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;
void func(int num, string str)
{
	for (int i = 0; i < 10; i++)
	{
		cout << "子线程：i = " << i << ", num: " << num << ", str :" << str << endl;
		//当前线程阻塞500ms
		this_thread::sleep_for(chrono::microseconds(500));
	}
	
}
void func1()
{
	for (int j = 0; j < 10; j++)
	{
		cout << "子线程：j = " << j << endl;
		//当前线程阻塞500ms
		this_thread::sleep_for(chrono::microseconds(500));
	}
}
int main(void)
{
	//获取当前CPU核心数，
	//每个线程独自占有一个CPU核心，这些线程就不用分时复用CPU时间片，此时层序并发效率最高
	cout << "当前计算机的CPU核心数" << thread::hardware_concurrency() << endl;

	cout << "主线程id :" << this_thread::get_id() << endl; //获取线程id
	//利用构造函数，创建现车个t和t1
	//同时利用参数将子线程函数与参数传入
	thread t(func, 520, "I Love You");
	thread t1(func1);
	// 输出子线程id
	cout << "子线程 t ID ： " << t.get_id() << endl;;
	cout << "子线程 t1 ID ： " << t1.get_id() <<endl;
	//主线程先执行完毕，
	
#if 0
	//1,等到子线程结束。将子线程加入到当前线程当中
	t.join();
	t1.join();
#else if 1
	//2，子线程与主线程分离，主线程运行完毕，释放空间，子线程可以脱离主线程继续运行，任务运行完毕之后，子线层会自动释放自己占用的系统资源
	t.detach();
	t1.detach();
#endif // 0
	cout << "主线程结束!!" << endl;
	//主线程等待子线程运行结束，等待5秒
	this_thread::sleep_for(chrono::microseconds(5000));
	//来不及运行的子程序将无法将输出结果呈现到主屏幕上
	return 0;
}
```

## `sleep_for()`

> 调用该线程会马上从`运行态变成阻塞态`，并在这种状态下休眠一定的时间，`让出CPU资源`

```cpp
//Rep数值类型，表示时钟树类型
// Period表示始终的周期
template <class Rep, class Period>
//duration 表示一段时间间隔,

void sleep_for (const chrono::duration<Rep,Period>& rel_time);
//当前线程休息一秒钟
this_thread::sleep_for(chrono::seconds(1));
//秒： std::chrono::seconds
//分钟：std::chrono::minutes
// 毫秒：std::chrono::milliseconds
```
## ``sleep_until() `

> 指定线程阻塞到一个`指定的时间点`time_point类型，`之后解除阻塞`

## `yield()`

> 调用yield之后会主`动放弃CPU资源`，变为就绪态，但是会`马上`加入`下`一轮`CPU`的资源`抢夺`。
> ·`this_thread::yield()`

## 线程同步

![线程同步](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvXJRJqUjeibMeWIa6MRMpOpUrBkJatbLhCOg8ZgWQ9X00ZeKiaywFO05iaJav4ZmVOebOsgRdkKesSg/640?wx_fmt=jpeg "线程同步")

![线程池](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvXJRJqUjeibMeWIa6MRMpOpXbibs6wph6tBu8NuyxfA0LIgwiaNkgicWZPsOA90gXdSFyAA8I4s9MJ6g/640?wx_fmt=png "线程池")

```cpp
//全局创建互斥锁
mutex mtx;
//加锁
mtx.lock();
//解锁
mtx.unlocl();
```

## 线程池

> 指定数量创建好的线程，当需要用时，从线程中取出，如果线程不够，管理者线程将创建线程，若空闲数量过多，将适量销毁不使用的线程。
> 避免创建和销毁线程锁需要的开销


## 原理

* 任务队列: 需要处理的任务 -> 生产者线程

* 工作的线程：任务队列的消费者 -> 读线程

* 管理者线程： 1个  任务多，创建线程，任务少，适当销毁线程
> 在线程池初始化过程中，定义一个管理者线程，定义最小初始化的工作线程，工作线程复制运行函数，
> 任务队列类，负责封装任务函数指针与参数，在线程池中存入到队列当中，在添加任务时不断更新头结点和尾节点位置。
> 管理者线程 如果线程池没有关闭就一直检测线程，读取当前任务个数与存活的线程数，如果任务过多，线程过于紧张，向线程池队列中添加线程。如果运行的线程的二倍小于存活的线程数目
> 工作线程，一直不停地工作。每次循环，若任务队列为空，工作线程阻塞，如线程池被关闭了，退出当前工作线程。执行函数，执行函数之后，降低忙碌的线程

### C `线程池`
#### 头文件
```c
#ifndef _THREADPOOL_H
#define _THREADPOOL_H
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#define NUMBER 3
//任务队列
typedef struct Task
{
    void (*function)(void* arg);  //任务队列存储函数指针
    void* arg;  //存储函数的参数指针
}Task;
//线程池结构体
typedef struct ThreadPool
{
    Task* taskQ; //任务队列(通过申请空间，乘以容量，即可成为任务队列)
    int queueCapacity; //任务队列容量
    int queueSize;  //当前任务个数
    int queueFront;  //队头取数据
    int queueRear;   //队尾  放数据

    pthread_t managerID; //管理者线程ID 仅一个
    pthread_t *threadIDs; //工作的线程ID，同样申请多个
    int minNum;  //最小线程数量
    int maxNum;   //最大小城数量
    int busyNum;  //忙的线程个数 运行中
    int liveNum;  //存活的线程个数
    int exitNum;  //要销毁的线程个数
    pthread_mutex_t mutexPool; //锁整个的线程池
    pthread_mutex_t mutexBusy; //锁busyNum变量
    pthread_cond_t notFull;   //任务队列是不是满了
    pthread_cond_t notEmpty; //任务队列是不是空

    int shutdown;  //是不是要销毁线程池，销毁为1，不销毁为0

}ThreadPool;

//创建线程并初始化,传入参数最小线程数量，最大线程数量，和队列容量
ThreadPool *threadPoolCreate(int min,int max,int queueSize);
// 该线程添加任务
// 传入参数线程结构体，执行任务的函数指针,函数参数
void threadPoolAdd(ThreadPool* pool,void(*func)(void*),void* arg);

//获取当前线程池中工作的线程个数
int threadPoolBusyNum(ThreadPool *pool);

//获取当前线程池中活着的线程个数
int threadPoolAliveNum(ThreadPool* pool);

//工作的线程(消费者线程)任务函数
void* worker(void*arg);
//管理者线程任务函数
void * manager(void* arg);

//单个线程退出
void threadExit(ThreadPool* pool);
//销毁线程池  传入线程pool
int threadPoolDestroy(ThreadPool* pool);
//线程池所有线程和任务皆为空
int threadPoolISEmpty(ThreadPool* pool);
#endif
```
#### 头文件实现
```c
#include "threadpool.h"

ThreadPool* threadPoolCreate(int min,int max,int queueSize)
{
    //初始化结构体空间
    ThreadPool* pool = (ThreadPool*)malloc(sizeof(ThreadPool));

    do
    {
        //判断申请空间是否成功
        if(pool == NULL)
        {
            printf("malloc threadPool fail...\n");
            break;
        }
        //申请工作线程ID攻坚
        //申请最大空间的
        pool->threadIDs =(pthread_t*)malloc(sizeof(pthread_t)*max);
        if(pool->threadIDs == NULL)
        {
            printf("malloc threadIds fail");
            break;
        }
        //对工作线程的线程id初始化为
        memset(pool->threadIDs,0,sizeof(pthread_t)* max);
        //对结构体内部数据进行初始化
        pool->minNum = min; //最小线程数
        pool->maxNum = max; //最大线程数
        pool->busyNum=0;   //当前忙碌的线程
        pool->liveNum = min; //当前活着线程数，等于最小个数线程数
        pool->exitNum = 0; //要销毁的线程个数
        //对互斥锁与条件变量进行初始化
        if(pthread_mutex_init(&pool->mutexPool,NULL)!=0 ||
            pthread_mutex_init(&pool->mutexBusy,NULL) != 0 ||
            pthread_cond_init(&pool->notEmpty,NULL) !=0 ||
            pthread_cond_init(&pool->notFull,NULL) != 0)
        {
            printf("mutex or condition init fail...\n");
            break;
        }
        //任务队列初始化
        pool->taskQ = (Task*)malloc(sizeof(Task) * queueSize);  //大小为参数给定的容量
        //TODO
        // if(pool->taskQ == NULL)
        // {
        //     printf("malloc TaskID fail ... ");
        //     break;
        // }
        pool->queueCapacity = queueSize; //设置任务队列容 量
        pool->queueSize = 0;  //设置任务队列大小
        pool->queueFront = 0;  //设置任务队列头节点
        pool->queueRear = 0;  //队列位节点位置

        pool->shutdown = 0; //是不是要销毁线程池

        //创建管理者线程一个
        pthread_create(&pool->managerID,NULL,manager,pool);
        //创建工作线程
        for (int i = 0; i < min; ++i)
        {
            pthread_create(&pool->threadIDs[i],NULL,worker,pool);
        }
        return pool;
    } while (0);
    
    //如果上面的申请空间失败，则释放申请的资源
    if(pool && pool->threadIDs)
    {
        free(pool->threadIDs);
    }
    if(pool && pool->taskQ) free(pool->taskQ);
    if(pool) free(pool);

    return NULL;
    
}
//销毁当前线程
int threadPoolDestroy(ThreadPool* pool)
{
    //如果线程池结构体为空
    if(pool == NULL)
    {
        return -1;
    }
    //关闭线程池
    pool->shutdown = 1;  //通过子线程的不断检测，如关闭线程池，则关闭释放先关线程与执行
    //阻塞回收管理者线程
    pthread_join(pool->managerID,NULL);
    //唤醒阻塞的消费者线程  还存活的线程个数
    for(int i=0;i<pool->liveNum;++i)
    {
        pthread_cond_signal(&pool->notEmpty); //通过信号量唤醒，那些因为任务队列满了，判断是够有空的线程
    }
    //释放对内存
    if(pool->taskQ)
    {
        free(pool->taskQ);
    }
    if(pool->threadIDs)
    {
        free(pool->threadIDs);
    }
    //销毁互斥锁与条件变量
    pthread_mutex_destroy(&pool->mutexPool);
    pthread_mutex_destroy(&pool->mutexBusy);
    pthread_cond_destroy(&pool->notEmpty);
    pthread_cond_destroy(&pool->notFull);
    //释放线程池结构体
    free(pool);
    pool = NULL;
    printf("线程池完全退出.....\n");
    return 0;
}
//添加线程,在初始化后通过此向线程池中调用线程
void threadPoolAdd(ThreadPool* pool,void(*func)(void*),void* arg)
{
    //添加整个线程互斥锁
    pthread_mutex_lock(&pool->mutexPool);
    //当任务队列当前任务个数等于任务队列容量 并且线程池非销毁的状态性,阻塞等待任务队列空闲或者被销毁
    while(pool->queueSize == pool->queueCapacity && !pool->shutdown)
    {
        //阻塞生产线程
        pthread_cond_wait(&pool->notFull,&pool->mutexPool); //处于 线程队列是不是有空 条件变量，解锁线程互斥锁
    }
    //如果处于销毁线程
    if(pool->shutdown)
    {
        //解锁线程互斥锁
        pthread_mutex_unlock(&pool->mutexPool);
        return;
    }
    //添加任务
    pool->taskQ[pool->queueRear].function = func; //使用尾插法,传入函数指针
    pool->taskQ[pool->queueRear].arg = arg; //传入参数
    //更新尾插入的位置
    pool->queueRear = (pool->queueRear + 1) % pool->queueCapacity;
    //
    pool->queueSize++; //当前任务个数++

    // 任务队列满 控件 
    // 唤醒在条件变量上的线程，至少一个被解除阻塞
    pthread_cond_signal(&pool->notEmpty);
    //解整个线程互斥锁
    pthread_mutex_unlock(&pool->mutexPool);
}

// 返回当前忙碌的线程个数
int threadPoolBusyNum(ThreadPool *pool)
{
    //阻塞忙碌的线程
    pthread_mutex_lock(&pool->mutexBusy);
    int busyNum = pool->busyNum;
    pthread_mutex_unlock(&pool->mutexBusy);
    return busyNum;
}
int threadPoolAliveNum(ThreadPool* pool)
{
    //阻塞线程池互斥锁
    pthread_mutex_lock(&pool->mutexPool);
    int aliveNum = pool->liveNum;
    pthread_mutex_unlock(&pool->mutexPool);
    return aliveNum;
}
//工作线程，负责消费线程运行任务队列
void* worker(void*arg)
{
    //将传入的结构体转换
    ThreadPool* pool = (ThreadPool*)arg;
    while(1)
    {
        //线程池锁
        pthread_mutex_lock(&pool->mutexPool);
        //当前任务队列是否为空并且不处于销毁线程状态
        while(pool->queueSize ==0 && !pool->shutdown)
        {
            //阻塞工作线程，阻塞，任务队列满
            pthread_cond_wait(&pool->notEmpty,&pool->mutexPool); //
            //判断是不是要销毁线程
            if(pool->exitNum > 0) //与管理员线程销毁线程相关联
            {
                pool->exitNum --;
                //如果存活的线程大于最小线程数，适当减小存活的线程
                if(pool->liveNum > pool->minNum)
                {
                    pool->liveNum--;
                    pthread_mutex_unlock(&pool->mutexPool);
                    //释放线程
                    threadExit(pool);
                }
            }
        }
        //判断线程池是否被关闭了
        if(pool->shutdown)
        {
            pthread_mutex_unlock(&pool->mutexPool);
            threadExit(pool);
        }
        
        //从任务队列取出一个任务
        Task task;
        //从队列头部去函数指针与参数
        task.function = pool->taskQ[pool->queueFront].function;
        task.arg = pool->taskQ[pool->queueFront].arg;
        //移动头结点
        pool->queueFront =(pool->queueFront + 1) % pool->queueCapacity;
        pool->queueSize--;
        //解锁
        pthread_cond_signal(&pool->notFull); //判断是否满的条件变量，释放一次任务队列是否满空间，仍阻塞的任务队列进入
        pthread_mutex_unlock(&pool->mutexPool);  //解锁线程池锁

        //运行线程
        printf("thread %ld start working...\n",pthread_self());
        //加锁，当前正在忙碌的线程+1
        pthread_mutex_lock(&pool->mutexBusy);
        pool->busyNum++;
        pthread_mutex_unlock(&pool->mutexBusy);
        //运行当前线程
        task.function(task.arg);
        //释放参数空间
        free(task.arg);
        //将参数设置为空
        task.arg = NULL;
        //运行之后，忙碌的线程-1
        printf("thread %ld end working ...\n",pthread_self());
        pthread_mutex_lock(&pool->mutexBusy);
        pool->busyNum--;
        pthread_mutex_unlock(&pool->mutexBusy);
    }
    return NULL;
}
void *manager(void* arg)
{
    //传入的结构体
    ThreadPool* pool = (ThreadPool*)arg;
    //当并没有关闭时
    while(!pool->shutdown)
    {
        //每隔3秒检测一次
        sleep(3);
        // 取出线程池中任务的数量和当前线程的数量
        pthread_mutex_lock(&pool->mutexPool);
        
        int queueSize = pool->queueSize;
        int liveNum = pool->liveNum;

        pthread_mutex_unlock(&pool->mutexPool);

        //取出忙的线程数量
        pthread_mutex_lock(&pool->mutexBusy);
        int busyNum = pool->busyNum;
        pthread_mutex_unlock(&pool->mutexBusy);
        
        //添加线程
        //当任务的个数 > 存活的线程个数&& 存活的线程数 < 最大线程数
        if(queueSize > liveNum && liveNum < pool->maxNum)
        {
            pthread_mutex_lock(&pool->mutexPool);
            int counter = 0;
            for(int i = 0; i < pool->maxNum && counter < NUMBER && pool->liveNum < pool->maxNum;++i)
            {
                if(pool->threadIDs[i] == 0)
                {
                    pthread_create(&pool->threadIDs[i],NULL,worker,pool);
                    counter++;
                    pool->liveNum++; //存活的线程个数
                }
            }
            pthread_mutex_unlock(&pool->mutexPool);
        }
        //销毁线程
        // 忙的线程*2 < 存活的线程数 && 存活的线程 > 最小线程数
        if(busyNum * 2 < liveNum && liveNum > pool->minNum)
        {
            pthread_mutex_lock(&pool->mutexPool);
            pool->exitNum = NUMBER;
            pthread_mutex_unlock(&pool->mutexPool);
            //让工作的线程自杀
            for(int i =0;i<NUMBER;++i)
            {
                pthread_cond_signal(&pool->notEmpty);  //释放工作线程，使其释放线程
            }
        }
    }
    return NULL;
}
//退出线程
void threadExit(ThreadPool* pool)
{
    pthread_t tid = pthread_self();
    for (int i = 0; i < pool->maxNum; ++i)
    {
        if (pool->threadIDs[i] == tid)
        {
            pool->threadIDs[i] = 0;
            printf("threadExit() called, %ld exiting...\n", tid);
            break;
        }
    }
    pthread_exit(NULL);
}
int threadPoolISEmpty(ThreadPool* pool)
{
    //要销毁线程，任务队列，忙的线程和都为零
    if(pool->queueSize == 0 && pool->busyNum == 0 && threadPoolAliveNum(pool) == pool->minNum)
    {
        return 1;
    }
    else
    {
        return 0;
    }
}
```
#### 测试函数
```c
#include "threadpool.h"
#include <unistd.h>
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <stdlib.h>
void taskFunc(void* arg)
{
    int num =*(int*)arg;
    printf("thread %ld is working,number = %d\n",pthread_self(),num);
    sleep(1);
}
int main(void)
{
    //创建线程池
    ThreadPool *pool = threadPoolCreate(3,100,1000);
    for(int i=0;i<100;++i)
    {
        int* num =(int*)malloc(sizeof(int));
        *num = i + 100;
        //添加任务队列
        threadPoolAdd(pool,taskFunc,num);
    }
    while(1)
    {
        if(threadPoolISEmpty(pool) == 1)
        {
            printf("销毁线程池 ing \n");
            threadPoolDestroy(pool);
            break;
        }
    }
    return 0;
}
```
### 基于POSIX线程池

#### 头文件

##### threadpool.h
```cpp
#ifndef _THREADPOOL_H
#define _THREADPOOL_H
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#define NUMBER 3
//任务队列
typedef struct Task
{
    void (*function)(void* arg);  //任务队列存储函数指针
    void* arg;  //存储函数的参数指针
}Task;
//线程池结构体
typedef struct ThreadPool
{
    Task* taskQ; //任务队列(通过申请空间，乘以容量，即可成为任务队列)
    int queueCapacity; //任务队列容量
    int queueSize;  //当前任务个数
    int queueFront;  //队头取数据
    int queueRear;   //队尾  放数据

    pthread_t managerID; //管理者线程ID 仅一个
    pthread_t *threadIDs; //工作的线程ID，同样申请多个
    int minNum;  //最小线程数量
    int maxNum;   //最大小城数量
    int busyNum;  //忙的线程个数 运行中
    int liveNum;  //存活的线程个数
    int exitNum;  //要销毁的线程个数
    pthread_mutex_t mutexPool; //锁整个的线程池
    pthread_mutex_t mutexBusy; //锁busyNum变量
    pthread_cond_t notFull;   //任务队列是不是满了
    pthread_cond_t notEmpty; //任务队列是不是空

    int shutdown;  //是不是要销毁线程池，销毁为1，不销毁为0

}ThreadPool;

//创建线程并初始化,传入参数最小线程数量，最大线程数量，和队列容量
ThreadPool *threadPoolCreate(int min,int max,int queueSize);
// 该线程添加任务
// 传入参数线程结构体，执行任务的函数指针,函数参数
void threadPoolAdd(ThreadPool* pool,void(*func)(void*),void* arg);

//获取当前线程池中工作的线程个数
int threadPoolBusyNum(ThreadPool *pool);

//获取当前线程池中活着的线程个数
int threadPoolAliveNum(ThreadPool* pool);

//工作的线程(消费者线程)任务函数
void* worker(void*arg);
//管理者线程任务函数
void * manager(void* arg);

//单个线程退出
void threadExit(ThreadPool* pool);
//销毁线程池  传入线程pool
int threadPoolDestroy(ThreadPool* pool);
//线程池所有线程和任务皆为空
int threadPoolISEmpty(ThreadPool* pool);

#endif
```

##### TaskQueue.h

```cpp
#ifndef _TASKQUEUE_H
#define _TASKQUEUE_H
#include <pthread.h>
#include <iostream>
#include <queue>
#include <unistd.h>
#include <string.h>
using namespace std;
// 定义任务结构体 定义别名
using callback = void(*)(void*);
//任务结构体
template <typename T>
struct Task
{
    //构造函数
    Task<T>()
    {
        function = nullptr;
        arg = nullptr;
    }
    //有参函数构造
    Task<T>(callback f, void* arg)
    {
        function = f;
        this->arg = (T*)arg;
    }
    //类 函数指针
    callback function;
    //函数参数
    T* arg;
};

// 任务队列
template <typename T>
class TaskQueue
{
public:
    //构造函数
    TaskQueue();
    ~TaskQueue();

    // 添加任务
    void addTask(Task<T> task);
    void addTask(callback func, void* arg);

    // 取出一个任务
    Task<T> takeTask();

    // 获取当前队列中任务个数
    inline size_t taskNumber() //内联函数提高效率
    {
        return m_queue.size(); //返回队列个数
    }

private:
    //取任务和添加任务，加锁
    pthread_mutex_t m_mutex;    // 互斥锁
    //任务结构体的队列
    std::queue<Task<T>> m_queue;   // 任务队列
};
#endif
```

#### 实现函数

##### threadpool.cpp

```cpp
#include "threadpool.h"

ThreadPool* threadPoolCreate(int min,int max,int queueSize)
{
    //初始化结构体空间
    ThreadPool* pool = (ThreadPool*)malloc(sizeof(ThreadPool));

    do
    {
        //判断申请空间是否成功
        if(pool == NULL)
        {
            printf("malloc threadPool fail...\n");
            break;
        }
        //申请工作线程ID攻坚
        //申请最大空间的
        pool->threadIDs =(pthread_t*)malloc(sizeof(pthread_t)*max);
        if(pool->threadIDs == NULL)
        {
            printf("malloc threadIds fail");
            break;
        }
        //对工作线程的线程id初始化为
        memset(pool->threadIDs,0,sizeof(pthread_t)* max);
        //对结构体内部数据进行初始化
        pool->minNum = min; //最小线程数
        pool->maxNum = max; //最大线程数
        pool->busyNum=0;   //当前忙碌的线程
        pool->liveNum = min; //当前活着线程数，等于最小个数线程数
        pool->exitNum = 0; //要销毁的线程个数
        //对互斥锁与条件变量进行初始化
        if(pthread_mutex_init(&pool->mutexPool,NULL)!=0 ||
            pthread_mutex_init(&pool->mutexBusy,NULL) != 0 ||
            pthread_cond_init(&pool->notEmpty,NULL) !=0 ||
            pthread_cond_init(&pool->notFull,NULL) != 0)
        {
            printf("mutex or condition init fail...\n");
            break;
        }
        //任务队列初始化
        pool->taskQ = (Task*)malloc(sizeof(Task) * queueSize);  //大小为参数给定的容量
        //TODO
        // if(pool->taskQ == NULL)
        // {
        //     printf("malloc TaskID fail ... ");
        //     break;
        // }
        pool->queueCapacity = queueSize; //设置任务队列容 量
        pool->queueSize = 0;  //设置任务队列大小
        pool->queueFront = 0;  //设置任务队列头节点
        pool->queueRear = 0;  //队列位节点位置

        pool->shutdown = 0; //是不是要销毁线程池

        //创建管理者线程一个
        pthread_create(&pool->managerID,NULL,manager,pool);
        //创建工作线程
        for (int i = 0; i < min; ++i)
        {
            pthread_create(&pool->threadIDs[i],NULL,worker,pool);
        }
        return pool;
    } while (0);
    
    //如果上面的申请空间失败，则释放申请的资源
    if(pool && pool->threadIDs)
    {
        free(pool->threadIDs);
    }
    if(pool && pool->taskQ) free(pool->taskQ);
    if(pool) free(pool);

    return NULL;
    
}
//销毁当前线程
int threadPoolDestroy(ThreadPool* pool)
{
    //如果线程池结构体为空
    if(pool == NULL)
    {
        return -1;
    }
    //关闭线程池
    pool->shutdown = 1;  //通过子线程的不断检测，如关闭线程池，则关闭释放先关线程与执行
    //阻塞回收管理者线程
    printf("进入管理者");
    pthread_join(pool->managerID,NULL);
    //唤醒阻塞的消费者线程  还存活的线程个数
    printf("阻塞在管理者");
    for(int i=0;i<pool->liveNum;++i)
    {
        pthread_cond_signal(&pool->notEmpty); //通过信号量唤醒，那些因为任务队列满了，判断是够有空的线程
    }
    //释放对内存
    if(pool->taskQ)
    {
        free(pool->taskQ);
    }
    if(pool->threadIDs)
    {
        free(pool->threadIDs);
    }
    //销毁互斥锁与条件变量
    pthread_mutex_destroy(&pool->mutexPool);
    pthread_mutex_destroy(&pool->mutexBusy);
    pthread_cond_destroy(&pool->notEmpty);
    pthread_cond_destroy(&pool->notFull);
    //释放线程池结构体
    free(pool);
    pool = NULL;
    return 0;
}
//添加线程,在初始化后通过此向线程池中调用线程
void threadPoolAdd(ThreadPool* pool,void(*func)(void*),void* arg)
{
    //添加整个线程互斥锁
    pthread_mutex_lock(&pool->mutexPool);
    //当任务队列当前任务个数等于任务队列容量 并且线程池非销毁的状态性,阻塞等待任务队列空闲或者被销毁
    while(pool->queueSize == pool->queueCapacity && !pool->shutdown)
    {
        //阻塞生产线程
        pthread_cond_wait(&pool->notFull,&pool->mutexPool); //处于 线程队列是不是有空 条件变量，解锁线程互斥锁
    }
    //如果处于销毁线程
    if(pool->shutdown)
    {
        //解锁线程互斥锁
        pthread_mutex_unlock(&pool->mutexPool);
        return;
    }
    //添加任务
    pool->taskQ[pool->queueRear].function = func; //使用尾插法,传入函数指针
    pool->taskQ[pool->queueRear].arg = arg; //传入参数
    //更新尾插入的位置
    pool->queueRear = (pool->queueRear + 1) % pool->queueCapacity;
    //
    pool->queueSize++; //当前任务个数++

    // 任务队列满 控件 
    // 唤醒在条件变量上的线程，至少一个被解除阻塞
    pthread_cond_signal(&pool->notEmpty);
    //解整个线程互斥锁
    pthread_mutex_unlock(&pool->mutexPool);
}

// 返回当前忙碌的线程个数
int threadPoolBusyNum(ThreadPool *pool)
{
    //阻塞忙碌的线程
    pthread_mutex_lock(&pool->mutexBusy);
    int busyNum = pool->busyNum;
    pthread_mutex_unlock(&pool->mutexBusy);
    return busyNum;
}
int threadPoolAliveNum(ThreadPool* pool)
{
    //阻塞线程池互斥锁
    pthread_mutex_lock(&pool->mutexPool);
    int aliveNum = pool->liveNum;
    pthread_mutex_unlock(&pool->mutexPool);
    return aliveNum;
}
//工作线程，负责消费线程运行任务队列
void* worker(void*arg)
{
    //将传入的结构体转换
    ThreadPool* pool = (ThreadPool*)arg;
    while(1)
    {
        //线程池锁
        pthread_mutex_lock(&pool->mutexPool);
        //当前任务队列是否为空并且不处于销毁线程状态
        while(pool->queueSize ==0 && !pool->shutdown)
        {
            //阻塞工作线程，阻塞，任务队列满
            pthread_cond_wait(&pool->notEmpty,&pool->mutexPool); //
            //判断是不是要销毁线程
            if(pool->exitNum > 0) //与管理员线程销毁线程相关联
            {
                pool->exitNum --;
                //如果存活的线程大于最小线程数，适当减小存活的线程
                if(pool->liveNum > pool->minNum)
                {
                    pool->liveNum--;
                    pthread_mutex_unlock(&pool->mutexPool);
                    //释放线程
                    threadExit(pool);
                }
            }
        }
        //判断线程池是否被关闭了
        if(pool->shutdown)
        {
            pthread_mutex_unlock(&pool->mutexPool);
            threadExit(pool);
        }
        
        //从任务队列取出一个任务
        Task task;
        //从队列头部去函数指针与参数
        task.function = pool->taskQ[pool->queueFront].function;
        task.arg = pool->taskQ[pool->queueFront].arg;
        //移动头结点
        pool->queueFront =(pool->queueFront + 1) % pool->queueCapacity;
        pool->queueSize--;
        //解锁
        pthread_cond_signal(&pool->notFull); //判断是否满的条件变量，释放一次任务队列是否满空间，仍阻塞的任务队列进入
        pthread_mutex_unlock(&pool->mutexPool);  //解锁线程池锁

        //运行线程
        printf("thread %ld start working...\n",pthread_self());
        //加锁，当前正在忙碌的线程+1
        pthread_mutex_lock(&pool->mutexBusy);
        pool->busyNum++;
        pthread_mutex_unlock(&pool->mutexBusy);
        //运行当前线程
        task.function(task.arg);
        //释放参数空间
        free(task.arg);
        //将参数设置为空
        task.arg = NULL;
        //运行之后，忙碌的线程-1
        printf("thread %ld end working ...\n",pthread_self());
        pthread_mutex_lock(&pool->mutexBusy);
        pool->busyNum--;
        pthread_mutex_unlock(&pool->mutexBusy);
    }
    return NULL;
}
void *manager(void* arg)
{
    //传入的结构体
    ThreadPool* pool = (ThreadPool*)arg;
    //当并没有关闭时
    while(!pool->shutdown)
    {
        //每隔3秒检测一次
        sleep(3);
        // 取出线程池中任务的数量和当前线程的数量
        pthread_mutex_lock(&pool->mutexPool);
        
        int queueSize = pool->queueSize;
        int liveNum = pool->liveNum;

        pthread_mutex_unlock(&pool->mutexPool);

        //取出忙的线程数量
        pthread_mutex_lock(&pool->mutexBusy);
        int busyNum = pool->busyNum;
        pthread_mutex_unlock(&pool->mutexBusy);
        
        //添加线程
        //当任务的个数 > 存活的线程个数&& 存活的线程数 < 最大线程数
        if(queueSize > liveNum && liveNum < pool->maxNum)
        {
            pthread_mutex_lock(&pool->mutexPool);
            int counter = 0;
            for(int i = 0; i < pool->maxNum && counter < NUMBER && pool->liveNum < pool->maxNum;++i)
            {
                if(pool->threadIDs[i] == 0)
                {
                    pthread_create(&pool->threadIDs[i],NULL,worker,pool);
                    counter++;
                    pool->liveNum++; //存活的线程个数
                }
            }
            pthread_mutex_unlock(&pool->mutexPool);
        }
        //销毁线程
        // 忙的线程*2 < 存活的线程数 && 存活的线程 > 最小线程数
        if(busyNum * 2 < liveNum && liveNum > pool->minNum)
        {
            pthread_mutex_lock(&pool->mutexPool);
            pool->exitNum = NUMBER;
            pthread_mutex_unlock(&pool->mutexPool);
            //让工作的线程自杀
            for(int i =0;i<NUMBER;++i)
            {
                pthread_cond_signal(&pool->notEmpty);  //释放工作线程，使其释放线程
            }
        }
    }
    return NULL;
}
//退出线程
void threadExit(ThreadPool* pool)
{
    pthread_t tid = pthread_self();
    for(int i = 0;i<pool->maxNum;++i)
    {
        if(pool->threadIDs[i] ==tid)
        {
            pool->threadIDs = 0;
            printf("threadExit() called, %ld exiting ...\n",tid);
            break;
        }
    }
    pthread_exit(NULL);
}
int threadPoolISEmpty(ThreadPool* pool)
{
    //要销毁线程，任务队列，忙的线程和都为零
    if(pool->queueSize == 0 && pool->busyNum == 0 && threadPoolAliveNum(pool) == pool->minNum)
    {
        return 1;
    }
    else
    {
        return 0;
    }
}
```

##### TaskQueue.cpp

```cpp
#include "TaskQueue.h"
template <typename T>
TaskQueue<T>::TaskQueue()
{
    pthread_mutex_init(&m_mutex, NULL);
}
template <typename T>
TaskQueue<T>::~TaskQueue()
{
    pthread_mutex_destroy(&m_mutex);
}
template <typename T>
void TaskQueue<T>::addTask(Task<T> task)
{
    pthread_mutex_lock(&m_mutex);
    m_queue.push(task); //将任务压入队列
    pthread_mutex_unlock(&m_mutex);
}
//重载添加任务队列
template <typename T>
void TaskQueue<T>::addTask(callback func, void* arg)
{
    pthread_mutex_lock(&m_mutex);
    Task<T> task; //创建结构体，将所需要的函数和参数传入结构体，然后压入队列
    task.function = func;
    task.arg = arg;
    m_queue.push(task);
    pthread_mutex_unlock(&m_mutex);
}
template <typename T>
Task<T> TaskQueue<T>::takeTask()
{
    Task<T> t;
    pthread_mutex_lock(&m_mutex);
    if (m_queue.size() > 0)
    {
        t = m_queue.front(); //取头部节点
        m_queue.pop(); //弹出
    }
    pthread_mutex_unlock(&m_mutex);
    return t;
}
```



#### 调用测试

```cpp
#include "threadpool.h"
#include <unistd.h>
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <stdlib.h>



void taskFunc(void* arg)
{
    int num =*(int*)arg;
    printf("thread %ld is working,number = %d\n",pthread_self(),num);
    sleep(1);
}
int main(void)
{
    //创建线程池
    ThreadPool *pool = threadPoolCreate(3,100,1000);
    for(int i=0;i<100;++i)
    {
        int* num =(int*)malloc(sizeof(int));
        *num = i + 100;
        //添加任务队列
        threadPoolAdd(pool,taskFunc,num);
        
    }
    while(1)
    {
        if(threadPoolISEmpty(pool) == 1)
        {
            printf("销毁线程池 ing \n");
            threadPoolDestroy(pool);
            break;
        }
    }
    
    return 0;
}
```

### 基于C++11的跨平台线程池
#### 头文件
##### threadpool.h
```cpp
#ifndef _THREADPOOL_H
#define _THREADPOOL_H
#include <iostream>
#include <queue>
#include <string.h>
#include <string>
#include "TaskQueue.h"
#include "TaskQueue.cpp"
#include <thread>
#include <Windows.h>
#include <mutex>
#include <vector>
#include <condition_variable>
using namespace std;
//线程池类
template <typename T>
class ThreadPool
{
public:
    //默认构造函数
    ThreadPool(int min, int max);
    ~ThreadPool();

    // 添加任务
    void addTask(Task<T> task);
    // 获取忙线程的个数
    int getBusyNumber();
    // 获取活着的线程个数
    int getAliveNumber();

private:
    // 工作的线程的任务函数
    static void* worker(void* arg);
    // 管理者线程的任务函数
    static void* manager(void* arg);

private:
    mutex m_lock;
    condition_variable m_notEmpty;
    vector<thread> m_threadIDs;  //工作线程
    thread m_managerID;  //管理者线 程
    TaskQueue<T>* m_taskQ;  //任务队列
    static const int NUMBER = 2;
    int m_minNum;  //最小数量
    int m_maxNum;  //最大数量
    int m_busyNum;  //忙碌的线程
    int m_aliveNum;  //存活的线程数
    int m_exitNum;  //要销毁的线程数
    bool m_shutdown = false;  //是否要销毁线程池
};
#endif
```
##### TaskQueue.h
```cpp
#pragma once
#ifndef _TASKQUEUE_H
#define _TASKQUEUE_H
#include <thread>
#include <iostream>
#include <queue>
#include <string.h>
#include <Windows.h>
#include <mutex>
#include <condition_variable>
using namespace std;
// 定义任务结构体 定义别名
using callback = void(*)(void*);
//任务结构体
template <typename T>
struct Task
{
    //构造函数
    Task<T>()
    {
        function = nullptr;
        arg = nullptr;
    }
    //有参函数构造
    Task<T>(callback f, void* arg)
    {
        function = f;
        this->arg = (T*)arg;
    }
    //类 函数指针
    callback function;
    //函数参数
    T* arg;
};

// 任务队列
template <typename T>
class TaskQueue
{
public:
    //构造函数
    TaskQueue();
    ~TaskQueue();

    // 添加任务
    void addTask(Task<T> task);
    void addTask(callback func, void* arg);

    // 取出一个任务
    Task<T> takeTask();

    // 获取当前队列中任务个数
    inline size_t taskNumber() //内联函数提高效率
    {
        return m_queue.size(); //返回队列个数
    }

private:
    //取任务和添加任务，加锁
    mutex m_mutex;    // 互斥锁
    //任务结构体的队列
    std::queue<Task<T>> m_queue;   // 任务队列
};
#endif
```


#### 实现函数

##### threadpool.cpp
```cpp
#include "threadPool.h"
//初始化线程池，需要的最小和最大数量参数
template <typename T>
ThreadPool<T>::ThreadPool(int minNum, int maxNum)
{
    // 实例化任务队列
    m_taskQ = new TaskQueue<T>;
    do
    {
        // 初始化线程池
        m_minNum = minNum;
        m_maxNum = maxNum;
        m_busyNum = 0;
        m_aliveNum = minNum;

        m_managerID = thread(manager, this);

        m_threadIDs.resize(m_maxNum);
        for (int i = 0; i < m_minNum; i++)
        {
            m_threadIDs[i] = thread(worker, this);
        }
        return;
    } while (0);
}

template <typename T>
ThreadPool<T>::~ThreadPool()
{
    cout << "线程进入退出程序 ....!" << endl;
    m_shutdown = true;
    // 销毁管理者线程
    //pthread_join(m_managerID, NULL);
    if (m_managerID.joinable())
    {
        m_managerID.join();
    }
    // 唤醒所有消费者线程
    m_notEmpty.notify_all();
    for (int i = 0; i < m_maxNum; ++i)
    {
        if (m_threadIDs[i].joinable())
        {
            m_threadIDs[i].join();
        }
    }
    
    cout << "线程退出完毕.....!" << endl;
}
template <typename T>
void ThreadPool<T>::addTask(Task<T> task)
{
    if (m_shutdown)
    {
        return;
    }
    // 添加任务，不需要加锁，任务队列中有锁
    //调用 任务队列函数,从线程池中调用即可
    m_taskQ->addTask(task);
    // 唤醒工作的线程
    //pthread_cond_signal(&m_notEmpty); //不是空的，会进行对应的任务处理
    m_notEmpty.notify_all();
}
template <typename T>
int ThreadPool<T>::getAliveNumber()
{
    int threadNum = 0;
    m_lock.lock();
    threadNum = m_aliveNum;
    m_lock.unlock();
    return threadNum;
}
template <typename T>
int ThreadPool<T>::getBusyNumber()
{
    int busyNum = 0;
    m_lock.lock();
    busyNum = m_busyNum;
    m_lock.unlock();
    return busyNum;
}


// 工作线程任务函数
template <typename T>
void* ThreadPool<T>::worker(void* arg)
{
    ThreadPool* pool = static_cast<ThreadPool*>(arg);
    // 一直不停的工作
    while (true)
    {
        // 访问任务队列(共享资源)加锁
        unique_lock<std::mutex> lk(pool->m_lock);
        // 判断任务队列是否为空, 如果为空工作线程阻塞
        while (pool->m_taskQ->taskNumber() == 0 && !pool->m_shutdown)
        {
            cout << "thread " << this_thread::get_id() << " waiting..." << endl;
            // 阻塞线程
            //pthread_cond_wait(&pool->m_notEmpty, &pool->m_lock);
            pool->m_notEmpty.wait(lk);

            // 解除阻塞之后, 判断是否要销毁线程
            if (pool->m_exitNum > 0)
            {
                pool->m_exitNum--;
                if (pool->m_aliveNum > pool->m_minNum)
                {
                    pool->m_aliveNum--;
                    cout << "threadId :" << this_thread::get_id() <<"exit ......" << endl;
                    lk.unlock();
                    return nullptr;
                }
            }
        }
        // 判断线程池是否被关闭了
        if (pool->m_shutdown)
        {
            cout << "threadId :" << this_thread::get_id() << "exit ......" << endl;
            //lk.unlock();
            return nullptr;
        }

        // 从任务队列中取出一个任务
        Task<T> task = pool->m_taskQ->takeTask();
        // 工作的线程+1
        pool->m_busyNum++;
        // 线程池解锁
        lk.unlock();
        // 执行任务
        cout << "thread " << this_thread::get_id() << " start working..." << endl;
        // 执行取出的函数
        task.function(task.arg);
        delete task.arg;
        task.arg = nullptr;

        // 任务处理结束
        cout << "thread " << this_thread::get_id() << " end working..." << endl;
        lk.lock();
        pool->m_busyNum--;
        lk.unlock();
    }

    return nullptr;
}


// 管理者线程任务函数
template <typename T>
void* ThreadPool<T>::manager(void* arg)
{
    ThreadPool* pool = static_cast<ThreadPool*>(arg);
    // 如果线程池没有关闭, 就一直检测
    while (!pool->m_shutdown)
    {
        // 每隔5s检测一次
        this_thread::sleep_for(2s);
        // 取出线程池中的任务数和线程数量
        //  取出工作的线程池数量
        unique_lock<mutex> lk(pool->m_lock);
        int queueSize = pool->m_taskQ->taskNumber();
        int liveNum = pool->m_aliveNum;
        int busyNum = pool->m_busyNum;
        lk.unlock();

        // 创建线程

        // 当前任务个数>存活的线程数 && 存活的线程数<最大线程个数
        if (queueSize > liveNum && liveNum < pool->m_maxNum)
        {
            // 线程池加锁
            lk.lock();
            int num = 0;
            for (int i = 0; i < pool->m_maxNum && num < NUMBER
                && pool->m_aliveNum < pool->m_maxNum; ++i)
            {
                if (pool->m_threadIDs[i].get_id() == thread::id())
                {
                    //pthread_create(&pool->m_threadIDs[i], NULL, worker, pool);
                    pool->m_threadIDs[i] = thread(worker, pool);
                    num++;
                    pool->m_aliveNum++;
                }
            }
            lk.unlock();
        }

        // 销毁多余的线程
        // 忙线程*2 < 存活的线程数目 && 存活的线程数 > 最小线程数量
        if (busyNum * 2 < liveNum && liveNum > pool->m_minNum)
        {
            lk.lock();

            pool->m_exitNum = NUMBER;
            lk.unlock();
            for (int i = 0; i < NUMBER; ++i)
            {
                pool->m_notEmpty.notify_all();
            }
        }
    }
    return nullptr;
}

```
##### TaskQueue.cpp
```cpp
#include "TaskQueue.h"
template <typename T>
TaskQueue<T>::TaskQueue()
{
    //pthread_mutex_init(&m_mutex, NULL);
}
template <typename T>
TaskQueue<T>::~TaskQueue()
{
    //pthread_mutex_destroy(&m_mutex);
}
template <typename T>
void TaskQueue<T>::addTask(Task<T> task)
{
    m_mutex.lock();
    m_queue.push(task); //将任务压入队列
    m_mutex.unlock();
}
//重载添加任务队列
template <typename T>
void TaskQueue<T>::addTask(callback func, void* arg)
{
    m_mutex.lock();
    Task<T> task; //创建结构体，将所需要的函数和参数传入结构体，然后压入队列
    task.function = func;
    task.arg = arg;
    m_queue.push(task);
    m_mutex.unlock();
}
template <typename T>
Task<T> TaskQueue<T>::takeTask()
{
    Task<T> t;
    m_mutex.lock();
    if (m_queue.size() > 0)
    {
        t = m_queue.front(); //取头部节点
        m_queue.pop(); //弹出
    }
    m_mutex.unlock();
    return t;
}
```

#### 测试代码调用

```cpp
#include "threadPool.h"
#include "threadPool.cpp"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <iostream>
#include <thread>
#include <Windows.h>
#include <mutex>
#include <condition_variable>
using namespace std;

void taskFunc(void* arg)
{
    int num = *(int*)arg;
    cout << "thread" << this_thread::get_id() << " is working, number =" << to_string(num) << endl;
    this_thread::sleep_for(1s);
}
int main(void)
{
    //创建线程池
    ThreadPool<int> pool(3, 8);
    for (int i = 0; i < 100; ++i)
    {

        int* num = new int(i + 100); //new和delete对应
        //添加任务队列
        pool.addTask(Task<int>(taskFunc, num));
    }
    this_thread::sleep_for(30s);
    return 0;
}
```
