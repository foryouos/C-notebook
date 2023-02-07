### 数组
数组是具有一定顺序关系的若干相同类型变量的集合体，组成数组的变量。
#### 范围for循环
```C++
#include <iostream>
using namespace std;
int main(void)
{
	int a[10], b[10];
	for (int i = 0; i < 10; i++)
	{
		a[i] = i * 2 - 1;
		b[10 - i - 1] = a[i];
	}
	//遍历容器十分方便，遍历的另一种形式，从a中依次取出元素，进行操作
	//auto 自动类型，根据a的类型确定e的类型
	for (const auto& e : a)  //范围for循环，输出a中每个元素
		cout << e << " ";
	cout << endl;
	for (int i = 0; i < 10; i++) //下标迭代循环，输出b中每个元素
	{
		cout << b[i] << " ";
	}
	cout << endl;

	return 0;
}
```
****
#### 数组初始化
* 如果不作任何初始化，局部作用域的非静态数组中会存在垃圾数据，static数组中的数组默认初始化为0
* 如果只对部分元素初始化，剩下的未显式初始化的元素，将自动被初始化为零
#### 对象数组
* 定义对象数组
	* 类名 数组名[元素个数]
* 访问对象数组元素
	* 通过下标访问
	* 数组名[下标].成员名
* 对象数组初始化
	* 数组中每一个元素对象被创建时，系统都会调用类构造函数初始化该对象。
	* 通过初始化列表赋值
	* Point a\[2]=\{Point\(1,2),Point\(3,4)}
	* 如果没有为数组元素指定显式初始值，数组元素便使用默认值初始化(调用默认构造函数)
* 对象数组的析构
	* 当数组中每一个对象被删除时，系统都要调用一次析构函数
```C++
#include <iostream>
#include "Pont.h"
using namespace std;
Point::Point() :x(0), y(0)
{
	cout << "Default Constructor called." << endl;
}
Point::Point(int x, int y) :x(x), y(y)
{
	cout << "Constructor called." << endl;
}
Point::~Point()
{
	cout << "Destructor called" << endl;
}
void Point::move(int newX, int newY)
{
	cout << "Moving the point to(" << newX << "," << newY << ")" << endl;
	x = newX;
	y = newY;
}
int main(void)
{
	cout << "Entering main..." << endl;
	Point a[2];
	for (int i = 0; i < 2; i++)
	{
		a[i].move(i + 10, i + 20);
	}
	cout << "Exiting main..." << endl;
	return 0;
}
/*
* Entering main...
* Default Constructor called.
* Default Constructor called.
* Moving the point to(10,20)
* Moving the point to(11,21)
* Exiting main...
* Destructor called
* Destructor called
*/
```
### 指针
* 指针：内存地址，用于间接访问内存单元
* 指针变量:用于存放地址的变量
![指针变量](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbvicMgsRBhgayuYtLPvnzX9llE8Xe4ibd1m43EhagTrgffUobhpTjW9gDnRYSicHVD956NrMh7dIXfsw/0?wx_fmt=jpeg "指针变量")
* `零`可以赋给指针，表示空指针
* 向指针变量赋值的值必须是地址常量，不能是普通整数
* 允许定义或声明指向void类型的指针，该指针可以被赋予任何类型对象的地址
```C++
void *general;
```
* 以往用0或者NULL去表达空指针的问题
	* C/C++的NULL宏是个有很多潜在BUG的宏。因为有的库把其定义为整数0，有的定义成\(void*)0.
	* C++11使用nullptr关键字，是表达更准确，类型安全的空指针
#### 指向常量的指针
不能通过指向常量的指针改变所指对象的值，但指针本身可以改变，可以指向另外的对象
```C++
int a;
const int *p1 = &a; //p1是指向常量的指针
int b;
p1 = &b; //正确，p1本身的值可以改变
*p1 = 1;//编译时出错，不能通过p1改变所指的对象
b=6;
```
#### 指针类型的常量
若声明指针常量，则指针本身的值不能被改变，指针本身是常量
```C++
int a;
int * const p2 = &a;
*p2 = 3;  //可以
int b;
p2 = &b;  //错误，p2是指针常量，值不能被改变
```
#### 指针数组
* 数组的元素是指针型
```C++
Point *pa[2];
// 由pa[0],pa[1]两个指针组成
```
利用数组与二维数组对比比
```C++
#include <iostream>
using namespace std;

int main(void)
{
	int line1[] = { 1,0,0 };
	int line2[] = { 0,1,0 };
	int line3[] = { 0,0,1 };
	//定义整型指针数组并初始化
	int* Pline[3] = { line1,line2,line3 };
	cout << "Matrix test:" << endl;
	//输出矩阵
	for (int i = 0; i < 3; i++)
	{
		for (int j = 0; j < 3; j++)
		{
			cout << Pline[i][j] << " ";
		}
		cout << endl;
	}

	return 0;
}
/*
Matrix test:
1 0 0
0 1 0
0 0 1
*/
```
#### 以指针作为函数参数
* 需要数据双向传递时\(引用也可以达到响应效果)
	> 用指针作为函数的参数，可以使被调函数通过形参指针存取主调函数中实参指针指向的数组，实现数据的双向 传递
* 小传递一组数据，只传首地址运行效率比较高
	> 实参是数组名时形参可以是指针
```C++
/*
project:从浮点数中取整数和浮点数
注：浮点数在机器内部近似存储
*/
#include <iostream>
using namespace std;
void splitFloat(float x, int* intpart, float* fracPart)
{
	*intpart = static_cast<int>(x); //取x的整数部分
	*fracPart = x - *intpart; //取x的小数部分
}
int main(void)
{
	cout << "Enter 3 float point numbers:" << endl;
	for (int i = 0; i < 3; i++)
	{
		float x, f;
		int n;
		cin >> x;
		splitFloat(x, &n, &f);
		cout << "integer Part=" << n << " Fraction Part = " << f << endl;
	}
	return 0;
}
/*
Enter 3 float point numbers:
3.14
integer Part=3 Fraction Part = 0.14
9.7
integer Part=9 Fraction Part = 0.7
5
integer Part=5 Fraction Part = 0
*/
```
#### 指针常量
```C++
/* 
Project：有关常量的几个问题
*/
#include <iostream>
using namespace std;
const int N = 6;   //N值不可修改
int const M = 9;
const int* p; //*p不可修改,p可以更改指针
int const *q = 0; //可通过赋值指针修改
const int* const A = &N; //只可初始化，不可膝盖
int main(void)
{
	//N = 7;//错误
	//M = 5; //此值也不允许修改
	int a = 7;
	p = &a;  //地址不能改变，也不能*p赋值
	//*p = 9; //左值也不可修改
	p = &N;
	//*q = 9; //左值不可修改
	q = &a;
	cout << *q << endl;
	return 0;
}
```
****
```C++
/*
project:使用const进行权限管理
*/
#include <iostream>
using namespace std;
const int N = 6;
void print(const int* p, int n) //加const常量，其只有读的权限，不能修改
{
	cout << "{" << *p;
	for (int i = 1; i < n; i++)
	{
		cout << "," << *(p + i);
	}
	cout << "}" << endl;
}
int main(void)
{
	int array[N];
	for (int i = 0; i < N; i++)
	{
		cin >> array[i];
	}
	print(array, N);
	return 0;
}
/*1 2 3 4 5 6
{1,2,3,4,5,6}
*/
```
#### 指针类型的函数
```C++
存储类型 数据类型 *函数名()
{
	//函数体语句
}
```
注意：
* 不要将非静态局部地址用作函数的返回值
	* 在`子函数中定义局部变量`后将其~地址~返回给主函数，就是非法地址
* 返回的指针要确保在主调函数中是有效，合法的地址
	* 主函数中定义的数组，在子函数中对该数组元素进行某种操作后，返回其中一个元素的地址
	* 在子函数中通过动态内存分配new操作取得的内存地址返回给主函数是合法有效的，但是内存分配和释放不在同一级别，要注意不能忘记释放，避免内存泄露。
```C++
#include <iostream>
int* newintvar()
{
	int* p = new int();  
	return p; //返回的地址指向的是动态分配的空间
	//函数运行结束时，p中的地址仍有效
}
int main(void)
{
	int* newintvar();
	int* intptr = newintvar();
	*intptr = 5; //访问的是合法有效的地址
	delete intptr;  //如果忘记在这里释放，会造成内存泄露
	return 0;
}
```
`内存泄露`:程序中已分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。
#### 函数指针的定义:函数指针指向的是程序代码存储区。
```C++
存储类型 数据类型 (*函数指针名);
```
##### 函数指针的典型用途
* 通过函数指针调用的函数
	* 将函数的指针作为参数传递给一个函数，使得在处理相似事件的时候可以灵活的使用不同的方法
* 调用者不关心谁是被调用者
	* 需直到存在一个具有特定原型和限定条件的被调用函数
```C++
#include <iostream>
using namespace std;
int compute(int a, int b, int(*func)(int, int)) //函数指针
{
	return func(a, b);
}
int max(int a, int b)  //求最大值
{
	return ((a > b) ? a : b);
}
int min(int a, int b)  //求最小值
{
	return ((a < b) ? a : b);
}
int sum(int a, int b) //求和
{
	return a + b;
}
int main(void)
{
	int a, b, res;
	cout << "请输入整数a: ";
	cin >> a;
	cout << "请输入整数b：";
	cin >> b;

	res = compute(a, b, &max);
	cout << "Max of" << a << "and" << b << "is" << res << endl;
	res = compute(a, b, &min);
	cout << "Min of" << a << "and" << b << "is" << res << endl;
	res = compute(a, b, &sum);
	cout << "Sum of" << a << "and" << b << "is" << res << endl;
}
```

#### 对象指针
* 对象指针定义形式
	* 类名 * 对象指针名
```C++
Point a(5,10);
Point *ptr;
ptr = &a;
```
* 通过指针访问对象成员
	* 对象指针名->成员名
```C++
ptr->getx() 相当于(*ptr).getx();
```
****
```C++
#include <iostream>
using namespace std;
class Point
{
public:
	Point(int x = 0, int y = 0) :x(x), y(y)
	{
	}
	int getX() const 
	{ 
		return x; 
	}
	int getY() const
	{
		return y;
	}
private:
	int x, y;
};
int main(void)
{
	Point a(4, 5);
	Point* p1 = &a;  //定义对象指针，用a的地址初始化
	cout << p1->getX() << endl; //用指针访问对象成员
	cout << a.getY() << endl;	//用对象名访问对象成员
	return 0;
}
```
#### this 指针
* 隐含于类的每一个非静态成员函数中
* 指出成员函数所操作的对象
	* 当通过一个对象调用成员函数时，系统先将该对象的地址赋给this指针，然后调用成员函数，成员函数对对象的数据成员进行操作时，就隐含使用了this指针
	* Point类的getX函数中return x相当于return this->x;

#### 前向引用声明
```C++
class Fred; //前向引用生命
class Barney
{
	Fred *x;  //不使用指针的话，由于编译器不知道fred细节，没法分配空间等操作，将报错，使用对象指针达到效果
}
class Fred
{
	Barney y;
}
```
#### 动态内存分配
* new 类型名T\(初始化参数)
* 功能:
	* 在程序执行期间，申请用于存放T类型对象的存储空间，并依次初始化参数进行初始化。
	* 基本类型初始化:如果有初始化参数，依初始化参数进行初始化，如果没有括号和初始化参数，不进行初始化，新分配的内存中内容不确定，如果有括号但初始化参数为空，初始化为0
	* 对象类型：如果有初始化参数，以初始化参数中的值为参数调用构造函数进行初始化，如果没有括号和初始化参数或者有括号但初始化参数为空，用默认构造函数初始化
* 结果值：成功：T类型的指针，指向新分配的内存，失败：抛出异常
****
#### 动态申请动态数组
* new 类型名T[表达式][常量表达式]......()
* 功能:
	* 在程序执行期间，申请用于存档T类型对象数组的内存空间，可以有"\()"但初始化列表必须为空
	* 如果有"\()",对每个元素的初始化与执行"new T\()"所做进行初始化的方式相同
	* 如果没有"\()",对每个元素的初始化与执行"new T"所做进行初始化的方式相同
* 结果值
	* 如果内存申请成功,返回一个指向新分配内存首地址的指针
	* 如果失败，抛出异常

```C++
double* array = new double[n]();
char(*fp)[3];  //一维数组
fp = new char[n][3];
```
#### 释放内存操作符delete
* delete 指针p
* 功能:释放指针p所指向的内存，p必须是new操作的返回值
* delete[] 指针p
* 功能:释放指针p所指向的数组，p必须是用new分配得到的数组首地址
****
```C++
#include <iostream>
using namespace std;
class Point
{
public:
	Point(int x, int y)
	{
		cout << "Destructor called." << endl;
	};
	Point() :x(x), y(y)
	{
		cout << "Default Destructor called." << endl;
	};
	~Point()
	{
		cout << "Destructor called" << endl;
	}
	int getX() const { return x; } //常函数
	int getY() const { return y; }
	void move(int newX, int newY)
	{
		x = newX;
		y = newY;
	}
private:
	int x, y;
};

int main(void)
{
	cout << "step one:" << endl;
	Point* ptr1 = new Point;//调用默认构造函数
	delete ptr1;  //删除对象，自动调用析构函数

	cout << "Step two" << endl;
	ptr1 = new Point(1, 2);
	delete ptr1;
	return 0;
}
/*
step one:
Default Destructor called.
Destructor called
Step two
Destructor called.
Destructor called
*/
```
****
```C++
#include <iostream>
using namespace std;
class Point
{
public:
	Point(int x, int y)
	{
		cout << "Destructor called." << endl;
	};
	Point() :x(x), y(y)
	{
		cout << "Default Destructor called." << endl;
	};
	~Point()
	{
		cout << "Destructor called" << endl;
	}
	int getX() const { return x; } //常函数
	int getY() const { return y; }
	void move(int newX, int newY)
	{
		x = newX;
		y = newY;
	}
private:
	int x, y;
};

int main(void)
{
	Point* ptr = new Point[2];//创建对象数组
	ptr[0].move(5, 10); //通过指针访问数组元素的成员
	ptr[1].move(15, 20); 
	cout << "Deleting..." << endl;
	delete[] ptr; //删除整个对象数组
	return 0;
}
/*
Default Destructor called.
Default Destructor called.
Deleting...
Destructor called
Destructor called
*/
```
#### 创建动态数组
```C++
#include <iostream>
using namespace std;
int main(void)
{
	int n;
	cin >> n;
	//创建动态内存分配孔家
	int(*cp)[9][8] = new int[n][9][8]; //第一个为表达式，后面为常量
	for (int i = 0; i < 7; i++)
	{
		for (int j= 0; j < 9; j++)
		{
			for (int k = 0; k < 8; k++)
			{
				*(*(*(cp + i) + j) + k) = (i * 100 + j * 10 + k);
			}
		}
	}
	for (int i = 0; i < 7; i++)
	{
		for (int j = 0; j < 9; j++)
		{
			for (int k = 0; k < 8; k++)
			{
				cout << cp[i][j][k] << " ";
			}
			cout << endl;
		}
		cout << endl;
	}

	delete[] cp; //释放整个动态多维数组空间

}
```
#### 将动态数组封装成类
* 更加简介，便于管理
	* 建立和删除数组的过程比较繁琐
	* 封装成类后更加简洁，便于管理
* 可以在访问数组元素前检查下标是否越界
	* 用assert来检查，assert只在调试时生效



![皮卡丘](https://p.ananas.chaoxing.com/star3/origin/13ed5511d475673295812f74c6a833dd.png)























