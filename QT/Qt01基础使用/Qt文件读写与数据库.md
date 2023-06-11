# 文件

## 打开&写入文件&文件信息

![打开文件](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOftbGllPcWQnHjG3xqMwMRuDTj55eEu5CxqdRNX0kfZQcf2oTBEY5Ww/640?wx_fmt=png "打开文件")



```cpp
#include "widget.h"
#include "ui_widget.h"
#include <QFileDialog>  //获取文件
#include <QFile> //打开文件
#include <QFileInfo> //查看文件属性信息
#include <QDebug>
#include <QDateTime>
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    //当按钮被点击之后，
    connect(ui->pushButton, &QPushButton::clicked, [&](){
        //打开文件夹，获取用户点击的文件
       QString filename = QFileDialog::getOpenFileName(this, "打开文件", "E:\\");
       ui->lineEdit->setText(filename); //将文件呈现到lineEdit中

       QFile file(filename); //使用QFile打开点击的文件
       file.open(QIODevice::ReadOnly); //以只读的形式打开
       //QByteArray array = file.readAll();
       QByteArray array; //设置接受数据的QByteArray，字符数组
       //当文件不再最后时，每次读一行
       while(!file.atEnd())
       {
            array += file.readLine();
       }
        //将读到的数据显示到UI中
       ui->textEdit->setText(array);
       file.close();  //关闭文件
        //以添加的方式打开文件
       file.open(QIODevice::Append);
       //写入文件
       file.write("I love you, Rick!");
       file.close();
    });
    //打开此文件
    QFile file("E:/Qian_qt/36.FileInfo/File/widget.cpp");
    QFileInfo info(file); //获取file的文件先关信息
    qDebug() << "绝对路径：" << info.absoluteFilePath();
    qDebug() << "文件名：" << info.fileName();
    qDebug() << "后缀名：" << info.suffix();
    //qDebug() << "创建时间：" << info.created();
    qDebug() << "创建时间：" << info.created().toString("yyyy.MM.dd hh:mm:ss");
    qDebug() << "文件大小：" << info.size();

    file.open(QIODevice::ReadOnly); //以只读的形式打开文件
    file.seek(0); //seek设置偏移量
    QByteArray array = file.read(5); //读5个字节的数据
    qDebug() << "前5个字节： " << array;
    qDebug() << "当前位置： " << file.pos(); //获取当前字节的位置
    file.seek(15); //再偏移15个位子
    array = file.read(5); //再读5个字节的数据
    qDebug() << "第16到第20个字节： " << array;
    file.close();
}

//输出
//绝对路径： "E:/Qian_qt/36.FileInfo/File/widget.cpp"
//文件名： "widget.cpp"
//后缀名： "cpp"
//创建时间： "2023.06.03 08:54:29"
//文件大小： 1915
//前5个字节：  "#incl"
//当前位置：  5
//第16到第20个字节：  "t.h\"\r"
```

## 文件夹

> `#include <QDir>`

![读取文件夹](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYO4W6DwNeqWMAib51WmsNqW9siaNEBd6Qhdu3uJLSBffjQhVhXetqW7gVQ/640?wx_fmt=png "文件夹")

```cpp
//头文件
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QFileSystemWatcher>  //文件夹信息变化函数

namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

private:
    Ui::Widget *ui;
    QFileSystemWatcher myWatcher;  //若对应的文件夹发生变化，会发出信号

private slots:
    void showMessage(const QString &path); //发生变化后的函数
};

#endif // WIDGET_H

//实现函数
#include "widget.h"
#include "ui_widget.h"
#include <QDir>  //获取文件夹的头文件
#include <QDebug>

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    //QDir myDir(QDir::currentPath()); //当前文件夹Debug所在的目录，若release为此
    //qDebug() << myDir;
    QDir myDir(QDir::currentPath());
    qDebug() << myDir;
    ui->listWidget->addItem(myDir.absolutePath());//添加绝对路径
    ui->listWidget->addItems(myDir.entryList()); //将整个文件列表输入到list当中
    //设置名字过滤器，过滤掉.cpp的所有文件
    myDir.setNameFilters(QStringList("*.cpp"));
    ui->listWidget->addItems(myDir.entryList());
    //创建新的文件夹
    myDir.mkdir("mydir");

    connect(&myWatcher, &QFileSystemWatcher::directoryChanged, this, &Widget::showMessage); //当文件目录发生改变
    connect(&myWatcher, &QFileSystemWatcher::fileChanged, this, &Widget::showMessage); //放文件发生了改变
    myWatcher.addPath(QDir::currentPath()); //添加监察的目录
    myWatcher.addPath(QDir::currentPath()+"widget.cpp"); //添加检查的文件
    //
}

Widget::~Widget()
{
    delete ui;
}

void Widget::showMessage(const QString &path)
{
    if(path == QDir::currentPath()) //如果时目录发生了改变
        ui->listWidget->addItem(" 目录发生改变");
    else
        ui->listWidget->addItem("文件发生改变");
}

```

## 文件流

> `文本流`:与当前系统有关
>
> `数据流`: 以`二进制`的形式进行`文件传输`



```cpp
#include "widget.h"
#include "ui_widget.h"
#include <QFileDialog>
#include <QFile>
#include <QTextStream>  //文件流
#include <QDataStream>  //数据流
#include <QDebug>

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    connect(ui->pushButton, &QPushButton::clicked, [&](){
       QString filename = QFileDialog::getOpenFileName(this, "打开文件", "C:\\Users\\jackb");
       ui->lineEdit->setText(filename);

       QFile file(filename);  //打开用户选择的文件
       file.open(QIODevice::ReadOnly); //以只读的形式

       QTextStream stream(&file); //文本流
       stream.setCodec("utf-8");  //设置文件格式，否则会出现中文乱码问题

       //QByteArray array = file.readAll();
       QByteArray array; //字符数组
       while(!stream.atEnd())//while(!file.atEnd())  //当文本流没有达到最后
       {
            array += stream.readLine();//array += file.readLine();
       }
       ui->textEdit->setText(array);
       file.close();

    });

    QFile file("E:\\file.dat"); //打开文件
    file.open(QIODevice::WriteOnly); //以只写的方式
    QDataStream data_stream(&file); //根据此文件创建数据流
    data_stream << QString("Hello world") << static_cast<qint64>(65);  //向文件中输入数据
    file.close();

    file.open(QIODevice::ReadOnly);  //以只读的形式
    QDataStream data_in_stream(&file);
    QString str;
    qint32 n;
    data_in_stream >> str >> n;  //将读取的数据存入str和n当中
    qDebug() << "str is " << str << ", n is " << n; //输出对应的数据
    file.close();

/*
    QImage image("C:\\flower.jpg");  //图片也已数据流的形式报错
    QByteArray arr;
    QDataStream stream(&arr, QIODevice::ReadWrite);
    stream << image;
*/
}
```

# 数据库

> 使用`SQLite3` 轻量级的嵌入式数据库

## 数据库创建

```cpp
//在.pro文件夹加入sql模块
QT       += core gui sql
//函数实现
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>
#include <QSqlDatabase>
#include <QSqlError>

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    QStringList drivers = QSqlDatabase::drivers(); //输出当前Qt支持的数据容器
    //使用foreach将支持的数据库进行输出
    foreach(QString driver, drivers)
        qDebug() << driver;
    //对sqlite1进行数据库丽娜姐
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("dataset.db"); //设置数据库名称，文件将保存保存当前运行的目录
    if(!db.open()) //如果没有打开成功
    {
        qDebug() << "Error: Failed to connect to the database." << db.lastError();
    }
    else
    {
        qDebug() << "Connect database successful!";
    }
}
```

## 数据库插入与查找

```cpp
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>
#include <QSqlDatabase>  //数据库创建头文件
#include <QSqlError> //数据库报错头文件
#include <QSqlQuery> //数据库执行的头文件

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("dataset.db");
    if(!db.open())
    {
        qDebug() << "Error: Failed to connect to the database." << db.lastError();
    }
    else
    {
        qDebug() << "Connect database successful!";
    }

    QSqlQuery sql_query;
    //创建数据库表
    QString create_sql = "create table student(id int, name varchar(30), age int)";
    //准备数据库连接
    sql_query.prepare(create_sql);
    if(!sql_query.exec())
    {
        qDebug() << "Error, Failed to create table." << sql_query.lastError();
    }
    else
        qDebug() << "Table created";
    //插入数据
    QString insert_sql = "insert into student values(?, ?, ?)";
    //数据库执行转呗
    sql_query.prepare(insert_sql);
    //根据三个问号的顺序，依次添加数据
    sql_query.addBindValue(1);
    sql_query.addBindValue("Rick");
    sql_query.addBindValue(10);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
        qDebug() << "Insert Student NO.1!";
    //插入第二条数据
    sql_query.prepare(insert_sql);
    sql_query.addBindValue(2);
    sql_query.addBindValue("Jack");
    sql_query.addBindValue(11);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
        qDebug() << "Insert Student NO.2!";

    //查看数据库student表所有数据
    QString select_all_sql = "select * from student";
    sql_query.prepare(select_all_sql);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError(); //返回数据库表格错误的最新错误
    }
    else
    {
        while(sql_query.next())  //如果还有下一条数据
        {
            int id = sql_query.value(0).toInt();
            QString name = sql_query.value(1).toString();
            int age = sql_query.value(2).toInt();
            qDebug() << "id:" << id << "   name:" << name << "  age:" << age; //将对应的数据输出
        }
    }
    //更新数据库
    QString update_sql = "update student set name = :nm where id = :n";
    sql_query.prepare(update_sql);
    sql_query.bindValue(":nm", "Michael");
    sql_query.bindValue(":n", "1");
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
        qDebug() << "Updated";
	//查询数据库
    select_all_sql = "select * from student";
    sql_query.prepare(select_all_sql);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
    {
        while(sql_query.next())
        {
            int id = sql_query.value(0).toInt();
            QString name = sql_query.value(1).toString();
            int age = sql_query.value(2).toInt();
            qDebug() << "id:" << id << "   name:" << name << "  age:" << age;
        }
    }
	//删除指定的数据库信息
    QString delete_sql = "delete from student where id = ?";
    sql_query.prepare(delete_sql);
    sql_query.addBindValue(1);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
        qDebug() << "Deleted";

    select_all_sql = "select * from student";
    sql_query.prepare(select_all_sql);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
    {
        while(sql_query.next())
        {
            int id = sql_query.value(0).toInt();
            QString name = sql_query.value(1).toString();
            int age = sql_query.value(2).toInt();
            qDebug() << "id:" << id << "   name:" << name << "  age:" << age;
        }
    }
	//删除数据库表
    QString clear_sql = "delete from student";
    sql_query.prepare(clear_sql);
    if(!sql_query.exec())
    {
        qDebug() << sql_query.lastError();
    }
    else
        qDebug() << "Table cleared";

}
Widget::~Widget()
{
    delete ui;
}
```

### 用户接口层

> 用户可以直接通过`UI`的形式`直接`操作`数据库`



```cpp
//头文件
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
//用户接口层所需要头文件
#include <QSqlQueryModel>
#include <QSqlTableModel>//tablemodel
#include <QSqlRelationalTableModel>
#include <QTableView>  // QTable的文件

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
    void on_pushButton_clicked();

    void on_pushButton_2_clicked();

    void on_pushButton_3_clicked();

private:
    Ui::Widget *ui;
    QSqlTableModel *model;
};

#endif // WIDGET_H
```




```cpp
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("dataset.db");
    if(!db.open())
    {
        qDebug() << "Error: Failed to connect to the database." << db.lastError();
    }
    else
    {
        qDebug() << "Connect database successful!";
    }
    QSqlQuery sql_query;
        //创建model模型
    model = new QSqlTableModel(this);
    //将model模型应用到Table里面
    ui->tableView->setModel(model);
    //设置model绑定的数据名称
    model->setTable("student");
    //执行selece语句
    model->select();
    //对Table的头标题进行名称初始化
    model->setHeaderData(0, Qt::Horizontal, "学号");
    model->setHeaderData(1, Qt::Horizontal, "姓名");
    model->setHeaderData(2, Qt::Horizontal, "年龄");
    //设置数据智能手动提交修改
    //若不设置此句，在table修改数据将自动更新数据库
    model->setEditStrategy(QSqlTableModel::OnManualSubmit);
}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_pushButton_clicked()
{
    //提交信息到数据库
    model->submitAll();
}

void Widget::on_pushButton_2_clicked()
{
    //撤销数据库更新
    model->revertAll();
    //提交数据库
    model->submitAll();
}

void Widget::on_pushButton_3_clicked()
{
    //选择数据
    QString name = ui->lineEdit->text(); //读取用户输入的数据
    QString nm = QString("name = '%1'").arg(name);
    //设置过滤条件
    model->setFilter(nm);  //  name =  'Jack'
    //model->setFilter("name =  'Jack'");
    //执行select语句
    model->select();
}
```