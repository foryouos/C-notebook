# Linux多线程
> 多线程的编程功力直接决定服务器性能的优异
> 充分利用内核：使用到所有的内核，内核不空闲

## POSIX多线程
> 需要依赖头文件pthread.h 在编译的时候要加上库pthread表示包含多线程库文件
![常见的API函数](E:\学习笔记\C-notebook\Linux系统编程\assets\常见API.jpg "常见的API函数")

### 线程的创建
```cpp
int pthread_create(pthread_t *pid,const pthread_attr_t *attr,void *(*start_routine)(void *),void *arg)
//pid为线程创建后的id
//attr为指向线程属性结构的pthread_attr_t的指针，NULL使用默认属性
//start_routine指向线程函数的地址，要执行的函数
//arg指向线程函数的参数
//如果成功，函数返回0
```
### pthread_join
> 函数pthread_join来等待子线程结束，函数会让主线程挂起(即休眠，让出CPU)，指导子线程退出，同时pthread_join能让子线程所占资源也释放
```cpp
int pthread_join(pthread_t pid,void **value_ptr);
//pid是所等待线程的ID，value_ptr通常设为NULL。
//如果不为NULL，pthread_join复制一份线程退出值到一个内存区域，因此pthread_join还有一个重要功能就是获得子线程的返回值。如果函数成功，返回0,否则返回错误码
```

#### 创建线程(不传参数)
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
#### 传递字符串
```cpp

```

## C++11/14 多线程



> 参考资料:
> Linux C/C++服务器开发实战