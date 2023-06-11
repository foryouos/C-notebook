# 绘图事件
> 定义绘图事件,可以实现大部分`2D`绘图，启动会调用此`事件函数`
> `void paintEvent(QPaintEvent *);`
>
![绘图事件](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYONX8EpVl6o5spF6Hk3dDODARLNZ9ibnLsDLRH4AkIvias65GgmT8fpc2w/640?wx_fmt=png "绘图事件")


```cpp
//绘图事件，实现大部分绘图
void Widget::paintEvent(QPaintEvent *)
{
	//Constructs a painter that begins painting the paint device immediately.
    //定义一个QPainter类，实现2D绘图
    QPainter painter(this);
    //QPen为笔，设置相关笔的颜色和风格
    //Constructs a black pen with 1 width and the given style.
    QPen pen(QColor(255, 0, 0));
    pen.setWidth(3); //设置笔显示的宽度
    //设置显示风格
    pen.setStyle(Qt::DashDotLine);
    //将此pen风格应用到painter绘图上
    painter.setPen(pen);//将QPainter以pen为笔显示
    //painter绘画，
    //参数：起点，终点
    painter.drawLine(QPoint(0, 0), QPoint(100, 100)); //划线
    painter.drawEllipse(QPoint(100, 100), 20, 20);  //画圆
    painter.drawEllipse(QPoint(100, 100), 50, 80);
    //画正方形
    //Draws the current rectangle with the current pen and brush.
    painter.drawRect(100, 100, 30, 30);
    //画长方形，起点，长度，宽度
    painter.drawRect(100, 100, 30, 80);
    //绘画文字信息，位置和现实的字符串
    painter.drawText(100, 100, "I love you, Rick!");
    //设置QBrush刷子，将QPainter的显示区域，显示成QBrush刷子显示的样式
    //设置成背景绿色，
    QBrush brush(Qt::green, Qt::Dense3Pattern);
    painter.setBrush(brush); //设置当前刷子
    //绘制长方形,起始位置200,200，长度50，宽度80
    painter.drawRect(200, 200, 50, 80);
    //锥形，渐变
    QConicalGradient conicalGradient(QPointF(200, 100), 0);
    //setColorAt创建一个停止的点，必须给位置，位置范围为0~1，
    //逆时针
    conicalGradient.setColorAt(0.2, Qt::cyan); //设置渐变颜色
    conicalGradient.setColorAt(0.4, Qt::black);
    conicalGradient.setColorAt(0.7, Qt::red);
    conicalGradient.setColorAt(1, Qt::yellow);
    //painter设置Brush
    painter.setBrush(conicalGradient);
    //绘画成员
    painter.drawEllipse(QPointF(200, 100), 50, 50);
}
```
### `Qt::DashDotLine`

![Qt::DashDotLine](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOW7cybDewxnxgiaC8jUau8DibicIhNWP1uqp3GiboDu88icVSR5uv6RaCd2w/640?wx_fmt=png "Qt::DashDotLine")

## 坐标平移

![坐标平移](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOHWYloN4Sl2fIFd7JQKHicY0ibOssC2ZIUMcibV9Bic46urWmt9Uwaq7HGA/640?wx_fmt=png "坐标平移")

```cpp
//1.Antialliasing, translate
painter.setPen(QPen(Qt::red, 15));
//QPoint(200, 150) 圆心位置，宽度（横轴半径），高度
painter.drawEllipse(QPoint(200, 150), 100, 100);
painter.setRenderHints(QPainter::Antialiasing);
painter.translate(200, 0);// 坐标平移 横坐标，纵坐标
painter.drawEllipse(QPoint(200, 150), 100, 100);
```

## 旋转

![旋转](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOZSYpiaKJroFfIDAQnu9uDeicS6wAjSgCApLCicenCm0J1uu6IrQZEXVvg/640?wx_fmt=png "旋转")



```cpp
//2.rotate

painter.setPen(QPen(Qt::red, 10));
painter.drawLine(QPoint(10, 10), QPoint(100, 100));
painter.save(); //保存当前坐标系
painter.translate(200, 0);
painter.rotate(90);
painter.setPen(QPen(Qt::blue, 10)); //设置颜色和宽度
painter.drawLine(QPoint(10, 10), QPoint(100, 100)); //画显
painter.restore(); //回到之前保存的坐标系
painter.drawText(10,10,"I Love you");
```





## 缩放



![缩放](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOu0FZJqma5SUskcUlYe5wwudJL8NbTOoKZPR9V7EEelXfxPM95Yo30A/640?wx_fmt=png "缩放")

```cpp
//3.scale
painter.setPen(QPen(Qt::red));
painter.setBrush(Qt::yellow); //使用 刷子染色

painter.drawRect(50, 50, 50, 50);
painter.save(); //保存当前坐标信息
painter.scale(0.5, 0.5);
painter.setBrush(Qt::green);
painter.drawRect(50, 50, 50, 50);
painter.restore();
```



## 扭曲

![扭曲](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOfWGSTj8fgnUTGIVdb4CXzu5eleRvOo3vEZiaAzeF56v8hOBsjUrSviaA/640?wx_fmt=png "扭曲")

```cpp
 //4.shear扭曲x轴和y轴
painter.setPen(QPen(Qt::blue));
painter.setBrush(Qt::yellow);
painter.drawRect(50, 50, 50, 50);
painter.save();
//0~1 1为一条直线扭曲幅度
painter.shear(0.9, 0.9);
painter.setBrush(Qt::gray);
painter.drawRect(50, 50, 50, 50);
painter.restore();
```



## 调用时间处理函数

![调用时间处理函数](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOdLScoclG59slLMqfSkQuDQn4cuSibXmoNw5EkibwwqK4xm7R4sXB4XIA/640?wx_fmt=png "调用时间处理函数")

```cpp
//头文件
int angle;
//实现函数
angle = 0;
QTimer *timer = new QTimer(this);
timer->start(1000); //每一秒刷新一次
connect(timer, &QTimer::timeout, [&]()
        {
            update();//repaint也可以，但是update的性能更好Qt优化速度，调用此函数，会自动调用paintEvent
        });
void Widget::paintEvent(QPaintEvent *)
{
    angle += 6; //每一秒旋转6度
    if(angle == 360)
        angle = 0;
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing); //设置抗锯齿，使表面光滑
    painter.translate(width()/2, height()/2); //使平移到中心位置，
    painter.drawEllipse(QPoint(0, 0), 120, 120); //以上面的中心，画圆
    painter.rotate(angle); //旋转角度
    painter.drawLine(QPoint(0, 0), QPoint(0, -100)); //在旋转的角度，画线
}
```



## 绘制文字

![绘制文字](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOxtoVy3YZQPFF5ialmR7USfdGUibFK9aM0FgtXKrI3ga5rRaJnUibiab7fQ/640?wx_fmt=png "绘制文字")

```cpp
void Widget::paintEvent(QPaintEvent *)
{
    QPainter painter(this);

    //QFont font("华文琥珀", 20, QFont::Bold, true);
    QFont font("宋体", 50, QFont::Bold, true);  //设置文字信息
    font.setUnderline(true); //设置下划线

    painter.setFont(font); //将上面的字体信息应用到此painter上
    painter.setPen(Qt::blue); //设置笔为blue
    painter.drawText(10, 100, "I love you！"); //设置初始位置以及显示的文字信息
}
```



## 绘制路径

![绘制路径](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOS1ia9J7I8ozGaibN44AHF9ibbHibzl4Wjr1tBS7AkQXtpuNPdeewic9PbpQ/640?wx_fmt=png "绘制路径")

```cpp
void Widget::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QPainterPath path; //设置路径

    path.moveTo(50, 250);  //路径移动到
    path.lineTo(50,200);
    path.lineTo(100, 100);
    path.addEllipse(QPoint(100, 100), 30, 30); //添加圆
    painter.setPen(Qt::red); //设置画笔颜色
    painter.drawPath(path);  //将此路径应用到此painter

    path.translate(200, 0); //坐标平移
    painter.setPen(Qt::blue); //设置画笔颜色
    //会将path路径的画法映射出来，自定义的特性将取代path中相同的
    painter.drawPath(path); //将路径应用到此painter
}
```



## 图片

* `QPixmap`：图片类，用于显示图片，对图片显示做`优化处理`，与平台有关
* `QImage`: 依赖于`平台`，可以做`像素级`需修改(对具体的像素点)
* `QBitmap` ：仅能`黑白图片`显示
* `QPicture`: 绘图容器里面保存有绘图记录和重置绘图的指令，存储的是`二进制性的文件`(无法再win端双击打开picture文件)，但可通过`QPicture.load`加载出来

![修改像素点](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOx6Lk8gIUwEia8eLhfHr4ZxLJG7IRQXGOZ9t8UPXbnhbqZ5dzagrzsBg/640?wx_fmt=png "修改像素点")

```cpp
void Widget::paintEvent(QPaintEvent *event)
{
    QPainter painter(this);
    QImage img; //依赖于平台，可以做像素级需修改(对具体的像素点)
    img.load(":/Image/Image/optimus prime.jpeg");

    for(int i = 50; i < 100; i++) //修改 相对应的像素点
    {
        for(int j = 50; j < 100; j++)
        {
            QRgb rgb = qRgb(255, 0, 0);
            img.setPixel(i, j, rgb);
        }
    }

    painter.drawImage(0, 0, img);
}
```

```cpp
  /*   QPixmap
QPixmap pix(400, 300);
pix.fill(Qt::white);
QPainter painter(&pix); //使用QPixmap呈现，并不会呈现到主窗口上
painter.setPen(Qt::red);
painter.drawEllipse(QPoint(100, 100), 50 ,50);
pix.save("E://mypix.jpg"); //保存到本地
*/
//QImage
QImage img(400, 300, QImage::Format_RGB32);
img.fill(Qt::white);
QPainter painter(&img);  //使用QImage呈现
painter.setPen(Qt::red);
painter.drawEllipse(QPoint(100, 100), 50 ,50);
img.save("E://myImage.jpg");
//QPicture  可以理解为是一个绘图的容器，里面保存有绘图的纪录和重绘制的指令
//存储的形式是二进制形式，也就是说我们无法双击打开picture文件
QPicture pic;
//QPainter painter(&pic);
QPainter painter;
painter.begin(&pic);
painter.setPen(Qt::red);
painter.drawEllipse(QPoint(100, 100), 50, 50);
painter.end();
pic.save("E://abc.xyz");
*/
//存储的为黑白图片
QBitmap bm(400, 300);
QPainter painter(&bm);
painter.setPen(Qt::red);
painter.drawEllipse(QPoint(100, 100), 50, 50);
bm.save("E://bm.jpg");

//函数加载存储的二进图片
void Widget::paintEvent(QPaintEvent *)
{
    /*
    QPainter painter(this);
    QPicture pic;
    pic.load("E://abc.xyz");
    painter.drawPicture(0, 0, pic);
    */
    QPainter painter(this);
    QBitmap bm;
    bm.load("E://optimus prime.jpeg");
    painter.drawPixmap(0, 0, bm);
}
```



## 绘制弧线



![绘制弧线](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbs2HsxeVHavKGibCl24ELjYOo6XibRLAJJkvdHu2NQ9ZuJvKibtxchtk7TACbMlicibo0dn1pwZ03mW14g/640?wx_fmt=png "绘制弧线")

```cpp
	//构造函数设置
    resize(1600, 900);
    setStyleSheet("background-color: balck"); //设置背景颜色

void Widget::paintEvent(QPaintEvent *event)
{
    QPainter painter(this);
    QPen pen;

    pen.setColor(Qt::green);
    painter.setPen(pen);
    //startAngle和spanAngle跨角必须是1/16th作为单位，起始角度和结束角度，三点钟方向为起始，逆时针为正类似坐标系
    //int x, int y, int width, int height, int startAngle, int spanAngle
    //x,y弧对应的正方形装的左上角，width和height为弧对应的完整的形状对应的长度和宽度
    painter.drawArc(25, 25, 1550, 1550, 0 * 16, 180 * 16);
    painter.drawArc(180, 180, 1240, 1240, 0 * 16, 180 * 16);
    painter.drawArc(335, 335, 930, 930, 0 * 16, 180 * 16);
    painter.drawArc(490, 490, 620, 620, 0 * 16, 180 * 16);
    painter.drawArc(645, 645, 310, 310, 0 * 16, 180 * 16);
}
```

