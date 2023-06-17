# Qt多线程文件传输项目

## 通信流程

![项目流程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsFicNFBiapQDKsSRiahVHibKMxSxSOXS7CC8icZxSWuGTKaP2KfpR4iayAjGVCKm8ibwEtwwJzgCfewneEw/640?wx_fmt=jpeg "项目流程")



## 项目文件
> `pro`文件添加`network模块`

### 服务端

#### `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include "mytcpsever.h"
namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void on_start_clicked();

    void on_selectfile_clicked();

    void on_pushButton_clicked();
signals:
    void start(QString name); //通知子线程可以工作啦
private:
    Ui::MainWindow *ui;
    MyTcpSever* m_server;
};

#endif // MAINWINDOW_H

```

#### `mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QThread>
#include <QMessageBox>
#include <QFileDialog>
#include <QRandomGenerator>
#include "sendfile.h"
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setWindowTitle("文件发送端");
    ui->ip->setText("127.0.0.1");
    qDebug()<<"当前主线程的Id:" <<QThread::currentThreadId();
    m_server = new MyTcpSever(this);
    ui->port->setText("8989");

    connect(m_server,&MyTcpSever::newClient,this,[=](qintptr socket) //子线程worker对象使用
    {
        //在ui中显示已连接
        ui->status->setText("已连接");

        //处理子线程闲逛的动作
        QThread* subThread = new QThread;
        //添加工作的类，使其工作移动到子线程中去工作
        //将socket文件内容传递进去
        sendfile* worker = new sendfile(socket); //不能指定this父对象，work要移动到子线程中，若添加this则不可指定
        worker->moveToThread(subThread);
        connect(this,&MainWindow::start,worker,&sendfile::working); //主线程调用start信号，子线程就可以调用working工作

        //当子线程working工作完成之后发出一个信号，主线程得到信号之后就可以销毁这些资源
        connect(worker,&sendfile::done,this,[=]()
        {
            //销毁了子线程，不能继续发送文件啦
//            qDebug()<<"销毁子线程和任务对象资源....";
//            subThread->quit(); //有些任务还没有完成
//            subThread->wait(); //等待完成
//            subThread->deleteLater(); //销毁子线程
//            worker->deleteLater();  //也销毁worker进程
//            ui->status->setText("未连接");
        });
        connect(ui->close,&QPushButton::clicked,this,[=]()
        {
            //销毁了子线程，不能继续发送文件啦
            qDebug()<<"销毁子线程和任务对象资源....";
            subThread->quit(); //有些任务还没有完成
            subThread->wait(); //等待完成
            subThread->deleteLater(); //销毁子线程
            worker->deleteLater();  //也销毁worker进程
            ui->status->setText("未连接");
            ui->start->setEnabled(true);
        });

        connect(worker,&sendfile::text,this,[=](QByteArray msg)
        {
            //修改每次发送的不同颜色
            QVector<QColor> colors =
            {
                Qt::red,Qt::green,Qt::black,Qt::blue,Qt::darkRed,Qt::cyan,Qt::magenta
            };
            int index = QRandomGenerator::global()->bounded(colors.size()); //获取colors里面的随机数
            //取出某一种颜色
            ui->msg->setTextColor(colors.at(index)); //设置一种随机颜色

            ui->msg->append(msg);

        });

        //同步文件发送的总长度
        connect(worker,&sendfile::tot_size_signal,this,[=](qint64 size)
        {
            //qDebug()<<"发送文件的总大小为"<<size;
            ui->progressBar->setMinimum(0);
            ui->progressBar->setMaximum(static_cast<int>(size));
            ui->progressBar->setValue(0); //设置当前值
        });
        //同步文件发送的目前长度
        connect(worker,&sendfile::now_size_signal,this,[=](qint64 size)
        {
            //qDebug()<<"发送当前文件大小"<<size;
            ui->progressBar->setValue(static_cast<int>(size)); //设置当前值
        });

        subThread->start(); //启动子线程 ，启动之后，woker不能工作，需要主线程给子线程发送信号，让其子线程工作
    });
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_start_clicked()
{
    //绑定端口
    unsigned short port = ui->port->text().toUShort();
    m_server->listen(QHostAddress(ui->ip->text()),port);
    //通信的套接字对象不可以跨线程访问
    ui->status->setText("已启动服务器");
    ui->start->setEnabled(false);
}

void MainWindow::on_selectfile_clicked()
{
    QString path = QFileDialog::getOpenFileName(this);//获取文件
    if(!path.isEmpty())
    {
        //将得到路径设置到
        ui->path->setText(path);
    }
    qDebug()<<"用户选中的内容为空";
}
//发送文件
void MainWindow::on_pushButton_clicked()  //发送文件按钮点击之后
{
    //得到文件路径
    if(ui->path->text().isEmpty())
    {
        QMessageBox::information(this,"提示","要发送的文件不能为空");
        return;
    }
    if(!(ui->status->text() == "已连接"))
    {
        QMessageBox::information(this,"提示","还未连接终端或服务器");
        return;
    }

    emit start(ui->path->text());
}

```

#### `mytcpserver.h`

```cpp
#ifndef MYTCPSEVER_H
#define MYTCPSEVER_H

#include <QObject>
#include <QTcpServer>
class MyTcpSever : public QTcpServer
{
    Q_OBJECT
public:
    explicit MyTcpSever(QObject *parent = nullptr);
protected:
    //虚函数，改写行为
    void incomingConnection(qintptr socketDescriptor);
signals:
    void newClient(qintptr socket);
public slots:
};

#endif // MYTCPSEVER_H

```

#### `mytcpserver.cpp`

```cpp
#include "mytcpsever.h"

MyTcpSever::MyTcpSever(QObject *parent) : QTcpServer(parent)
{

}

void MyTcpSever::incomingConnection(qintptr socketDescriptor)
{
    //将文件描述符发送出去
    emit newClient(socketDescriptor);

}

```

#### `sendfile.h`

```cpp
#ifndef SENDFILE_H
#define SENDFILE_H

#include <QObject>
#include <QTcpSocket>
class sendfile : public QObject
{
    Q_OBJECT  //Qt的信号槽机制
public:
    explicit sendfile(qintptr socket,QObject *parent = nullptr);
    void working(QString name);

signals:
    void done(); //working 完成之后
    void text(QByteArray msg);  //发送的文件内容
    void tot_size_signal(qint64 size);  //得到文件总长度，发送信号文件总长度
    void now_size_signal(qint64 size);  //每次发送，同步
public slots:

private:
    qintptr m_socket;
    QTcpSocket* m_tcp;
    qint64 totalsize;  //记录文件的总长度
    qint64 nowsize;    //记录文件的现有长度
};

#endif // SENDFILE_H

```

#### `sendfile.cpp`

```cpp
#include "sendfile.h"
#include <QThread>
#include <QDebug>
#include <QFile>
#include <QFileInfo>
#include <QtEndian>
sendfile::sendfile(qintptr socket,QObject *parent) : QObject(parent)
{
    m_socket = socket; //传递的用于通信的套接字，通过此可以与主函数建立连接的套接字通信
}

void sendfile::working(QString path)
{
    qDebug()<<"当前子线程的Id:" <<QThread::currentThreadId();
    m_tcp = new QTcpSocket;
    m_tcp->setSocketDescriptor(m_socket); //设置socket文件描述符，m_socket就可以通信啦

    connect(m_tcp,&QTcpSocket::disconnected,this,[=]()
    {
        m_tcp->close();
        m_tcp->deleteLater();
        emit done();
        qDebug() << "客户端数据已经接受完毕，并断开了连接,开始销毁套接字对象,拜拜...";
    });
    qDebug()<<"发送的文件名字:"<<path;
    nowsize = 0;
    //通过QFile打开文件
    QFile file(path);
    //获取文件的总长度
    QFileInfo FileData(path);
    totalsize = FileData.size();
    emit tot_size_signal(totalsize);
    //将文件总大小传过去
    QString size = QString::number(totalsize);
    qint64 length = m_tcp->write(size.toUtf8());
    m_tcp->waitForBytesWritten();

    if(length > 0)
    {
        QThread::msleep(50); //休息
    }
    bool bl = file.open(QFile::ReadOnly);
    if(bl)
    {
        while(!file.atEnd()) //如果文件没有读完
        {

            QByteArray line = file.readLine();

            // 添加包头
            int len = qToBigEndian(line.size());
            QByteArray data(reinterpret_cast<char*>(&len), 4);
            data.append(line);

            // 发送数据
            m_tcp->write(data);
            //读取数据

            // 信号
            emit text(line);
            //发送目前的line量
            nowsize +=line.size();
            emit now_size_signal(nowsize);

            //qDebug()<<"发送的数据为"<<line;
            //
            QThread::msleep(50); //休息
        }
    }
    file.close();
}

```


### 客户端

#### `mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include "recvfile.h"
#include <QThread>
namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();
private slots:
    void on_connect_clicked();

signals:
    void startConnect(QString ip,unsigned short port);
private:
    Ui::MainWindow *ui;
    QThread* subThread;
    RecvFile* worker;
};

#endif // MAINWINDOW_H

```

#### `mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QThread>
#include <QMessageBox>
#include <QRandomGenerator>
#include <QDebug>
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setWindowTitle("接收端");
    //主线程负责窗口时间，子线程负责文件传输
    qDebug()<<"当前主线程线程ID:"<<QThread::currentThreadId();
    //设置默认值
    ui->ip->setText("127.0.0.1");
    ui->port->setText("8989");

    //创建子线程
    subThread = new QThread;
    worker = new RecvFile; //不可指定父对象
    //worker想工作，除了subThread start还需要信号量连接调用workin
    worker->moveToThread(subThread); //移动到子线程中区

    subThread->start(); //子线程开始工作

    connect(this,&MainWindow::startConnect,worker,&RecvFile::connectServer);

    //当子线程连接服务器成功，通知主线程
    connect(worker,&RecvFile::connectOK,this,[=]()
    {
        //通知程序的使用者
        QMessageBox::information(this,"提示","已经成功连接到了服务器");
        ui->connect->setText("已连接");
        ui->connect->setEnabled(false);
    });

    //当子线程接收到数据呈现给父进程，将接受的数据呈现到UI上面
    connect(worker,&RecvFile::message,this,[=](QByteArray msg)
    {
        //修改每次发送的不同颜色
        QVector<QColor> colors =
        {
            Qt::red,Qt::green,Qt::black,Qt::blue,Qt::darkRed,Qt::cyan,Qt::magenta
        };
        int index = QRandomGenerator::global()->bounded(colors.size()); //获取colors里面的随机数
        //取出某一种颜色
        ui->msg->setTextColor(colors.at(index)); //设置一种随机颜色

        ui->msg->append(msg);

    });
    //当子进程结束
    connect(worker,&RecvFile::gameover,this,[=]()
    {
        //子线程运行结束，进行资源的释放
        qDebug()<<"子进程文件上传完毕";
    });

    connect(ui->close,&QPushButton::clicked,this,[=]()
    {
        subThread->quit();
        subThread->wait();
        subThread->deleteLater();
        ui->connect->setEnabled(true);
        ui->connect->setText("连接服务器");
    });

    //当子进程更新总文件大小，设置progressBar
    connect(worker,&RecvFile::total_size_signal,this,[=](qint64 size)
    {
        qDebug()<<"接受文件总大小为"<<size;
        ui->progressBar->setMinimum(0);
        ui->progressBar->setMaximum(static_cast<int>(size));
        ui->progressBar->setValue(0); //设置当前

    });
    connect(worker,&RecvFile::now_size_signal,this,[=](qint64 size)
    {
        //qDebug()<<"接受文件目前大小为"<<size;
        ui->progressBar->setValue(static_cast<int>(size)); //设置当前

    });



}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_connect_clicked()
{

    //拿到IP端口，将Ip和端口传递给子线程，在子线程new QTcp套接字对象，在子线程中接受数据
    QString ip =ui->ip->text(); //读取ip
    unsigned short port = ui->port->text().toUShort();
    //通过信号传递给子线程
    emit startConnect(ip,port);
}


```



#### `recvfile.h`

```cpp
#ifndef RECVFILE_H
#define RECVFILE_H

#include <QObject>
#include <QTcpSocket>
class RecvFile : public QObject
{
    Q_OBJECT
public:
    explicit RecvFile(QObject *parent = nullptr);
    ~RecvFile();

    //连接服务器的函数
    void connectServer(QString ip,unsigned short port);
    void dealData();
signals:
    void connectOK();
    void message(QByteArray msg);
    void gameover();
    void total_size_signal(qint64);
    void now_size_signal(qint64);
public slots:

private:
    QTcpSocket* m_tcp;
    bool first_total = true;  //首次读取文件总大小
    qint64 nowsize =0;
};

#endif // RECVFILE_H

```

#### `recvfile.cpp`

```cpp
#include "recvfile.h"
#include <QHostAddress>
#include <QDebug>
#include <QThread>
#include <QtEndian>
#include <QDataStream>
RecvFile::RecvFile(QObject *parent) : QObject(parent)
{

}

RecvFile::~RecvFile()
{
    //销毁套接字
    m_tcp->close();
    m_tcp->deleteLater();

}

void RecvFile::connectServer(QString ip, unsigned short port)
{
    qDebug()<<"当子线程线程ID:"<<QThread::currentThreadId();
    // 连接服务器
    m_tcp = new QTcpSocket;
    //连接服务器
    m_tcp->connectToHost(QHostAddress(ip),port);
    connect(m_tcp,&QTcpSocket::connected,this,&RecvFile::connectOK); //成功建立连接，发送连接成功对象
    //接受到数据， 服务器发送给客户端数据完整之后
    connect(m_tcp,&QTcpSocket::readyRead,this,[=]()
    {
        //读取文件总大小
        if(first_total)
        {
            first_total = false;
            QByteArray array =  m_tcp->readLine();
            qDebug()<<"array传入文件大小为:"<<array.data();
            emit total_size_signal(array.toInt());
        }
        //QByteArray all = m_tcp->readAll();
//        emit message(all);
        //对数据进行拆包
        else {
            dealData();

            emit gameover();
        }

    });

}

void RecvFile::dealData()
{
    int totalBytes = 0;
    int recvBytes = 0;
    QByteArray block; //存储对应的数据块

    //判断有没有数据
    if(m_tcp->bytesAvailable() == 0)
    {
        qDebug()<<"没有数据咯";
        return ;
    }
    //读包头 可读的字节大于4个字节 //包头只可以读取字节，无法获取总字节


    if(static_cast<int>(m_tcp->bytesAvailable()) >= static_cast<int>(sizeof (int)))
    {

        QByteArray head = m_tcp->read(sizeof(int));
        totalBytes = qFromBigEndian(*reinterpret_cast<int*>(head.data()));
        //qDebug() << "接收方包头的长度: " << totalBytes;
    }
    else {

        return;  //递归 也会结束
    }
    //根据包头去读 数据块
    while(totalBytes-recvBytes >0 &&m_tcp->bytesAvailable() > 0)
    {
        block.append( m_tcp->read(totalBytes-recvBytes));
        recvBytes = block.size();
        nowsize +=recvBytes;
        emit now_size_signal(nowsize);
    }
    if(totalBytes == recvBytes)
    {

        emit message(block);

    }
    //如果还有数据
    if(m_tcp->bytesAvailable() > 0)
    {
        qDebug()<<"开始 递归调用....";
        dealData();
    }

}
```

## 运行`结果`

![运行结果](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsFicNFBiapQDKsSRiahVHibKMxnAsMMAVwjQZ2lqE9GPp83YtWaceJibpibdlvL2ibRMibnbUkPE6KjwP4cQ/640?wx_fmt=png "运行结果")