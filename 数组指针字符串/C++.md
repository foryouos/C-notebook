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






