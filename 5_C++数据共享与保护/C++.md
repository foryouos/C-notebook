### 数据的共享与保护

```c++
#include <iostream>
//C++标准程序库的所有标识符都被声明在std命名空间内
using namespace std;

int main(void)
{
	cout << "hello std!" << endl;
	//std::cout << "Hello std!" << std::endl;
	return 0;
}
```
#### 定义
不同位置定义的变量和对象，其作用域，可见性，生存期都不同。程序模需要协作共同完成整个系统的功能，模块间需要共享数据，就需要直到应该将变量和对象定义在什么位置
#### 函数原形作用域
函数原型作用域中的参数，其作用域始于"\(",结束于"\)"
```C++
double area(double radius);
```
#### 局部作用域
函数的形参，在块中声明的标识符，其作用域自声明处起，限于块中
![局部作用域](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvicMgsRBhgayuYtLPvnzX9lFgmkOGPQnS55qoejMgRic6ydZy4s9TT0cniaLI0DJiadbIVC1lwOqX3dw/0?wx_fmt=jpeg "局部作用域")
#### 类作用域
其范围包括类体和非内联成员函数的函数体
#### 命名空间作用域
命名空间可以解决类名，函数名等的命名冲突
```C++
namespace 命名空间名
	{
		各种声明(函数声明，类声明.....)
	}
```
例:
```C++
namespace SomeNs
{
	class SomeClass
	{
		...
	}
}
//引用类名:
SomeNs::Someclass objl;
```
`using`语句的两种形式
* using 命名空间名::标识符名;
* `using namespace` 命名空间名;
#### 特殊的命名空间
* 全局命名空间：默认全局
* 匿名命名空间\(给空间没有名字):对每个源文件是惟一的(限于当前文件)
#### 限定作用域的枚举类
```C++
enum color{red,yellow,green};//不限定作用域
enum color2{red,yellow,green};//错误，重复定义
enum class color2{red,yellow,green};//正确，限定作用域
color c=red;//全局作用域color枚举类
color2 c2 = red; //错误，color2元素不在有效作用域内
color2 c2 = color2::red; //正确，使用color2作用域枚举元素
```
#### 可见性
* 可见性表示从内层作用域向外层作用域"看"时能看见什么
* 如果标识在某处可见，就可以在该处引用此标识符
* 如果某个标识符在外层中声明，且在内层中没有同一标识符的声明，则该标识符在内层可见
* 对于两个嵌套的作用域，如果在内层作用域内与外层作用域中同名的标识符，则外层作用域的标识符在内层不可见

#### 对象的生存期
对象从产生到结束这段时间，在对象生存周期，最想将保持它的值 ，直到被更新为止
* 静态生存期:生存期与程序的运行期相同，在文件作用域中声明的对象具有这种生存期，在函数内部声明静态生存期对象，要冠以关键字static
* 动态生存期:没有用`static`声明，开始于程序执行到声明点时，结束于命名该标识符的作用域结束处。
```C++
#include <iostream>
//C++标准程序库的所有标识符都被声明在std命名空间内
using namespace std;
int i =1; //i为全局变量
void other()
{
	//a,b为静态全局变量，具有全局寿命，局部可见，只第一次进入函数时被初始化
	static int a = 2;
	static int b;
	//c为局部变量
	int c = 10;
	a += 2;  //4
	i += 32; //33
	c += 5;  //15
	cout << "---- - other----" << endl;
	cout << " i：" << i << " a:" << a << " b:" << b << " c" << c << endl;
}
int main(void)
{

	//a为静态局部变量，具有全局 寿命，局部可见
	static int a;
	//b,c为局部变量，具有动态生存期
	int b = -10;
	int c = 0;
	cout << "——----main----" << endl;
	//i=1,a默认为0，b=-10,c=0
	cout << " i：" << i << " a:" << a << " b:" << b << " c" << c << endl;
	c += 8;  //c=8

	other();

	cout << "——----main----" << endl;
	cout << " i：" << i << " a:" << a << " b:" << b << " c" << c << endl;
	i += 10;
	other();

	return 0;
}
```
#### 对象间的共享
* 同类对象数据共享:静态数据成员
* 同类对象功能共享:静态函数成员
* 类与外部数据共享:友元
##### 静态数据成员
	* 用关键字`static`声明
	* 为该类的所有对象共享，静态数据成员具有静态生存期
	* 一般在类外初始化，用\(::)来指明所属的类
	* C++11支持静态常量\(`const`或`constexpr`修饰)类内初始化，此时类外仍可定义该静态成员，但不可再次初始化操作
静态成员举例
```C++
#include <iostream>
//C++标准程序库的所有标识符都被声明在std命名空间内
using namespace std;
class Point
{
public: //外部接口
	Point(int x = 0, int y = 0):x(x), y(x) //析构函数
	{
		//在构造函数中对cout累加，所有对象共同共同维护一个count
		count++;
	}
	Point(Point& p) //复制构造函数
	{
		x = p.x;
		y = p.y;
		count++;
	}
	~Point()  //析构函数
	{
		count--;
	}
	int getX()
	{
		return x;
	}
	int getY()
	{
		return y;
	}
	void showCount()
	{
		cout << " Object count:" << count << endl;
	}
private:
	int x, y;
	static int count;  //静态数据成员声明，用于记录点的个数
};
int::Point::count = 0;//静态数据成员定义和初始化，使用类名限定
int main(void)
{
	Point a(4, 5);
	cout << "Point A:" << a.getX() << "," << a.getY();
	a.showCount();//输出对象个数

	Point b(a);
	cout << "Point B:" << b.getX() << "," << b.getY();
	b.showCount();//输出对象个数
	return 0;
}
/*
* 输出：
Point A:4,4 Object count:1
Point B:4,4 Object count:2
*/
```
##### 类的静态函数成员
* 类外代码可以使用类名和作用域操作符来调用静态成员函数
* 静态成员函数主要用于处理该类的静态数据成员，可以直接调用静态成员函数
* 如果访问非静态成员，要通过对象来访问
```C++
	static void showCount()  //静态函数成员，主函数可以直接调用查看count
	{
		cout << " Object count:" << count << endl;
	}
private:
	int x, y;
	static int count;  //静态数据成员声明，用于记录点的个数
};
int Point::count = 0;//静态数据成员定义和初始化，使用类名限定
int main(void)
{
	Point::showCount(); //输出对象个数
```
#### 友元
* 友元是C++提供的一种破坏数据封装和数据隐藏的机制
* 通过经一个模块声明为另一个模块的友元，一个模块能够引用到另一个模块中本被隐藏的信息
* 可以使用友元函数和友元类
* 为了确数据的完整性，及数据封装与隐藏的原则，建议尽量不适用或少使用友元
#### 友元函数
* 友元函数是在类声明中由关键字friend修饰说明的非成员函数，在它的函数体中能够通过对象名访问private和protected成员
* 作用：增加灵活性，使程序员可以在封装和快速性方面做出合理选择
* 访问对象中的成员必须通过对象名
****
##### 使用友元函数计算两点间的距离
```C++
#include <iostream>
using namespace std;
class Point
{
public:
	Point(int x = 0, int y = 0) :x(x), y(y)
	{
	}
	int getX()
	{
		return x;
	}
	int getY()
	{
		return y;
	}
	friend float dist(Point& a, Point& b); //友元函数，只要在类体中就行
private: //私有函数成员
	int x, y;
};
float dist(Point& a, Point& b)
{
	double x = a.x - b.x;
	double y = a.y - b.y;
	return static_cast<float>(sqrt(x * x + y * y));
}
int main(void)
{
	Point p1(1, 1), p2(4, 5);
	cout << "This distance is:";
	cout << dist(p1, p2) << endl;
	return 0;
}
/*This distance is:5*/
```
#### 友元类
![友元类举例](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvicMgsRBhgayuYtLPvnzX9lIaCOw1eCP1BUAgJkgId0waWCYC4jW1HbjibibxCwibddWB8FXjXX4o9JQ/0?wx_fmt=jpeg "友元类举例")
****
`类的友元关系是单向的`：如果声明B类是A类的友元，B类的成员函数就可以访问A类的私有和保护数据，但A类的成员函数却不能访问B类的私有，保护数据
****
#### 共享数据的保护
* 对于既需要共享，又需要防止改变的数据应该声明为常类型(用const进行修饰)
* 对于不改变对象状态的成员函数应该声明为常函数
#### 常对象
* 用const修饰的对象
	* 使用const关键字说明的函数
	* 常成员函数不更新对象的数据成员
	* 常成员函数说明格式:
		* 类型说明符 函数名 （参数表) const;
		* 这里，const是函数类型的一个组成部分，因此在实现部分也要带const关键字
		* const关键字可以被用于参与对重载函数的区分
* 通过常对象只能调用它的常成员函数
* 常数据成员
	* 使用const说明的数据成员
```C++
#include <iostream>
using namespace std;
class R
{
public:
	R(int r1,int r2):r1(r1),r2(r2)
	{
	}
	void print();
	void print() const;//常成员函数
private:
	int r1, r2;
};
void R::print()
{
	cout << r1 << ";" << r2 << endl;
}
void R::print() const
{
	cout << r1 << ";" << r2 << endl;
}
int main(void)
{
	R a(5, 4);
	a.print();
	const R b(20, 52);
	b.print();  //调用void print() const
	return 0; 
}
```



```C++
A const a(3,4);  //a是常对象，不能被更新


```




****

