# 串口
> 串口: 一种计算即通信接口，通过穿行传输数据，用于将计算机与外部设备(例如打印机，调制解调器，传感器等)连接起来

* `QSerialPort`：Qt中的`串口`类，用于实现串口通信功能。
* `QSerialPortInfo`：Qt中的`串口信息类`，用于获取系统中`可用`的串口信息。
* 波特率（`Baud Rate`）：表示`每秒钟传输的比特数`，也就是串口传输数据的速度。在Qt中，可以使用``QSerialPort::setBaudRate()`函数设置`波特率。`
* 数据位（Data Bits）：表示`每个字符使用的比特数`，通常为7或8位。在Qt中，可以使用`QSerialPort::setDataBits()`函数设置`数据位`。
* 停止位（Stop Bits）：表示在每个字符传输结束后，发送端在传输线上保持的电平状态。通常为`1或2位`。在Qt中，可以使用`QSerialPort::setStopBits()`函数设置`停止位`。
* 校验位（Parity Bit）：用于`检查数据传输是否出错`，常用的校验方式有`奇偶校验、偶校验和无校验`。如果使用校验位，通常为`1位`。在Qt中，可以使用`QSerialPort::setParity()`函数设置校验位。
* 数据流控制（`Flow Control`）：用于控制数据传输的速度，常用的方式有硬件流控制和软件流控制。在Qt中，可以使用QSerialPort::setFlowControl()函数设置数据流控制。硬件流控制通常使用`RTS`（请求发送）和`CTS`（清除发送）`控制信号`，`软件流控制`通常使用`XON`和`XOFF`字符。
* 串口缓冲区（Serial Buffer）：用于存储串口接收和发送的数据，Qt中提供了`QSerialPort::read()`和`QSerialPort::write()`函数用于读取和发送数据。
* 帧（`Frame`）：表示一个数据包，通常包含起始位、数据位、校验位和停止位。在Qt中，可以使用`QSerialPort::setFrameSize()`函数设置帧的长度。
* 串口状态（`Serial Port Status`）：用于表示串口的`状态`，包括`是否打开`、`是否有可用数据`等。在Qt中，可以使用`QSerialPort::isOpen()`函数判断串口`是否打开`，使用`QSerialPort::bytesAvailable()`函数获取`可用数据的字节数。
## UI

![串口调试UI](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsMW39IOCFlyaIVRyBjA9Z0tBicdvLdozkyLrEib7E1OVV3opeNnUicWBibicxv1qRvTEr9AC1uTOAS4LQ/640?wx_fmt=png "串口调试UI")

## 头文件
```cpp
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QtSerialPort/QtSerialPort>
#include <QTimer>

namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = nullptr);
    ~Widget();

private slots:
    void TimerEvent();  //定时器实现

    void on_pushButton_clicked();
    void serialPort_readyRead(); //接受数据

    void on_checkBox_clicked();

    void on_checkBox_2_clicked();

    void on_checkBox_3_clicked();

    void on_pushButton_2_clicked();

    void on_pushButton_3_clicked();

    void on_pushButton_4_clicked();

private:
    Ui::Widget *ui;
    QTimer *timer;//定时器
    QStringList portStringList; //串口列表
    QSerialPort *serial;  //创建串口
    QString Receivetext, Sendtext;//接受的数据和发送的数据
    long Receive_Byte, Send_Byte; //接受的总字节数和发送的总字节数
};

#endif // WIDGET_H
```
## 实现函数

```cpp
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>
#include <QMessageBox>

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    //窗口大小
    resize(800, 600);
    setWindowTitle("串口调试助手"); //串口标题

    //设置定时器
    timer = new QTimer(this);

    timer->start(500);  //500毫秒
    //扫描有没有可以用的串口端口
    connect(timer, &QTimer::timeout, this, &Widget::TimerEvent);


    serial = new QSerialPort(this);  //初始化串口

    //对相关数据进行默认处理
    ui->comboBox_2->setCurrentIndex(5);
    ui->comboBox_3->setCurrentIndex(3);
    ui->comboBox_4->setCurrentIndex(2);
    ui->comboBox_5->setCurrentIndex(0);

    //初始化数据清零操作
    Receive_Byte = 0; //记录接收数据的字节数
    Send_Byte = 0; //记录发送数据的字节数

    //信号连接接受数据
    //当serial收到数据，readRead接受数据就绪，之后调用读取接受到的数据
    connect(serial, &QSerialPort::readyRead, this, &Widget::serialPort_readyRead);

    ui->checkBox->setCheckState(Qt::Checked); //默认ACSII码是被选中的
}

Widget::~Widget()
{
    delete ui;
}

void Widget::TimerEvent()
{
    //更新可用的端口
    QStringList newPortStringList;
    newPortStringList.clear();
    //遍历可用的串口
    //将串口放到可用的容器里面
    foreach(const QSerialPortInfo &info, QSerialPortInfo::availablePorts())
        newPortStringList += info.portName(); //info.portName为QString类型

    //如果串口数据有更新，
    if(newPortStringList.size() != portStringList.size())
    {
        portStringList = newPortStringList; //将串口设置成现在的
        ui->comboBox->clear(); //清空之前的串口Box
        ui->comboBox->addItems(portStringList);  //添加现在有的串口
    }
}

void Widget::on_pushButton_clicked()  //打开串口之后的操作
{
    if(ui->pushButton->text() == QString("打开串口"))
    {
        //
        serial->setPortName(ui->comboBox->currentText()); //获取串口号
        serial->setBaudRate(ui->comboBox_2->currentText().toInt()); //获取波特率转成Int
        switch(ui->comboBox_3->currentText().toInt()) //波特率 //根据数据为执行相关操作
        {
            case 5:serial->setDataBits(QSerialPort::Data5);break;
            case 6:serial->setDataBits(QSerialPort::Data6);break;
            case 7:serial->setDataBits(QSerialPort::Data7);break;
            case 8:serial->setDataBits(QSerialPort::Data8);break;
            default:serial->setDataBits(QSerialPort::UnknownDataBits); //不知道波特率
        }
        switch(ui->comboBox_4->currentIndex())  //校验位
        {
            case 0:serial->setParity(QSerialPort::EvenParity);break; //设置校验位
            case 1:serial->setParity(QSerialPort::OddParity);break;
            case 2:serial->setParity(QSerialPort::NoParity);break;
            default:serial->setParity(QSerialPort::UnknownParity); //不知道波特率
        }
        switch(ui->comboBox_5->currentIndex())  //停止位
        {
            case 0:serial->setStopBits(QSerialPort::OneStop);break;
            case 1:serial->setStopBits(QSerialPort::OneAndHalfStop);break;
            case 2:serial->setStopBits(QSerialPort::TwoStop);break;
            default:serial->setStopBits(QSerialPort::UnknownStopBits);
        }

        serial->setFlowControl(QSerialPort::NoFlowControl);

        if(!serial->open(QIODevice::ReadWrite))//打开串口
        {
            QMessageBox::information(this, "错误提示", "无法打开串口", QMessageBox::Ok);
            return;
        }
        //打开串口之后，让相关设置改为不可修改
        ui->comboBox->setEnabled(false);
        ui->comboBox_2->setEnabled(false);
        ui->comboBox_3->setEnabled(false);
        ui->comboBox_4->setEnabled(false);
        ui->comboBox_5->setEnabled(false);

        ui->pushButton->setText("关闭串口");
    }
    else  //如果是关闭串口，用户关闭串口
    {
        //关闭串口之后，可以修改相关设备
        serial->close();
        ui->comboBox->setEnabled(true);
        ui->comboBox_2->setEnabled(true);
        ui->comboBox_3->setEnabled(true);
        ui->comboBox_4->setEnabled(true);
        ui->comboBox_5->setEnabled(true);

        ui->pushButton->setText("打开串口");
    }
}

void Widget::serialPort_readyRead() //接受数据函数实现
{
    QString last_text; //接受数据的存储位置
    int length;
    int i;
    if(ui->checkBox_3->checkState() != Qt::Checked)  //判断接受设置是否是暂停，如果是暂停则不进行相关操作
    {
        last_text = ui->textEdit->toPlainText();  //获取当前接受框的数据
        Receivetext = serial->readAll(); //读取接受到的数据readAll()
        Receive_Byte += Receivetext.length(); //原有的数据长度加上新接受的数据长度.length
        ui->label_9->setText(QString::number(Receive_Byte)); //在下面的接受字节更新接受数据长度QString::number将数字转换为字符串
        //判断接受数据设置是使用HExo还是ACSII并进行分别处理
        if(ui->checkBox_2->checkState() == Qt::Checked)  //hex
        {
            Receivetext = Receivetext.toLatin1().toHex();  //将收到的数据转换为16进制的代码
            length = Receivetext.length(); //获取当前的数据长度
            for(i = 0; i <= length/2; i++) //两个一组，总长度/2
            {
                //每两个字符后面添加空格，添加空格后次序为 2 5 8 11 为2+3*i
                Receivetext.insert(2+3*i, ' ');  //在数据背后添加空格
            }
        }
        else  //ASCII
        {
            Receivetext = Receivetext.toLatin1(); //toLatin1将接受的ASCII字符数组转换为QString
        }
        last_text = last_text.append(Receivetext);  //将原有的数据后面加上新接受到的数据append追加
        ui->textEdit->setText(last_text);

    }
}

//对接受设置只能选中一个，当选中一个，其它参数不能选中
//TODO 单选可解决！！！
void Widget::on_checkBox_clicked()
{
    ui->checkBox->setCheckState(Qt::Checked);
    ui->checkBox_2->setCheckState(Qt::Unchecked);
    ui->checkBox_3->setCheckState(Qt::Unchecked);
}

void Widget::on_checkBox_2_clicked()
{
    ui->checkBox->setCheckState(Qt::Unchecked);
    ui->checkBox_2->setCheckState(Qt::Checked);
    ui->checkBox_3->setCheckState(Qt::Unchecked);
}

void Widget::on_checkBox_3_clicked()
{
    ui->checkBox->setCheckState(Qt::Unchecked);
    ui->checkBox_2->setCheckState(Qt::Unchecked);
    ui->checkBox_3->setCheckState(Qt::Checked);
}

//点击发送之后，发送数据
void Widget::on_pushButton_2_clicked()
{
    QByteArray bytearray;  //定义字符

    Sendtext = ui->textEdit_2->toPlainText();  //捕获读取发送框的数据QString类型
    bytearray = Sendtext.toLatin1(); //将字符串转换为QByteArray类型
    serial->write(bytearray);  //写入
    Receive_Byte += Sendtext.length();  //更新写入长度
    ui->label_11->setText(QString::number(Receive_Byte)); //将写入的字节呈现到下面
}
//清空接收区
void Widget::on_pushButton_3_clicked()
{
    ui->textEdit->clear();
}
//清空发送区
void Widget::on_pushButton_4_clicked()
{
    ui->textEdit_2->clear();
}

```