- 抽象：是对同一类对象的共同属性和行为进行概括，形成类。

- 数据抽象：描述某类对象的属性或状态（对象相互区别的物理量）

- 代码抽象：描述某类对象的共有的行为特征或具有的功能

- 抽象的实现：类

- 封装：将抽象出的数据，代码封装在一起，形成类。

- 目的:增强安全性和简化编程，使用者不必了解具体的实现细节，而只需要通过外部接口，以特定的访问权限，来使用类的成员

- 实现封装：类声明中的{}

  

类与对象的关系

- 对象是现实中的对象在程序中的模拟

- 类是同一类对象的抽象，对象是类的某一特定实体

- 定义类的对象，才可以通过对象使用类中定义的功能。

  

  

  

- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class 类名称{  public:  //任何外部函数都可以访问共有类型数据和函数        公有成员（外部接口）  private: //只允许本类中额函数访问，类外部函数不能访问        私有成员  protected:     //为有继承关系设计        保护型成员}
```

对象定义的语法：

- 
- 
- 
- 
- 

```
Clock myClock; //类中成员相互访问:直接使用成员名访问//类外对象名.成员名//访问public属性的成员
```



类内初始化

- 可以为数据成员提供一个类内初始值
- 在创建对象时，类内初始值用于初始化数据成员
- 没有初始值的成员将被默认初始化



类的成员函数

- 在类中说明函数原型
- 可以在类外给出函数体实现，并在函数名前使用类名加以限定
- 也可以直接在类中给出函数体，形成内联成员函数
- 允许声明重载函数和带默认参数值的函数



内联成员函数

- 为了提高运行时的效率，对于较简单的函数可以声明为内联形式

- 内联函数体中不要有复杂结构（如循环语句和switch语句）

- 在类中声明内联成员函数的方式

- - 将函数体放在类的生命中
  - 使用inline关键字

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
/** project:钟表类* environment :VS2022* time:2023年1月31日*/#include <iostream>using namespace std;class Clock  //定义类边界{  public:  //特定访问权限    //外部接口，并赋默认参数值    void setTime(int newH = 0, int newM = 0, int newS = 0);      void showTime();  private: //公有函数共享私有成员    int hour, minute, second;};//成员函数的实现，类中的函数void Clock::setTime(int newH, int newM, int newS) {  hour = newH;  minute = newM;  second = newS;}void Clock::showTime(){  cout << "Time:" << hour << ":" << minute << ":" << second << endl;};
int main(void){  Clock C;  //定义对象  cout << "First time set and output:" << endl;  C.setTime(); //类外访问，使用默认时间  C.showTime(); //x显示时间  cout << "Second time set and output:" << endl;  C.setTime(12, 48, 30);  //类外访问  C.showTime();
  return 0;}
```



构造函数：

构造函数：在对象被创建时使用特定的值构造对象，将对象初始化为一个特定的初始状态。

形式：

- 函数名与类名相同
- 不能定义返回值类型，也不能有return语句
- 可以有形式参数，也可以没有形式参数
- 可以是内联函数
- 可以重载
- 可以带默认参数值



构造函数的调用时机：在对象创建时被自动调用。

隐含生成的构造函数

- 如果程序中未定义构造函数，编译器将在需要时自动生成一个默认构造函数

- - 参数列表为空，不为数据成员设置初始值
  - 如果类内定义了成员的初始值，则使用类内定义的初始值
  - 如果没有定义类内的初始值，则以默认方式初始化
  - 基本类型的数据默认初始化的值是不确定的

 

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class Clock  //定义类边界{  public:  //特定访问权限    Clock(int newH, int newM , int newS );  //构造函数    Clock(); //默认构造函数    //外部接口，并赋默认参数值    void setTime(int newH, int newM, int newS);      void showTime();  private: //公有函数共享私有成员    int hour, minute, second;};Clock::Clock(int newH, int newM, int newS):hour(newH),minute(newM),second(newS){}//若需要初始化判断，不能使用此方法 Clock::Clock():hour(0), minute(0), second(0)//描述初始化要求，不用再写，初始化直接装进去{}
```



“=default”

如果类中已定义构造函数，默认情况下编译器就不再隐含生成默认构造函数。如果此时依然希望编译器隐含生成默认构造函数，可以使用“=default”

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class Clock  //定义类边界{  public:  //特定访问权限    Clock() = default; //生成默认构造函数    Clock(int newH, int newM , int newS );  //构造函数    //Clock(); //默认构造函数    //外部接口，并赋默认参数值    void setTime(int newH, int newM, int newS);      void showTime();  private: //公有函数共享私有成员    int hour, minute, second;};
```



委托构造函数

类中往往有多个构造函数，只是参数表和初始化列表不同，其初始化算法都是相同的，这时，为了避免代码重复，可以使用委托构造函数.

委托构造函数使用类的其他构造函数执行初始化过程

- 
- 
- 
- 

```
Clock::Clock(int newH, int newM, int newS):hour(newH),minute(newM),second(newS){}Clock::Clock():Clock(0,0,0){}
```



复制构造函数

- 我们经常会需要用一个已经存在的对象，去初始化新的对象--复制构造函数
- 隐含生成的复制构造函数可以实现对应数据成员一一复制
- 自定义的复制构造函数可以实现特殊的复制功能

复制构造函数是一种特殊的构造函数，其形参为本类的对象引用。作用是用一个已存在的对象去初始化同类型的新对象。

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class 类名{  public:    类名(形参); //构造函数    类名(const 类名 & 对象名);   //复制构造函数,const常引用    //如果不希望对象被复制构造    类名(const 类名 & 对象名) = delete; //指示编译器不生成默认复制构造函数    //... } 类名::类 (const 类名 & 对象名)   //复制构造函数的实现 {   函数体 }
```



复制构造函数被调用的三种情况

- 定义一个对象时，以本类另一个对象作为初始值，发生复制构造，
- 如果函数的形参是类的对象，调用函数时，将使用实参对象初始化形参对象，发生复制构造
- 如果函数的返回值是类的对象，函数执行完成返回主调函数时，将使用return语句中的对象初始化一个临时无名对象，传递给主调函数，此时发生复制构造。（一般此项编译器可以选择取消）。



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
/** project:复制构造函数* environment :VS2022* time:2023年1月31日*/#include <iostream>using namespace std;class Point{  public:      Point(int xx = 0, int yy = 0)    {      x = xx;      y = yy;    };  //构造函数    Point(const Point& p);//复制构造函数    void setX(int xx){      x = xx;    }    void setY(int yy){      y = yy;    }    int getX() const { return x; } //常函数    int getY() const { return y; }  private:    int x,y;};//复制构造函数的实现Point::Point(const Point& p){  x = p.x;  y = p.y;  cout << "Calling the copy constructor" << endl;}void fun1(Point p){  cout << p.getX() << endl;}Point fun2(){  Point a(1, 2);  return a;}int main(void){  Point a(4,5);  Point b(a);  //用a初始化b  cout << b.getX() << endl;  fun1(b);  //对象b作为fun1的实参  b = fun2(); //函数的返回值是类对象  cout << b.getX() << endl;  return 0;}
```



左值右值：左值位于赋值运算左侧的对象或变量，右值是位于赋值运算右侧的值。

- 对持久存在变量的引用称为左值引用，用&表示
- 对短暂存在可被移动的右值的引用称之为右值引用，用&&表示

- 
- 
- 
- 

```
float n = 6;float &lr_n = n; #左值引用float &&rr_n = n;  #错误，右值引用不能绑定到左值float &&rr_n = n*n; #右值表达式绑定到右值引用
```



通过标准库<utility>中的move函数可将左值对象强行移动为右值

- 
- 
- 

```
float n = 10;float &&rr_n = std::move(n); //将n转化为右值//把n的值转给rr_n了，从此n就不能用了
```



移动构造函数

基于右值引用，移动构造函数通过移动数据方式构造新对象，与复制构造函数类似，移动构造函数参数为该类对象的右值引用

- 
- 
- 
- 
- 
- 
- 
- 

```
#include <utility>class astring{  public:    std::string s;    astring (astring&& o) noexcept:s(std::move(o.s)) //显式移动所有成员    {函数体} }
```



- 移动构造函数不分配新内存，理论上不会报错，为配合异常捕获机制，需声明noexcept表明不会抛出异常

- 被移动的对象不应再使用，需要销毁或重新赋值

  

**析构函数**

- 完成对象被删除前的一些清理工作

- 在对象的生存期结束的时刻系统自动调用它，然后再释放此对象所属的空间

- 如果程序中未声明析构函数，编译器将自动产生一个默认的析构函数，其函数体为空。

  

- 

```
函数名前加 ~
```



类的组合

- 类中的成员是另一个类的对象
- 可以在已有抽象的基础上实现更复杂的抽象

原则：不仅要负责对本类中的基本类型成员数据初始化，也要对对象成员初始化

- 
- 
- 
- 
- 

```
类名::类名(对象成员所需的形参，本类成员形参)  :对象1(参数),对象2(参数),.....{  函数题其他语句}
```



构造组合类对象时的初始化次序

- 首先对构造函数初始化列表中列出的成员（包括基本类型成员和对象成员）进行初始化，初始化次序是成员在类体中定义的次序

- - 成员对象构造函数调用顺序：按对象成员的生命顺序，先声明先构造
  - 初始化列表中未出现的成员对象，调用默认构造函数（即无形参）初始化

- 处理完初始化列表之后，再执行构造函数的函数体

  



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
/** project:类的组合，线段（Line）类* environment :VS2022* time:2023年1月31日*/#include <iostream>#include <cmath>using namespace std;class Point{public:  Point(int xx = 0, int yy = 0)  {    x = xx;    y = yy;  };  //构造函数  Point(const Point& p);//复制构造函数  int getX() const { return x; } //常函数  int getY() const { return y; }private:  int x, y;};//类的组合class Line{public: //外部接口  Line(Point xp1, Point xp2);  Line(Line& l);  double getLen()  {    return len;  }private:  //私有数据成员  Point p1, p2; //Point类的对象P1，P2  double len;};Point::Point(const Point& p){  x = p.x;  y = p.y;  cout << "Calling the copy constructor" << endl;}//组合类的构造函数Line::Line(Point xp1, Point xp2) :p1(xp1), p2(xp2){  cout << "Calling constructor of Line" << endl;  double x = static_cast<double>(p1.getX() - p2.getX());  double y = static_cast<double>(p1.getY() - p2.getY());  len = sqrt(x * x + y * y);  //如果长度断点可以变化}
Line::Line(Line& l) :p1(l.p1), p2(l.p2) //组合类的复制构造函数{  cout << "Calling the copy constructor of Line" << endl;  len = l.len;}
int main(void){  Point myp1(1, 1), myp2(4, 5);  //建立Point 类的对象  Line line(myp1, myp2);    //建立Line类的对象  Line line2(line);     //利用复制构造函数建立一个新对象  cout << "The length of line is";  cout << line.getLen() << endl;  cout << "The length of the line2 is:" << line2.getLen() << endl;
  return 0;}
```



前向引用声明

- 类应该先声明，后使用
- 如果需要在某个类的生命之前，引用该类，则应进行前向引用声明
- 前向引用声明只为程序引入一个标识符，但具体声明在其他地方
- 在提供一个完整的类声明之前，不能声明该类的对象，也不能在内联成员函数中使用该类的对象
- 当使用前向引用声明时，只能使用被声明的符号，而不能涉及类的任何细节。

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class B;  //前向引用声明,不引入细节，class A{  public:    void f(B b);};class B{public:    void g(A a);};
```



参考资料：

- 郑莉教授《C++语言程序设计》