# 事件
> 事件时系统或者`Qt`本身在不同的场景下发出的。当用户按钮/移动鼠标，敲下键盘，或者关闭窗口/大小发生变化/隐藏或显示都会发出一个相应的事件。基于窗口的应用程序都是基于事件,其目的主要用来实现回调。事件发生经过:`事件派发->事件过滤->事件分发->事件处理`.`event`事件派发到指定窗口，`eventFilter`事件过滤事件，`enterEvent`事件分发，`mouseMoveEvent`具体的时间处理
> 每个Qt都对应唯一的`QApplication`应用程序，然后调用这个对象`exec()函数`-程序将进入事件循环来`监听应用程序事件`
```cpp
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow* w = new MainWindow;
    w.show();
    return a.exec();
}
```

## 鼠标事件

```cpp
// 鼠标按下左键右键中键被按下，该函数自动调用
[virtual protected] void QWidget::mousePressEvent(QMouseEvent *event);
// 鼠标释放
[virtual protected] void QWidget::mouseReleaseEvent(QMouseEvent *event);
// 鼠标移动
[virtual protected] void QWidget::mouseMoveEvent(QMouseEvent *event);
// 鼠标双击事件
[virtual protected] void QWidget::mouseDoubleClickEvent(QMouseEvent *event);
//当鼠标进入窗口一瞬间，触发该事件只在进入的瞬间触发一次该事件
[virtual protected] void QWidget::enterEvent(QEvent *event);
//鼠标离开事件,当鼠标离开窗口的一瞬间，触发该事件只在离开的瞬间触发一次该事件
[virtual protected] void QWidget::leaveEvent(QEvent *event);

```
### 鼠标事件的使用
* 鼠标点击(局部/全局坐标)
* 鼠标释放
* 鼠标双击
* 鼠标实现窗体移动
* 滚轮放大输入框的文字

### 头文件
```cpp
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QMouseEvent>
namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = nullptr);
    ~Widget();
protected:
    void mousePressEvent(QMouseEvent* event);//鼠标点击事件
    void mouseReleaseEvent(QMouseEvent* event); //鼠标释放事件
    void mouseDoubleClickEvent(QMouseEvent* event); //鼠标双击事件
    void mouseMoveEvent(QMouseEvent* event); //鼠标移动 //实现移动窗口的效果
    //滚轮方法输入的文字
    void wheelEvent(QWheelEvent *event);

private:
    Ui::Widget *ui;
    QPoint pos;
};

#endif // WIDGET_H
```
### 函数实现
```cpp
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    connect(ui->pushButton,&QPushButton::clicked,[=]()
    {
        qDebug()<<ui->widget->getValue(); //获得当前的值，点击之后
    });

}

Widget::~Widget()
{
    delete ui;
}

void Widget::mousePressEvent(QMouseEvent *event)
{
    if(event->button() == Qt::LeftButton) //如果点击左键
    {
        qDebug()<<"左键局部坐标:"<<event->x()<<event->y();
    }
    else if (event->button() == Qt::RightButton)
    {
        qDebug()<<"右键全局坐标:"<<event->globalX()<<event->globalY(); //线束全局坐标
    }
}

void Widget::mouseReleaseEvent(QMouseEvent *event) //鼠标释放
{
    Q_UNUSED(event);
    qDebug()<<"Button is released";
}

void Widget::mouseDoubleClickEvent(QMouseEvent *event)
{
    //鼠标双击
    if(event->button() == Qt::LeftButton)//左键双击
    {
        //判断是否全屏
        if(windowState()!=Qt::WindowFullScreen)
        {
            //设置全屏
            setWindowState(Qt::WindowFullScreen);
        }
        else {
            //若已经全屏，再双击会到初始状态
            setWindowState(Qt::WindowNoState);
        }
    }
}

void Widget::mouseMoveEvent(QMouseEvent *event)
{
    if(event->buttons() & Qt::LeftButton)
    {
        QPoint tmp;
        tmp = event->globalPos() - pos; //鼠标当前的位置-窗口所在的位置
        move(tmp); //移动这个差值
    }
}

void Widget::wheelEvent(QWheelEvent *event)
{
    if(event->delta() > 0) //向上滚动
    {
        ui->textEdit->zoomOut();  //字体放大
    }
    else {
        ui->textEdit->zoomIn(); //字体缩小
    }
}
```

## 键盘事件

```cpp
//键盘按下事件，并通过参数得知按下的键
[virtual protected] void QWidget::keyPressEvent(QKeyEvent *event);
//键盘释放事件，该参数被自动调用，通过参数通知那个键
[virtual protected] void QWidget::keyReleaseEvent(QKeyEvent *event);
```

## 窗口事件

```cpp
//当窗口需要刷新时，该函数就会被自动调用：窗口大小，显示等
[virtual protected] void QWidget::paintEvent(QPaintEvent *event);
//窗口标题栏的关闭按钮被按下，窗口关闭之前该函数被调用
[virtual protected] void QWidget::closeEvent(QCloseEvent *event);
//当窗口大小发生变化，该函数被调用，重置窗口大小事件
[virtual protected] void QWidget::resizeEvent(QResizeEvent *event);
```

## 重写事件处理函数

> 事件处理函数都是虚函数，通过添加标砖窗口类的派生类，使子类继承父类的属性，同时在这个子类中重写父类的虚函数

```cpp
//头文件中
protected:
    // 重写事件处理器函数
    void closeEvent(QCloseEvent* ev);
    void resizeEvent(QResizeEvent* ev);
//函数实现
void MainWindow::closeEvent(QCloseEvent *ev)
{
    QMessageBox::Button btn = QMessageBox::question(this, "关闭窗口", "您确定要关闭窗口吗?");
    if(btn == QMessageBox::Yes)
    {
        // 接收并处理这个事件
        ev->accept();
    }
    else
    {
        // 忽略这个事件
        ev->ignore();
    }
}

void MainWindow::resizeEvent(QResizeEvent *ev)
{
    qDebug() << "oldSize: " << ev->oldSize()
             << "currentSize: " << ev->size();
}

```

## 自定义按钮

### 按钮头文件

```cpp
#ifndef MYBUTTON_H
#define MYBUTTON_H

#include <QWidget>

class MyButton : public QWidget
{
    Q_OBJECT
public:
    explicit MyButton(QWidget *parent = nullptr);

    void setImage(QString normal, QString hover, QString pressed);

protected:
    void mousePressEvent(QMouseEvent* ev);
    void mouseReleaseEvent(QMouseEvent* ev);
    void enterEvent(QEvent* ev);
    void leaveEvent(QEvent* ev);
    void paintEvent(QPaintEvent* ev);

signals:
    void clicked();

private:
    QPixmap m_normal;
    QPixmap m_press;
    QPixmap m_hover;
    QPixmap m_current;
};

#endif // MYBUTTON_H
```

### 按钮实现

```cpp
#include "mybutton.h"

#include <QPainter>

MyButton::MyButton(QWidget *parent) : QWidget(parent)
{

}

void MyButton::setImage(QString normal, QString hover, QString pressed)
{
    // 加载图片
    m_normal.load(normal);
    m_hover.load(hover);
    m_press.load(pressed);
    m_current = m_normal;
    // 设置按钮和图片大小一致
    setFixedSize(m_normal.size());
}

void MyButton::mousePressEvent(QMouseEvent *ev)
{
    // 鼠标被按下, 发射这个自定义信号
    emit clicked();
    m_current = m_press;
    update();
}

void MyButton::mouseReleaseEvent(QMouseEvent *ev)
{
    m_current = m_normal;
    update();
}

void MyButton::enterEvent(QEvent *ev)
{
    m_current = m_hover;
    update();
}

void MyButton::leaveEvent(QEvent *ev)
{
    m_current = m_normal;
    update();
}

void MyButton::paintEvent(QPaintEvent *ev)
{
    QPainter p(this);
    p.drawPixmap(rect(), m_current);
}
```

###  按钮使用

* 在`UI`自定义`QWidget`类型
* 鼠标`右键`
* 点击`提升为`，输入指定类名
* 将提升的`类名`称改为`指定按钮的类名`
* `点击添加`
* `点击提升`

```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 给自定义按钮设置图片
    ui->button->setImage(":/ghost-1.png", ":/ghost-2.png", ":/ghost-3.png");
    // 处理自定义按钮的鼠标点击事件
    connect(ui->button, &MyButton::clicked, this, [=]()
    {
        QMessageBox::information(this, "按钮", "莫要调戏我...");
    });
}

MainWindow::~MainWindow()
{
    delete ui;
}
```
## 时间过滤器

### 安装

```cpp
//filterObj 来过滤当前对象中的指定事件
void QObject::installEventFilter(QObject *filterObj);
//建立事件过滤器
[virtual] bool QObject::eventFilter(QObject *watched, QEvent *event);
```

## 使用案例

> 在`QTextEdit`中屏蔽掉回车键

### 头文件
```cpp
QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

    bool eventFilter(QObject *watched, QEvent *event);


private:
    Ui::MainWindow *ui;
};

```
### 函数实现
```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
	//安装过滤器，由this对应的主窗口进行事件过滤
        //this可以换成对应QWidget窗口
    ui->textEdit->installEventFilter(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

bool MainWindow::eventFilter(QObject *watched, QEvent *event)
{
    // 判断对象和事件
    if(watched == ui->textEdit && event->type() == QEvent::KeyPress)
    {
        QKeyEvent* keyEv = (QKeyEvent*)event;
        if(keyEv->key() == Qt::Key_Enter ||         // 小键盘确认
                keyEv->key() == Qt::Key_Return)     // 大键盘回车
        {
            qDebug() << "我是回车, 被按下了...";
            return true;
        }
    }
    return false;
}
```
