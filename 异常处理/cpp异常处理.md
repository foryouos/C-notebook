#### 异常
* 异常是程序运行过程中出现问题
* \"异常"并不经常出现
* 处理异常使程序像没有发生过问题一样继续执行
#### 何时使用异常处理

* 异常处理 用于处理同步错误\(语句执行时发生的错误)
* 常见的异常处理错误:
> 数组下标越界，算法溢出，被零除，函数参数不合法，内存不够引起的内存分配不成功
* 异常处理不是用于处理异步事件相关的错误的
> 磁盘读写结束，网络信息到达，点击鼠标或键盘，这些与程序的控制流程并行，独立

#### 异常处理的基本思想

![异常处理额基本思想](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtdBEE0yRM2cl2h5S9z9t0B9HbyfO9KGNscBicHTwbSn5lK5u8K1mqqK4mb7F4m4rQicLVPoNNCWiaOA/0?wx_fmt=png "异常处理额基思想")
#### 异常处理的语法
* 抛掷异常的程序段
```cpp
throw 表达式; //表达式子可以是变量也可以是语句或小段代码
```
* 捕获并处理异常的程序段
```cpp
try
	复合语句(保护段)
catch (异常声明)
	复合语句(异常处理程序)
catch (异常声明)
	复合语句
	...
```
* 若有异常则通过`throw`创建一个异常对象并抛掷
* 将可能抛出异常的程序段嵌在`try`块之中。通过正常的顺序执行到达`try`语句，然后执行try块内的保护段
* 如果在保护段执行期间没有引起异常，那么跟在try块后的catch子句就不执行。程序从try块后的最后一个catch子句后面的语句继续执行
* catch子句按其在try块后出现的顺序被检查。匹配的catch子句将被捕获并处理异常(或继续抛掷异常)
* 如果匹配的处理器未找到，则库函数`terminate`将被自动调用，其默认是调用`abort终止程序`

```cpp
/* 
处理除零异常
*/
#include <iostream>
using namespace std;

int divide(int x, int y)
{
	if (y == 0)
		throw x; //通过throw创建x的异常对象抛掷给x
	return x / y;
}
int main(void)
{
	try
	{
		cout << "5 / 2 = " << divide(5, 2) << endl;
		cout << "8 / 0 = " << divide(8, 0) << endl;
		cout << "7 / 1 = " << divide(7, 1) << endl;
	}
	catch (int e)
	{
		cout << e << " is divided by zero !" << endl;
	}
	cout << "That is ok." << endl;
	return 0;
}
/*
5 / 2 = 2
8 is divided by zero !
That is ok.
*/
```
****
```cpp
/*
带析构语句的类的C++异常处理
*/
#include <iostream>
#include <string>
using namespace std;
class MyException
{
public:
	MyException(const string& message) :message(message)
	{

	}
	~ MyException() {}
	const string& getMessage() const
	{
		return message;
	}
private:
	string message;
};
class Demo
{
public:
	Demo()
	{
		cout << "Constructor of Demo" << endl;
	}
	~Demo()
	{
		cout << "Destructor of Demo" << endl;
	}
};
void func() throw(MyException)
{
	Demo d;
	cout << "Throw MyException in func() " << endl;
	throw MyException("exception thrown by func()");
}
int main(void)
{
	cout << "In main function" << endl;
	try
	{
		func();
	}
	catch (MyException& e)
	{
		cout << "Caught an exception: " << e.getMessage() << endl;
	}
	cout << "Resume the exception of main()" << endl;
	return 0;
}
/*
In main function
Constructor of Demo
Throw MyException in func()
Destructor of Demo
Caught an exception: exception thrown by func()
Resume the exception of main()
*/
```
#### 异常接口声明

> 异常声明(异常规范,`Exception specifications`)一个函数显式声明可能抛出的异常，有利于函数的调用者为异常处理做好准备
* 可以在函数的声明中列出这个哈数可能抛掷的所有异常类型
```cpp
void fun() throw(A,B,C,D);
```
* 若无异常接口声明，则此函数可以抛掷`任何类型`的异常
* 不抛掷任何异常的函数声明如下
```cpp
void fun() throw();
```
#### `noexcept`异常说明

* 对明确`不会`抛出异常的函数使用`noexcept`说明修饰
* 声明方式:
```cpp
返回值类型 func(形参列表) noexcept;
```
* 异常处理使编译和运行时有`额外开销`，省去异常处理可优化`加速调用`
* 需保持该函数内部调用函数和定义语句均不会抛出异常的一致性
* noexcept运算符，可判断函数是否使用了noexcept说明
```cpp
void f() noexcept;
noexcept(f());  //返回true，因为f有noexcept说明
```
#### 慎用异常声明的情况
* 对于带类型参数的函数模版，要尽量避免使用exception specifications,因为不同类型对于相同行为的定义不同，抛出的异常也就不同，因而函数模版很难或不可能确定它具现代的函数实体所可能抛出的异常
* 使用回调\(callback)函数时
* 系统可能抛出的异常
#### 异常处理中的构造与析构
##### 自动的析构
* 找到一个匹配的catch异常处理后
	* 初始化异常参数
	* 将从对应的try块开始到异常被抛掷之间构造(且尚未析构)的所有自动对象进行析构
	* 从最后一个catch处理之后开始恢复执行

#### 标准程序库继承关系

![标准程序库继承关系](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtdBEE0yRM2cl2h5S9z9t0BrgiamoaUWaML3MGFNqzEWQ6V3mq3smy3HsubAcYvRn6EnAsCqAb31Ng/0?wx_fmt=png)

#### C++标准库各种异常类所代表的异常

| **异常类**            | **头文件** | **异常的含义**                                               |
| --------------------- | ---------- | ------------------------------------------------------------ |
| **bad_alloc**         | exception  | 用new动态分配空间失败                                        |
| **bad_cast**          | new        | 执行dynamic_cast失败（dynamic_cast参见8.7.2节）              |
| **bad_typeid**        | typeinfo   | 对某个空指针p执行typeid(*p)（typeid参见8.7.2节）             |
| **bad_exception**     | typeinfo   | 当某个函数fun()因在执行过程中抛出了异常声明所不允许的异常而调用unexpected()函数时，若unexpected()函数又一次抛出了fun()的异常声明所不允许的异常，且fun()的异常声明列表中有bad_exception，则会有一个bad_exception异常在fun()的调用点被抛出 |
| **ios_base::failure** | ios        | 用来表示C++的输入输出流执行过程中发生的错误                  |
| **underflow_error**   | stdexcept  | 算术运算时向下溢出                                           |
| **overflow_error**    | stdexcept  | 算术运算时向上溢出                                           |
| **range_error**       | stdexcept  | 内部计算时发生作用域的错误                                   |
| **out_of_range**      | stdexcept  | 表示一个参数值不在允许的范围之内                             |
| **length_error**      | stdexcept  | 尝试创建一个长度超过最大允许值的对象                         |
| **invalid_argument**  | stdexcept  | 表示向函数传入无效参数                                       |
| **domain_error**      | stdexcept  | 执行一段程序所需要的先决条件不满足                           |

#### 标准异常类的基础
* `exception`：标准程序库异常类的`公共基类`
* `logic_error`：表示可以在程序中被预先检测到的异常
	* 如果小心地编写程序，这类异常能够避免
* `runtime_error`:表示`难以`被预先检测的异常

```cpp
/*
计算三角形的面积
*/
#include <iostream>
#include <cmath>
#include <stdexcept>
using namespace std;
//给出三角形三边长，计算三角形面积
double area(double a, double b, double c)
{
	//判断三角形表长是否为正
	if (a <= 0 || b <= 0 || c <= 0)
	{
		throw invalid_argument("the side length should be positive");
	}
	//判断三边长是否满足三角不等式
	if (a + b <= c || b + c <= a || c + a <= b)
	{
		throw invalid_argument("the side length should fit the triangle inequation");
	}
	//由heron公式计算三角形面积
	double s = (a + b + c) / 2;
	return sqrt(s * (s - a) * (s - b) * (s - c));
}
int main(void)
{
	double a, b, c; //三角形三边长
	cout << "Please input the side lengths of a triangle:";
	cin >> a >> b >> c;
	try
	{
		double s = area(a, b, c);//尝试计算三角形面积
		cout << "Area:" << s << endl;
	}
	catch (exception& e)
	{
		cout << "Error:" << e.what() << endl;
	}
	return 0;
}
```

