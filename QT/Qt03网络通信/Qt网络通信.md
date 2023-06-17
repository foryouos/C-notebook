# TCP通信

> Qt需要`network`模块实现套接字通信，在pro文件中提前加入。所有通信函数与C++封装TCP通信类似。

## 通信与函数

![TCP通信函数](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbuAZEWbsPKptycVwDzZkVekqHGlibgmve07tfGR0CuIoicEUtbA8aPcnVEloaZYglROleE906pz4yQA/640?wx_fmt=jpeg "函数与信号")

## TCP通信案例

### 服务端

#### `UI`

![服务端](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibiaFCiaSicYQs14pMl4ic8NPQ5okzfXH69CdSF4aH3dCNV2N32hmiaRia0ZcA/640?wx_fmt=png "服务端")

#### `server.h`

```cpp
#ifndef SERVER_H
#define SERVER_H

#include <QMainWindow>
#include <QTcpSocket>
#include <QTcpServer>

namespace Ui {
class server;
}

class server : public QMainWindow
{
    Q_OBJECT

public:
    explicit server(QWidget *parent = nullptr);
    ~server();

private slots:
    void on_startServer_clicked();

    void on_sendMsg_clicked();

private:
    Ui::server *ui;
    QTcpServer* m_server;
    QTcpSocket* m_tcp;
};

#endif // SERVER_H

```



#### `server.cpp`

```cpp
#include "server.h"
#include "ui_server.h"

server::server(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::server)
{
    ui->setupUi(this);
    setWindowTitle("TCP - 服务器");
    //创建QTcpServer对象
    m_server = new QTcpServer(this);
    //检测是否有新链接
    connect(m_server,&QTcpServer::newConnection,this,[=]()
    {
       m_tcp = m_server->nextPendingConnection(); //获取TCP连接
       ui->m_status->setText("已连接");
       ui->records->append("成功和客户端建立了新连接");
       //客户端发送过来了数据
       connect(m_tcp,&QTcpSocket::readyRead,this,[=]()
       {
           //接受数据
           QString recvMsg=m_tcp->readAll();
           ui->records->append("客户端say:" + recvMsg);
           ui->records->setAlignment(Qt::AlignRight);
       });
       //客户端断开了连接
       connect(m_tcp,&QTcpSocket::disconnected,this,[=]()
       {
            ui->records->append("客户端已经断开了连接");
            m_tcp->deleteLater();
            ui->m_status->setText("已断开");
       });

    });
}

server::~server()
{
    delete ui;
}

void server::on_startServer_clicked()
{
    unsigned short port = ui->port->text().toInt();
    //设置服务器监听
    m_server->listen(QHostAddress::LocalHost,port);
    ui->startServer->setEnabled(false);
}

void server::on_sendMsg_clicked()
{
    QString sendMsg = ui->msg->toPlainText();
    m_tcp->write(sendMsg.toUtf8());
    ui->records->append("服务器Say:" + sendMsg);
    ui->records->setAlignment(Qt::AlignLeft);
    ui->msg->clear();
}
```



### 客户端

#### `UI`

![客户端](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibZSwtmHCDW7ib4Kbwdyr3tyJPDWlYApSibbaBSvlmsiczRiccuoTWjicVvhg/640?wx_fmt=png "客户端")

#### `client.h``

```cpp
#ifndef CLIENT_H
#define CLIENT_H

#include <QMainWindow>
#include <QTcpSocket>
namespace Ui {
class client;
}

class client : public QMainWindow
{
    Q_OBJECT

public:
    explicit client(QWidget *parent = nullptr);
    ~client();

private slots:
    void on_connect_clicked();

    void on_close_clicked();

    void on_send_clicked();

private:
    Ui::client *ui;
    QTcpSocket* m_tcp;
};

#endif // CLIENT_H
```



#### `client.cpp`

```cpp
#include "client.h"
#include "ui_client.h"

client::client(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::client)
{
    ui->setupUi(this);
    setWindowTitle("TCP - 客户端");
    //创建通信的套接字对象
    ui->ip->setText("127.0.0.1");
    m_tcp = new QTcpSocket(this);
    ui->close->setEnabled(false);
    //检测是否和服务器连接成功
    connect(m_tcp,&QTcpSocket::connected,this,[=]()
    {
        ui->rec->append("恭喜，服务器连接成功!");
        ui->status->setText("已连接");
    });
    //检测服务器是都恢复了数据
    connect(m_tcp,&QTcpSocket::readyRead,[=]()
    {
        QByteArray recvMsg = m_tcp->readAll();
        ui->rec->append("服务器say:"+recvMsg);
        ui->rec->setAlignment(Qt::AlignRight);
    });
    //检测服务器是否断开了连接
    connect(m_tcp,&QTcpSocket::disconnected,this,[=]()
    {
        ui->rec->append("服务器已经断开了连接");
        ui->connect->setEnabled(true);
        ui->close->setEnabled(false);
    });
}

client::~client()
{
    delete ui;
}

void client::on_connect_clicked()
{
    QString ip = ui->ip->text();
    unsigned short port = ui->port->text().toInt();
    //连接服务器
    m_tcp->connectToHost(ip,port);
    ui->connect->setEnabled(false);
    ui->close->setEnabled(true);
}

void client::on_close_clicked()
{
    m_tcp->close();
    ui->connect->setEnabled(true);
    ui->close->setEnabled(false);
}

void client::on_send_clicked()
{
    QString sendMsg = ui->msg->toPlainText();
    m_tcp->write(sendMsg.toUtf8());
    ui->rec->append("客户端Say:"+sendMsg);
    ui->rec->setAlignment(Qt::AlignLeft);
    ui->msg->clear();
}
```



![TCP通信](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibxwVvib82pHXYUD401Bn7CJpONrOPY47AfMbXA8ne7Uyg3cJrGjI9Zzg/640?wx_fmt=png "TCP通信")

# UDP通信

> Qt的客户端和服务端不区分，实现双向通信，彼此即是服务端，也是接收端

![UDP通信流程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibvZhnlh8sbDNJcWUAE9zPQxODcfuKwJbqaMjrhVCWhDRWn8QvFLUYJA/640?wx_fmt=jpeg "UDP通信流程")

## `UDP`通信案例

### 服务端

#### `UI`

![服务端UI](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibTrQBLmG6Kd9llQ6wic5iavEW5Yt7X1PCjv8QBIriaejLYkP0N9jEUTiaBw/640?wx_fmt=png "服务端UI")

#### `server.h`

```cpp
#ifndef UDF_SERVER_H
#define UDF_SERVER_H

#include <QMainWindow>
#include <QUdpSocket>
namespace Ui {
class udf_server;
}

class udf_server : public QMainWindow
{
    Q_OBJECT

public:
    explicit udf_server(QWidget *parent = nullptr);
    ~udf_server();

private slots:
    void on_send_clicked();

    void on_connect_clicked();

private:
    Ui::udf_server *ui;
    QUdpSocket* server;
};

#endif // UDF_SERVER_H

```



#### `server.cpp`

```cpp
#include "udf_server.h"
#include "ui_udf_server.h"

udf_server::udf_server(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::udf_server)
{
    ui->setupUi(this);
    setWindowTitle("UDP 服务器");

    server = new QUdpSocket(this);
    ui->ip->setText("127.0.0.1");

    ui->port_self->setText("8888"); //当前主机绑定端口号

    //读取对方发送的数据
    connect(server,&QUdpSocket::readyRead,this,[&]()
    {
        QByteArray datagram;
        datagram.resize(server->pendingDatagramSize());
        server->readDatagram(datagram.data(),datagram.size()); //读数据
        ui->recev->append("对方:"+datagram);
    });
}

udf_server::~udf_server()
{
    delete ui;
}

void udf_server::on_send_clicked()
{
    //发送环节设置
    //读取要发送的内容
    QByteArray datagram = ui->send_text->toPlainText().toUtf8();
    //发送的信息，大小，IP和端口
    server->writeDatagram(datagram.data(),datagram.size(),QHostAddress(ui->ip->text()),ui->port->text().toInt());
    ui->recev->append("本机" + ui->send_text->toPlainText());
    ui->send_text->clear();
}

//启动服务器
void udf_server::on_connect_clicked()
{
    //当前服务端绑定端口号
    server->bind(QHostAddress(ui->ip->text()),ui->port_self->text().toInt()); //设置当前主机绑定的端口
    ui->connect->setEnabled(false);
}

```



### 客户端

#### UI

![客户端UI](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibNgoiceVXrHYrN7v6b2bfP0rpRticlZrFS5lTN21z4zR9sgyZhsTibIiaEw/640?wx_fmt=png "客户端UI")

#### client.h

```cpp
#ifndef UDF_CLIENT_H
#define UDF_CLIENT_H

#include <QMainWindow>
#include <QUdpSocket>
namespace Ui {
class udf_client;
}

class udf_client : public QMainWindow
{
    Q_OBJECT

public:
    explicit udf_client(QWidget *parent = nullptr);
    ~udf_client();

private slots:
    void on_connect_clicked();

    void on_send_clicked();

private:
    Ui::udf_client *ui;
    QUdpSocket * recever;
};

#endif // UDF_CLIENT_H

```



#### `client.cpp`

```cpp
#include "udf_client.h"
#include "ui_udf_client.h"

udf_client::udf_client(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::udf_client)
{
    ui->setupUi(this);
    setWindowTitle("UDP 绑定IP");
    ui->ip->setText("127.0.0.1");
    recever = new QUdpSocket(this);



}

udf_client::~udf_client()
{
    delete ui;
}

void udf_client::on_connect_clicked()
{
    //点击连接之后，进入接受状态
    recever->bind(QHostAddress(ui->ip->text().toInt()),ui->port->text().toInt()); //绑定IP
    connect(recever,&QUdpSocket::readyRead,this,[&]()
    {
        QByteArray datagram;
        datagram.resize(recever->pendingDatagramSize());
        recever->readDatagram(datagram.data(),datagram.size()); //读数据
        ui->rev->append("对方:"+datagram);
    });
    ui->connect->setEnabled(false);
    //ui->rev->setEnabled(false);
}

void udf_client::on_send_clicked()
{
    //发送环节设置
    //读取要发送的内容
    QByteArray datagram = ui->send_test->toPlainText().toUtf8();
    //发送的信息，大小，IP和端口
    recever->writeDatagram(datagram.data(),datagram.size(),QHostAddress(ui->ip->text()),ui->port_other->text().toInt());
    ui->rev->append("本机" + ui->send_test->toPlainText());
    ui->send_test->clear();
}

```



![UDP通信](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtib0vbgtA8ZD7nZLIxTLlVA72QKn4ZZRwo9K7ibicVCUQtM7tRmRrDCKE1Q/640?wx_fmt=png "UDP通信")

# TCP文件传输

> 客户端连接服务器，可以实现多种文件向服务器进行发送

## 服务端`接收端

### `UI`

![服务端UI](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtib10LicwbJr6JAkkX4AXiahU4zMfqtdjicrZ8ic4SoEicCnLX8d3swGawFeGg/640?wx_fmt=png "服务端UI")

### `server.h`

```cpp
#ifndef SERVER_H
#define SERVER_H

#include <QMainWindow>
#include <QTcpSocket>
#include <QTcpServer>
#include <QFile>
#include <QTimer>
#include <QFileDialog>
#include <QFileInfo>
#include <QDebug>
#include <QMessageBox>
namespace Ui {
class server;
}

class server : public QMainWindow
{
    Q_OBJECT

public:
    explicit server(QWidget *parent = nullptr);
    ~server();

private slots:
    void on_startServer_clicked();

private:
    Ui::server *ui;
    QTcpServer* m_server;
    QTcpSocket* m_tcp;
    bool headinfo = true;
    QString filename;
    qint64 filesize;
    qint64 recvsize;
    QFile file;
};

#endif // SERVER_H

```



### `server.cpp`

```cpp
#include "server.h"
#include "ui_server.h"

server::server(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::server)
{
    ui->setupUi(this);
    setWindowTitle("TCP - 服务器接受文件");
    //创建QTcpServer对象
    m_server = new QTcpServer(this);
    //客户端断开了连接
//    connect(m_tcp,&QTcpSocket::disconnected,this,[=]()
//    {
//         m_tcp->deleteLater();
//         ui->m_status->setText("已断开");
//    });

}

server::~server()
{
    delete ui;
}

void server::on_startServer_clicked()
{
    unsigned short port = ui->port->text().toInt();
    //设置服务器监听
    m_server->listen(QHostAddress::LocalHost,port);

    //检测是否有新链接
    connect(m_server,&QTcpServer::newConnection,this,[&]()
    {
       m_tcp = m_server->nextPendingConnection(); //获取TCP连接
       ui->m_status->setText("已连接");
       //客户端发送过来了数据
       connect(m_tcp,&QTcpSocket::readyRead,this,[&]()
       {
            //读取数据
           QByteArray array = m_tcp->readAll();
           if(headinfo)
           {
               headinfo = false;
               recvsize = 0;
               filename = QString(array).section("**",0,0); //起始，结束
               filesize = QString(array).section("**",1,1).toInt();

               file.setFileName(filename); //设置文件

               file.open(QIODevice::WriteOnly);//只读的方式打开文件
               //设置传输的进度条
               ui->progressBar->setMinimum(0);
               ui->progressBar->setMaximum(filesize / 1024);
               ui->progressBar->setValue(0);  //设置当前值为0

           }
           else
           {
               //处理数据文件
                qint64 length = file.write(array);
                if(length > 0)
                {
                    recvsize +=length;
                }
                ui->progressBar->setValue(recvsize / 1024);
                qDebug() <<"接收大小"<< recvsize <<"文件大小"<< filesize;
                if(recvsize == filesize)
                {

                    QMessageBox::information(this,"完成","文件接受完成");
                    file.close();
                }

           }

       });
    });

}


```



## 客户端``发送端``

### `UI`

![客户端UI](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtib1CTW28SQjicTxliaYdYcuGZRicesk71XAPvVznIBvib3UFCHMRufB7IibRQ/640?wx_fmt=png "客户端UI")

### `client.h`

```cpp
#ifndef CLIENT_H
#define CLIENT_H

#include <QMainWindow>
#include <QTcpSocket>
#include <QFile>
#include <QTimer>
namespace Ui {
class client;
}

class client : public QMainWindow
{
    Q_OBJECT

public:
    explicit client(QWidget *parent = nullptr);
    ~client();

private slots:
    void on_connect_clicked();

    void on_close_clicked();

    void on_send_clicked();

    void on_open_clicked();

private:
    Ui::client *ui;
    QTcpSocket* m_tcp;
    QString filename;
    qint64 filesize;
    QFile file;
    QTimer *myTimer;
};

#endif // CLIENT_H

```

### `client.cpp`

```cpp
#include "client.h"
#include "ui_client.h"
#include <QFileDialog>
#include <QFileInfo>
#include <QDebug>
client::client(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::client)
{
    ui->setupUi(this);
    setWindowTitle("TCP - 客户端上传文件");
    //创建通信的套接字对象
    ui->ip->setText("127.0.0.1");
    m_tcp = new QTcpSocket(this);
    ui->close->setEnabled(false);
    //检测是否和服务器连接成功
    connect(m_tcp,&QTcpSocket::connected,this,[=]()
    {
        //ui->rec->append("恭喜，服务器连接成功!");
        ui->status->setText("已连接");
        ui->connect->setEnabled(false);
        ui->close->setEnabled(true);
    });
    //检测服务器是都恢复了数据
//    connect(m_tcp,&QTcpSocket::readyRead,[=]()
//    {
//        QByteArray recvMsg = m_tcp->readAll();
//        ui->rec->append("服务器say:"+recvMsg);
//        ui->rec->setAlignment(Qt::AlignRight);
//    });
    //检测服务器是否断开了连接
    connect(m_tcp,&QTcpSocket::disconnected,this,[=]()
    {
        //ui->rec->append("服务器已经断开了连接");
        ui->status->setText("未连接");
        ui->connect->setEnabled(true);
        ui->close->setEnabled(false);
    });
    myTimer = new QTimer(this);
    connect(myTimer,&QTimer::timeout,this,[&]()
    {
        myTimer->stop(); //关闭定时器
        qint64 sendSize =0;
        qint64 len;
        do{
            char buf[4*1024] = {0};
            len = file.read(buf,sizeof(buf)); //每次读取的空间
            m_tcp->write(buf,len); //读取的文件缓冲区放到buf里面
            sendSize +=len;
            ui->progressBar->setValue(sendSize / 1024);
        }while(len >0);

        if(sendSize == filesize)
        {
            qDebug()<<"文件发送成功";
            file.close();
            m_tcp->disconnectFromHost();
            m_tcp->close();
        }

    });

}

client::~client()
{
    delete ui;
}

void client::on_connect_clicked()
{
    QString ip = ui->ip->text();
    unsigned short port = ui->port->text().toInt();
    //连接服务器
    m_tcp->connectToHost(ip,port);

}

void client::on_close_clicked()
{
    m_tcp->close();
    ui->connect->setEnabled(true);
    ui->close->setEnabled(false);
}

void client::on_send_clicked()
{
    //点击 开始之后，发送数据，先发送头名称和大小
    // 文件名**文件大小
    QString head = filename + "**" + QString::number(filesize);
    qint64 length  = m_tcp->write(head.toUtf8()); //将头文件发送过去
    if(length > 0)
    {
        qDebug()<<length;
        //发送成功
        myTimer->start(20); //20毫秒之后再发，避免粘包
    }
    else
    {
        //发送失败
        file.close();//关闭文件
    }
}

void client::on_open_clicked()
{
    QString filepath = QFileDialog::getOpenFileName(this);
    //同时发送文件名称和文件大小
    QFileInfo FileData(filepath);
    filename = FileData.fileName(); //文件名称
    filesize = FileData.size(); //文件大小
    qDebug()<<"文件名"<<filename <<"  文件大小:"<<filesize;

    if(!filepath.isEmpty())
    {
        ui->file->setText(filename);
        file.setFileName(filepath); //设置文件
        file.open(QIODevice::ReadOnly);//只读的方式打开文件
    }
    //设置传输的进度条
    ui->progressBar->setMinimum(0);
    ui->progressBar->setMaximum(filesize / 1024);
    ui->progressBar->setValue(0);  //设置当前值为0

}
```

![文件发送](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsjDYUbUeUpnsDUlSjZPqtibgMYQmNTMC0lfqK17Y88at1NMYdo0xOduHodoWWOv03HjID0p5G8KYA/640?wx_fmt=png "文件发送")

