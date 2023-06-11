# 补充
## 使用函数指针调用函数重载
```cpp
void print(int a,int b);
void print(int a);

//使用函数指针 ()代表参数个数 pt为调用函数后的函数名称
void(*pt)(int) = print;
再使用 pt进行使用，就是调用的print
pt(3);
```
## 代码实现窗口
* 菜单栏
* 工具栏
* 状态栏(最下面)
* 浮动窗口
* 中心
```cpp
#include "mainwindow.h"

#include <QMenuBar>
#include <QToolBar>
#include <QStatusBar>
#include <QLabel>
#include <QDockWidget>
#include <QTextEdit>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    resize(400, 300);

    //MenuBa创建菜单栏
    QMenuBar *menubar = new QMenuBar(this);
    setMenuBar(menubar); //设置菜单栏
	//"文件(&F)"  &F代表快捷键
    QMenu *filename = menubar->addMenu("文件(&F)"); 菜单栏上添加菜单
    QMenu *editmenu = menubar->addMenu("编辑(&E)");
    QMenu *buildname = menubar->addMenu("构建(&B)");

    filename->addAction("新建文件(&N)");
    filename->addAction("打开文件(&O)");
    filename->addSeparator();  //添加分隔符
    filename->addAction("关闭文件(&C)");

    editmenu->addAction("恢复(&U)"); //在编辑下再创建菜单
    buildname->addAction("构建所有项目(&R)");

    //ToolBar工具栏
    QToolBar *toolbar = new QToolBar(this);
    addToolBar(Qt::TopToolBarArea, toolbar); //设置toolbar在窗口的顶部可上下左右四个位置可选

    toolbar->addAction("新建");
    toolbar->addAction("打开");
    toolbar->addSeparator();
    toolbar->addAction("关闭");

    //StatusBar 状态栏，只能在下边
    QStatusBar *stbar = new QStatusBar(this);
    setStatusBar(stbar);
    QLabel *label = new QLabel(this);
    label->setText("状态栏");
    stbar->addWidget(label); //状态栏添加此label

    //DockWidget浮动窗口
    QDockWidget *dockwidget = new QDockWidget("小窗口", this);
    addDockWidget(Qt::LeftDockWidgetArea, dockwidget); //添加在窗口左面

    //Central Widget中间核心区，
    QTextEdit *edit = new QTextEdit(this);
    setCentralWidget(edit);
}
```
## 窗口添加ICON
```cpp
//ui->actionNew->setIcon(QIcon("C:/Qt/images/new.png"));
//添加Qt资源文件使用格式   ": + 前缀 + 文件名"
ui->actionNew->setIcon(QIcon(":/Image/images/new.png"));
//ui->actionOpen->setIcon(QIcon("C:/Qt/images/open.png"));
ui->actionOpen->setIcon(QIcon(":/Image/images/open.png"));
```
## 模态与释放空间
> 模态(必须关闭显示的窗口，才可操作其它窗口)
> 非模态(显示窗口后，仍然可以操作其它窗口)
> 关闭窗口，释放本窗口内存空间：`dialog->setAttribute(Qt::WA_DeleteOnClose);`
> `QCoreApplication::processEvents(); `内核区去检查有没有其他事情要做

```cpp
QDialog *dialog = new QDialog(this);
dialog->resize(200, 100);

//dialog->show();       //非模态对话框

//dialog->exec();           //模态对话框 方式1

dialog->setModal(true);  //模态对话框 方式2
dialog->show();
//关闭此窗口对应空间会被释放掉
dialog->setAttribute(Qt::WA_DeleteOnClose);
```
## 错误对话框
```cpp
private:
	QErrorMessage *errordlg;

//函数实现
//点击之后显示错误对话框
void MainWindow::on_pushButton_7_clicked()
{
//    QErrorMessage *errordlg = new QErrorMessage(this);
    errordlg->setWindowTitle("错误");
    errordlg->showMessage("危险");
}

```
## 向导对话框
```cpp
QWizardPage *createPage1(void)
{
    QWizardPage *page = new QWizardPage;
    page->setTitle("第一步操作");
    return page;
}

QWizardPage *createPage2(void)
{
    QWizardPage *page = new QWizardPage;
    page->setTitle("第二步操作");
    return page;
}

void MainWindow::on_pushButton_8_clicked()
{
    QWizard wizard(this);
    wizard.setWindowTitle("向导对话框");
    wizard.addPage(createPage1()); //创建两个向导页面
    wizard.addPage(createPage2()); 
    wizard.exec();
}
```
# 控件
## Item 
> `Item view`在处理`大量数据`时(数据库操作时)`性能`要`大`于`Item widgets`
### list Widget
> 列表

![List UI](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsMW39IOCFlyaIVRyBjA9Z0Uh4xdyqVAu04SxUejib5vtZgWGJqib8s5kWokgIZruYa8ib82MskX43yA/640?wx_fmt=png "List ui")
```cpp
 //list列表
//添加item
ui->listWidget->addItem("Hello world");
ui->listWidget->addItem("Good morning");

QListWidgetItem *listitem = new QListWidgetItem("I love you, Rick!");
ui->listWidget->addItem(listitem);
listitem->setTextAlignment(Qt::AlignHCenter); //中间居中

QStringList list2;
//list<<str; //通过操作运算符<<将一个QString字符串存储到该列表中
list2 << "ABCD" << "EFGH" << "IJKL" << "LMNO"; //通过重载将数据加入到list2中
ui->listWidget_2->addItems(list2); //通过QStringList形式添加
```
### Tree Widget
> 树形结构

![树形结构](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsMW39IOCFlyaIVRyBjA9Z0xKdEMnCs0faj9RPn15qR81MMoickrWA2GbKmyzmic8zPSI8GN55jZ2iag/640?wx_fmt=png "树形结构")


```cpp
//QStringList list;
//list << "Name" << "Address";
//ui->treeWidget->setHeaderLabels(list);
//设置树的头标题
ui->treeWidget->setHeaderLabels(QStringList() << "Name" << "Address");
//为树添加顶部的栏目
QTreeWidgetItem *treeitem1 = new QTreeWidgetItem(QStringList("Bookmarks Toolbar"));
ui->treeWidget->addTopLevelItem(treeitem1);
QTreeWidgetItem *treeitem2 = new QTreeWidgetItem(QStringList("Bookmark Menu"));
ui->treeWidget->addTopLevelItem(treeitem2);
//为顶部栏目设置图标
treeitem1->setIcon(0, QIcon(":/Images/Image/folder.png"));
treeitem2->setIcon(0, QIcon(":/Images/Image/folder.png"));
//为顶部tree设置孩子
//第一个数据为Name，第二个为Address
QTreeWidgetItem *childitem1 = new QTreeWidgetItem(QStringList() << "QWidget" << "Page10");
treeitem1->addChild(childitem1); //将此添加到列表当中
QTreeWidgetItem *childitem2 = new QTreeWidgetItem(QStringList() << "QMainWindow" << "Page20");
treeitem1->addChild(childitem2);

QTreeWidgetItem *childitem3 = new QTreeWidgetItem(QStringList() << "QPushButton" << "Page30");
treeitem2->addChild(childitem3);
QTreeWidgetItem *childitem4 = new QTreeWidgetItem(QStringList() << "QLabel" << "Page40");
treeitem2->addChild(childitem4);
```
### Table

![Table ](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsMW39IOCFlyaIVRyBjA9Z0FA1Uz6T96lHWibM5OSNnJWFpAV5fNv9jszDDZPRfqs7ExmTBWNQbGRQ/640?wx_fmt=png "Table")

```cpp
ui->tableWidget->setColumnCount(3); //添加列
ui->tableWidget->setRowCount(3); //添加行
//增加顶部标题
ui->tableWidget->setHorizontalHeaderLabels(QStringList() << "name" << "sex" << "age");

//ui->tableWidget->setItem(0, 0, new QTableWidgetItem("Rick"));
//设置信息列表
QStringList NameList;
NameList << "Rick" << "Jack" << "Michael";
QStringList SexList;
SexList << "Male" << "Male" << "Male";

//将数据加入到Table中
for(int row = 0; row < 3; row++)
{
    int col = 0;
    ui->tableWidget->setItem(row, col++, new QTableWidgetItem(NameList[row]));
    ui->tableWidget->setItem(row, col++, new QTableWidgetItem(SexList[row]));
    ui->tableWidget->setItem(row, col, new QTableWidgetItem(QString::number(20)));  //int --> string
}
```
## Input Widgets
*  `comboBox` 下拉列表

![combox](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsMW39IOCFlyaIVRyBjA9Z07ZWLnLxciajibwSH0hTpyqPrxpZWzXMsTnkiaj6BdRgjbeg4KlQpicIKTQ/640?wx_fmt=png "combox")

```cpp
ui->comboBox->addItem("博士");
ui->comboBox->addItem("硕士");
ui->comboBox->addItem("本科");
ui->comboBox->addItem("专科");
ui->comboBox->addItem("高中及以下");
```
*  `Font combox` 字体下拉栏
* ` Line Edit` 一行
* `text Edit` 可改变格式
* `Plain Text Edit` ：只能写
* `SpinBox`数字增加减少
* `Date Edit` 日期
* `Time Edit` 事件
* `Date/Time Edit` 日期时间
```cpp
//获取当前时间
QDateTime curDateTime = QDateTime::currentDateTime();
ui->timeEdit->setTime(curDateTime.time());
ui->dateTimeEdit->setDateTime(curDateTime);
```
## Label

```cpp
//设置文字
ui->label->setText("I love you Rick!");
//设置图片，QPixmap
ui->label_2->setPixmap(QPixmap(":/Image/Image/1.jpeg"));
//使使用尺寸，覆盖全屏
ui->label_2->setScaledContents(true);
//显示未gif
QMovie *mv = new QMovie(":/Image/Image/2.gif");
// 显示MV
ui->label_3->setMovie(mv);
//适应磁村
ui->label_3->setScaledContents(true);
//呈现图片
mv->start();
```