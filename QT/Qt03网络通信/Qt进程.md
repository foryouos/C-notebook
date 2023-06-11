# Qt进线程

> `#include <QProcess>`
>
> 定义私有进程
> `QProcess myprocess;`

```cpp
void Widget::on_pushButton_clicked()
{
// 调用start调用其它进程，
    //调用进程具体的exe文件
myprocess.start("C:\\Qt\\Qt5.6.1\\5.6\\mingw49_32\\bin\\assistant.exe"); 
    //等待process进程结束,然后才可以操作本进程，阻塞到此，直到关闭此进程
    myprocess.waitForFinished();
}
void Widget::on_pushButton_2_clicked()
{
    qDebug() << "Hello world";
}
```

## 进程通信

![进程通信](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbshWwiapqz7vzD3cbGKibtuwOo4WdI8J0Vibxala5I8mO39CicJUHsvVyLAMqSScVDiciazjSIZSVECKQgg/640?wx_fmt=png "进程通信")

### read

```cpp
//头文件
#include <QSharedMemory>  //进程键通信
private:
    QSharedMemory sharedmemory; //进程通信类

//函数实现
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    //读端绑定共享区域关键字
    sharedmemory.setKey("My_Shared_Memory");
}

Widget::~Widget()
{

    delete ui;
}

void Widget::on_pushButton_clicked()
{
    //判断共享区域是否处于绑定状态
    if(!sharedmemory.attach())
    {
        qDebug() << "Attach shared memory faield";
        return;
    }
    //对读数据段进行加锁
    sharedmemory.lock();
    char *arr = new char[ sharedmemory.size() ];

    //将共享区域共享到arr中
    memcpy(arr, sharedmemory.data(), static_cast<size_t>(sharedmemory.size()));
    //将共享区域的文字显示到区域中
    ui->label->setText(QString(arr));
    //解锁
    sharedmemory.unlock();
    // 解除绑定
    sharedmemory.detach();
}
```





### write

```cpp
//头文件
#include <QSharedMemory>  //进程键通信
private:
    QSharedMemory sharedmemory; //进程通信类

//函数实现
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    //建立进程通信，设置通信Key，写端和读端必须相同
    sharedmemory.setKey("My_Shared_Memory");
}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_pushButton_clicked()
{
    //判断sharedmemory是否处于绑定状态
    if(sharedmemory.isAttached())
    {
        //如果处于绑定状态进行解除绑定
        sharedmemory.detach();
    }

    //创立进程通信连接
    //100为size空间大小
    //  QSharedMemory::ReadWrite
    if(!sharedmemory.create(100, QSharedMemory::ReadWrite))
    {
        //如果失败了，没有建立成功
        qDebug() << "Failed to create shared memory!";
        return;
    }
    //对进程共享区域加锁
    sharedmemory.lock();
    //向共享区域data写入数据
    QByteArray arr = ui->lineEdit->text().toLatin1();
    //qMin取参数最小值，参数为共享内存大小以及用户输入的空间大小，取最小，将用户输入的数据传递给共享区域
    //向共享区域发送数据
    memcpy(sharedmemory.data(), arr.data(), static_cast<size_t>(qMin(sharedmemory.size(), ui->lineEdit->text().size())));
    //对共享区域进行解说
    sharedmemory.unlock();
}

```

## 多线程信号量

![多进程信号量](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbshWwiapqz7vzD3cbGKibtuwOTiaiaxT5Br250T84r258GwyZvT637xiaghCB632XB3KicdiaDCLVNIdZsHA/640?wx_fmt=png "多进程信号量")

### `mythread.h`

```cpp

#ifndef MYTHREAD_H
#define MYTHREAD_H

#include <QObject>
#include <QThread>  //多线程头文件
#include <QSemaphore> //信号量头文件

//生产者
class Producer : public QThread  //创建饿线程类基于父类QThread
{
    Q_OBJECT
public:
    Producer(){stopped = false;}
    void stop();
protected:
    void run();
private:
    bool stopped;
};
//消费者
class Consumer : public QThread
{
    Q_OBJECT
public:
    Consumer(){stopped = false;} //默认是运行的
    void stop();  //线程停止
protected:
    void run();  //线程从run函数开始运行，类似与main函数，run函数的调用靠的是QThread类里面的start函数
private:
    bool stopped;  //是否停止
};

#endif // MYTHREAD_H
```

### `mythread.cpp`

```cpp
#include "mythread.h"
#include <QDebug>

const int size = 10;
unsigned int buffer[size]; //缓存区
QSemaphore usedSem(0); //已经使用的信号量
QSemaphore unusedSem(size); //未使用的信号量

void Producer::run() //生产者 运行函数
{
    unsigned int i = 0;
    while(!stopped) //当没有设置停止时
    {
        //使用的信号量减一
       unusedSem.acquire();  //unusedSem 减 1
       //将值赋值给缓存区
       buffer[i] = i;
       //休息一秒
       msleep(1000);
       qDebug() << "Write to the shared memory";
       //已经使用的缓冲区+1
       usedSem.release();  //usedSem加1
       i++;
       //如果i达到最大，将i设置为0，
       if(i == size)
           i = 0;
    }
    // 将停止设置为false
    stopped = false;
}

void Producer::stop()
{
    stopped = true;
}

void Consumer::run()
{
    unsigned int i = 1;
    while(!stopped) //当没有点击消费者停止运行信号量时
    {
        usedSem.acquire(); //已经使用的信号量-1
        qDebug() << buffer[i]; //输出缓存区数据
        unusedSem.release();  //未使用的信号量+1
        i++;
        if(i == 10)
            i = 0;
    }
    stopped = false;
}

void Consumer::stop()
{
    stopped = true;  //设置为true停止生产者或消费者运行
}
```

### `widget.cpp`

```cpp
//对应的头文件  部分
#include <mythread.h>
private:
    Ui::Widget *ui;
    Producer producer;
    Consumer consumer;
//函数实现
#include "widget.h"
#include "ui_widget.h"
#include "mythread.h"

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_pushButton_clicked()
{
    //点击之后，生产者开始工作，start相当于运行对应的run函数
    producer.start();
    //点击生产者工作，使生产者不能点击，
    ui->pushButton->setEnabled(false);
    ui->pushButton_2->setEnabled(true);
}

void Widget::on_pushButton_2_clicked()
{
    //判断是否在运行，否则结束运行
    if(producer.isRunning())
    {
        producer.stop();
        ui->pushButton->setEnabled(true);
        ui->pushButton_2->setEnabled(false);
    }
}

void Widget::on_pushButton_3_clicked()
{
    //消费者开始工作
    consumer.start();
    ui->pushButton_3->setEnabled(false);
    ui->pushButton_4->setEnabled(true);
}

void Widget::on_pushButton_4_clicked()
{
    if(consumer.isRunning())
    {
        consumer.stop();
        ui->pushButton_3->setEnabled(true);
        ui->pushButton_4->setEnabled(false);
    }
}
```

