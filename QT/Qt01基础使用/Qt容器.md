# 容器
## Group Box
> 在窗口的左上可添加`文字说明`

* `title`:组框的`标题`
* `Alignment`:标题的对齐方式，
* `flat`：设置组框的绘制方式
* `checkable`：设置是否可以给组框添加`复选框`
* `checked`：设置添加的复选框的``选择状态`

## Scroll Area
> 若`子窗口`的区域`超过`了会生成`向下/左右的滚动条`
*  ``widgetResizable`：是否`自动调整`滚动区域内部窗口大小
*  `alignment`:对齐方式
```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 创建一个垂直布局对象
    QVBoxLayout* vlayout = new QVBoxLayout;

    for(int i=0; i<11; ++i)
    {
        // 创建标签对象
        QLabel* pic = new QLabel;
        // 拼接图片在资源文件中的路径
        QString name = QString(":/images/%1.png").arg(i+1);
        // 给标签对象设置显示的图片
        pic->setPixmap(QPixmap(name));
        // 设置图片在便签内部的对其方式
        pic->setAlignment(Qt::AlignHCenter);
        // 将标签添加到垂直布局中
        vlayout->addWidget(pic);
    }

    // 创建一个窗口对象
    QWidget* wg = new QWidget;
    // 将垂直布局设置给窗口对象
    wg->setLayout(vlayout);
    // 将带有垂直布局的窗口设置到滚动区域中
    ui->scrollArea->setWidget(wg);
}
```

## Tool Box
> `工具盒子`，分为`很多层`，每一层都`对应一个窗口`，通过`Page页面`切换不同层的窗口

* `currentIndex` :工具向中当前选项卡对应的`索引`，从``0`开始
* `currentItemText`：工具箱中当前选项卡上`显示的标题`
* `curentItemName`：当前选项卡对应的子窗口的``objectName`
* `currentItemIcon`：当前选项卡显示的`图标`
* `currentItemToolTi`p：当前选项卡显示的``提示信息`
* `tabSpacing`: 工具箱中窗口折叠后，选项卡之间的``间隙`
```cpp
// 信号
// 工具箱中当前显示的选项卡发生变化, 该信号被发射, index为当前显示的新的选项卡的对应的索引
[signal] void QToolBox::currentChanged(int index);

// 槽函数
// 通过工具箱中选项卡对应的索引设置当前要显示哪一个选项卡中的子窗口
[slot] void QToolBox::setCurrentIndex(int index);
// 通过工具箱中选项卡对应的子窗口对象设置当前要显示哪一个选项卡中的子窗口
[slot] void QToolBox::setCurrentWidget(QWidget *widget);
```

## Tab Widget
> 带标签页的窗口，通过标签``切换窗口`

* `tabPosition`：标签在窗口中的位置
* `tabShape`：标签的形状
* `currentIndedx`:当前选中的标签对应的索引
* `iconSize`:标签上显示的图标大小
* `elideMode`:标签上的文本信息省略方式
* `usersSctollButtons`：标签太多时，是否添加滚动按钮
* `documentMode`：文档模式是否开启，如果开启窗口边框``会被去掉`
* `tabsClosable`：标签页上是否添加关`闭按钮`
* `movable`:标签页是够可以移动
* `tabBarAuoHide`:当标签`<2`时，标签栏是否自动隐藏
* `currentTabText`:当前标签上显示的文本信息
* `currentTabName`：当前标签对应的窗口的objectName
* `currentTabIcon`：当前标签上显示的图标
* `currentTabToolTip`：当前标签的`提示信息`

```cpp
//信号
// 每当当前页索引改变时，就会发出这个信号。参数是新的当前页索引位置，如果没有新的索引位置，则为-1
[signal] void QTabWidget::currentChanged(int index);
// 当用户单击索引处的选项卡时，就会发出这个信号。index指所单击的选项卡，如果光标下没有选项卡，则为-1。
[signal] void QTabWidget::tabBarClicked(int index)
// 当用户双击索引上的一个选项卡时，就会发出这个信号。
// index是单击的选项卡的索引，如果光标下没有选项卡，则为-1。
[signal] void QTabWidget::tabBarDoubleClicked(int index);
// 此信号在单击选项卡上的close按钮时发出。索引是应该被删除的索引。 	
[signal] void QTabWidget::tabCloseRequested(int index);
//槽函数

// 设置当前窗口中显示选项卡index位置对应的标签页内容
[slot] void QTabWidget::setCurrentIndex(int index);
// 设置当前窗口中显示选项卡中子窗口widget中的内容
[slot] void QTabWidget::setCurrentWidget(QWidget *widget);
```

```cpp

// mainwindow.cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 点击了标签上的关闭按钮
    connect(ui->tabWidget, &QTabWidget::tabCloseRequested, this, [=](int index)
    {
        // 保存信息
        //将点击关闭的tabwidget以及标题保存到enqueue队列当中，方便添加时调用
        QWidget* wg = ui->tabWidget->widget(index);
        QString title = ui->tabWidget->tabText(index);
        m_widgets.enqueue(wg);
        m_names.enqueue(title);
        // 移除tab页
        ui->tabWidget->removeTab(index);
        //设置当前窗体可用
        ui->addBtn->setEnabled(true);

    });

    // 当标签被点击了之后的处理动作
    connect(ui->tabWidget, &QTabWidget::tabBarClicked, this, [=](int index)
    {
        qDebug() << "我被点击了一下, 我的标题是: " << ui->tabWidget->tabText(index);
    });

    // 切换标签之后的处理动作
    connect(ui->tabWidget, &QTabWidget::currentChanged, this, [=](int index)
    {
        qDebug() << "当前显示的tab页, 我的标题是: " << ui->tabWidget->tabText(index);
    });

    // 点击添加标签按钮点击之后的处理动作
    connect(ui->addBtn, &QPushButton::clicked, this, [=]()
    {
        // 将被删除的标签页添加到窗口中
        // 1. 知道窗口对象, 窗口的标题
        // 2. 知道添加函数
        ui->tabWidget->addTab(m_widgets.dequeue(), m_names.dequeue());//从队列中取出数据
        //如果队列为空
        if(m_widgets.empty())
        {
        	//按钮不可用
            ui->addBtn->setDisabled(true);
        }
    });
}
```
## Stacked Widget
> 一个栈窗口，也是分为不同页面，但是页面切换需要借助外界的控件进行间接切换栈窗口

## Widget
> Qt最简单的窗口，Frame内嵌到窗口 里是可以带边框的，而Widget是不带边框的

* enabled:窗口能否使用，关闭窗口就不能使用
* geometry:当前窗口相当于其父窗口所在的位置和宽度高度
* sizePolicy：当前窗口的尺寸策略（尺寸策略生效，必须父窗口进行布局)
	- Fixed ： 大小固定，不随父窗口进行变化
	- Minimun: 默认最小状态显示，父窗口变化，子窗口也会随着变化
	- Maximum：默认以最大状态显示，但也会随着父窗口变化而变化
	- Perfered: 同时拥有Minimum和Maximum，窗口可以增长和缩放，默认会一一个合适大小显示，默认。
	- Expanding: 具有延展属性，可以侵略相邻窗口的空间
	- MinimumExpanding : 默认可以延伸，也可以侵略其它窗体控件
* minimumSize：当前窗口的最小尺寸
* maximumSize:当前窗口的最大尺寸
* sizeIncrement:当窗口大小发生变化之后，子窗口相对于父窗口位移如何进行计算
* font：窗口字体
* cursor:当前窗口的光标显示
* mouseTracing：设置窗口是否追踪鼠标移动事件
* TableTracking : 设置窗口是否追踪鼠标移动事件
* focusPolicy：设置窗口焦点策略
* contextMenuPolicy：设置窗口右键菜单策略
* toolTip：鼠标在窗口一定时间，进行提示
* toolTipDuration：设置窗口提示信息持续时长，单位毫秒
* styleSheet : 给窗口设置样式表，进行窗口美化，帮助文档有默认文档


#### Frame
> Qt最`简单`的窗口，Frame``内嵌到窗口` 里是可以`带边框的`，而Widget是不带边框的

* `frameShape`：设置`表框的形状`
* `frameshadow` :窗口`阴影形式`
	- `QFrame::Plain`: `简单的,朴素的, 框架和内容与周围环境显得水平;`
	             使用调色板绘制QPalette::WindowText颜色(没有任何3D效果)
    - `QFrame::Raised`: 框架和内容出现凸起;使用当前颜色组的明暗颜色绘制3D凸起线
    - `QFrame::Sunken`: 框架及内容物凹陷;使用当前颜色组的明暗颜色绘制3D凹线
* `lineWidth`:线条的`宽度`，默认值为`1`
* `midLineWidth`：设置`中线宽度`，主要`控制阴影`，默认值为`0`
![设置图](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtm3eiby0Hvabx3icLibZMIK2KicibxbT4JUNYOFYkKibX4AgB3XjeqKzaXjC7Kw7S8icNj5BvibsxfiaePbLw/640?wx_fmt=jpeg  "Frame设置图")

#### 其它
* `MDI Area`：``多文档区域容器`
* `Dock Widget`:`停靠窗口容器`
* `QAxWidgeet`：`Active插件`
