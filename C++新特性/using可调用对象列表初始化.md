# 定义别名
> `using` 和`typedef`一样，并不会创建新的类型，他们只是给某些类型定义了`新的别名`，using相教于typedef的优势在于定义函数`更加直观`，并且可以给模版`定义别名` 而`typedef `无法`定义模版`

```cpp
typedef 旧的类型名 新的类型名;
typedef unsigned int uint_t;
//使用using 定义
using 新的类型 = 旧的类型;
using uint_t = int;
//定义函数指针对比
typedef int(*func)(int,double);
using func = int(*)(int,double);
```
## 使用
```cpp
#include <iostream>
#include <functional>
#include <map>
using namespace std;
// typename定义模版
template <typename T>
struct MyMap
{
	typedef map<int, T> type;
};
//使用using定义模版
template <typename T>
using mymap = map<int, T>;
int main(void)
{
	MyMap<string>::type map;
	map.insert(make_pair(1,"Hello")); 
	map.insert(make_pair(2, "Word!"));

	mymap<string> m;
	m.insert(make_pair(1, "Hello using"));
	m.insert(make_pair(2, "word using"));
	return 0;
}
```
# 列表初始化

## 统一初始化
```cpp
#include <iostream>
using namespace std;
class  Test
{
public:
	Test(int)
	{
	};

private:
	Test(const Test&);

};


int main(void)
{
	//数组初始化
	int array[] = { 1,3,5,7,9 };
	double array1[3] = { 1.2,1.3,1.4 };
	//对象初始化
	struct Person
	{
		int id;
		double salary;
	}zhang3 {1, 3000};

	//对象初始化
	Test t1(520);  //最初初始化
	Test t2 = 520; //错误 拷贝构造函数是私有的,Linux会报错，VS不报错
	Test t3 = { 520 }; //C++11新特性
	Test t4{ 520 };
	//基础列表初始化
	int a1 = { 1314 };
	int arr1[] = { 1,2,3 };
	// C++11新特性，可直接在变量后边跟上初始化列表，来进行变量或对象的初始化
	int a2{ 1314 };
	int arr2[] = { 1,2,3 };

	//使用new操作符初始化
	int* p = new int{ 520 }; //指向一个new操作符返回的内存，通过列表初始化将内存数据初始化为520
	double b = double{ 52.11314 }; //匿名对象初始化，再进行拷贝初始化
	int* array = new int[3] { 1, 2, 3 }; // 在堆上动态分配一块内存，通过列表初始化直接完成多个元素初始化

	return 0;
}
```
## 初始化函数返回值
```cpp
#include <iostream>
#include <string>
using namespace std;
class Person
{
public:
	Person(int id, string name)
	{
		cout << "id: " << id << ",name: " << name << endl;
	}
};
Person func()
{
	return { 9527,"华安" };  

}
int main(void)
{
	Person p = func();
	return 0;
}
/*
id: 9527,name: 华安
*/
```

## 细节

### 聚合体

```cpp
#include <iostream>
#include <string>
using namespace std;
struct T1
{
	int x;
	int y;
}a = {123,321};  //对应的类型是聚合体才会将列表中的数据初始化对象
//对象a对自定义聚合类型进行了初始化，a将以拷贝的形式使用初始化列表中的数据来初始化T1结构体的变量
struct T2
{
	int x;
	int y;
	T2(int, int) :x(10), y(20) {}; //世界成员通过构造函数构造完成
}b={123,321};

int main(void)
{
	cout << "a.x" << a.x << ", a.y" << a.y << endl;
	cout << "b.x" << b.x << ", b.y" << b.y << endl;
	/*
	a.x123, a.y321
	b.x10, b.y20
	*/
	return 0;
}
```

###  什么是聚合体

* 普通数组本身可以看做是一个聚合类型
* 满足以下条件的类(class,struct,union) 可以看做是一个聚合类型
  * `无用户自定义`的构造函数
  * `无私有或保`护的非静态数据成员
    * 类中`有私有成员`，无法使用列表初始化进行初始化
    * 类中`有非静态成员`可以通过列表初始化进行初始化，但它不能初始化静态成员变量
  * `无基类`
  * 无虚函数 `virtual`
  * C++11类中不能有使用{}和=直接初始化的非静态数据成员(从`C++14开始`，使用列表初始化也可以初始化在类中`使用{}和=初始化`过的非静态数据成员)

```cpp
//普通数据本身就是一个聚合类型
int x[] = { 1,2,3,4,5,6 };
double y[3][3] =
{
	{1.23,2.34,3.45},
	{4.45,5.34,6.34},
	{7.34,8.43,9.43},
};
char carry[] = { 'a','b','c' };
string asrry[] = { "hello","world" };


struct T3
{
	int x;
	int y;
protected:
	static int z;
	
}b = { 123,321 }; //初始化x,y
//静态成员的初始化
int T3::z = 2;

struct T4
{
	int x;
	double y = 1.34;
	int z[3]{ 1,2,3 };
};

T4 t{ 520,13.14,{6,7,8} }; //C++11不支持，C++14支持
```



### 非聚合体

> 在类中内部定义一个构造函数，在构造函数中使用初始化列表对类成员变量进行初始化。
>
> 对于聚合类型，使用列表初始化相当于对其中的每个元素分别赋值，而对于非聚合类型，则需要先自定义一个合适的构造函数，此时使用列表初始化将会调用它对应的构造函数。

```cpp
#include <iostream>
#include <string>
using namespace std;
struct T1
{
	int x;
	double y;
	//在构造函数中使用初始化列表初始化类成员
	T1(int a, double b, int c) : x(a), y(b), z(c) {};
	virtual void print()
	{
		cout << "x: " << x << ",y :" << y << ",z :" << z << endl;
	}
private:
	int z;
};


struct T3
{
	int x;
	double y;
private:
	int z;
};
//聚合类型并非递归的，当一个类的非静态成员是非聚合类型时，这个类也可能是聚合类型
struct T2
{
	T3 t1;
	long x1;
	double y1;
};

int main(void)
{
	T1 t{ 520,13.14,1314 }; //基于构造函数使用初始化列表初始化成员
	t.print();

	T2 t2{ {},520,13.14 };

	return 0;
}
```





## std::initializer_list
> 在STL容器中，可以进行`任意长度`相同的数据初始化，

* 内部定义了`迭代器iterator`等容器概念，遍历时得到的迭代器是`只读`的
* 对于`std::initializer_list<T>`而言，它可以接受`任意长度`的初始化列表，但是要求元素`必须`是``同种类型T`
* 三个接口:`size()`,`begin()`,`end()`
* `std::initializer_list` 对象只能被`整体初始化`或者`赋值`

### 作为构造函数参数
```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;
void traversal(std::initializer_list<int> a)
{

}
class Test
{
public:
	Test(std::initializer_list<string> list)
	{
		for (auto it = list.begin(); it != list.end(); ++it)
		{
			cout << *it << " ";
			m_names.push_back(*it);
		}
		cout << endl;
	}
private:
	vector<string> m_names;
};
int main(void)
{
	Test t({ "jack","lucy","tom" });  //使用大括号
	Test t1({ "hello","world","nihao","shijie" });
	return 0;
}
/*
jack lucy tom
hello world nihao shijie
*/
```
# 可调用对象

## 函数指针

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;
void print(int a, double b)
{
	cout << a << b << endl;
}
//定义函数指针
void (*func)(int, double) = &print;
int main(void)
{
	print(1, 2.3);
	func(1, 2.3);
	return 0;
}
/*
输出:
12.3
12.3
*/
```
## 伪函数

> operator()成员函数的类对象

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;
struct Test
{
	// operator() 操作父重载,括号内为空
	void operator()(string msg)
	{
		cout << "msg :" << msg << endl;
	}
};
int main(void)
{
	Test t;
	t("操作符重载");
	return 0;
}
/*
msg :操作符重载
*/
```
## 可被转换为函数指针的类对象

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

using func_ptr = void(*)(int, string);

struct Test
{
	static void print(int a, string b)
	{
		cout << a << b << endl;
	}
	// 将类对象转为函数指针
	operator func_ptr()
	{
		return print;
	}
};
int main(void)
{
	Test t;
	t(520,"类对象转为函数指针");
	return 0;
}
/*
520类对象转为函数指针
*/
```
## 类成员函数指针或者类成员指针

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;
struct Test
{
	void print(int a, string b)
	{
		cout << a << b << endl;
	}
	int num;
};
int main(void)
{
    // 使用using
    using fptr = vod(Test::*)(int,string);
    fptr = Test::print;
    // 定义类成员函数指针指向类成员函数
	void (Test:: * func_ptr)(int, string) = &Test::print;
	// 类成员指针指向类成员变量
	int Test::* obj_ptr = &Test::num;
    //使用是哪个
    using  ptr1 = int Test::*;
    ptr1 pt = &Test::num;

	Test t;
	// 通过类成员函数指针调用类成员函数
	(t.*func_ptr)(01, "端午节");
	// 通过类成员指针初始化类成员变量
	t.*obj_ptr = 22;
	cout << "number is:" << t.num << endl;
	return 0;
}
/*
1端午节
number is:22
*/
```
# 包装器
> `std::function`是`可调用对象的包装器`。它是一个`类模版`，可以`容纳`除了`类成员(函数指针)`之外的`所有可调用对象`。通过统一的方式处理函数，函数对象，函数指针，并`允许``保存和延迟`执行它们。

## 基本使用
```cpp
#include <functional>
std::function(返回值类型(参数类型列表)) diy_name = 可调用对象;
```
## 用法
```cpp
#include <iostream>
#include <functional>
using namespace std;

int add(int a, int b)
{
	cout << a << "+" << b << "=" << a + b << endl;
	return a + b;
}
class T1
{
public:
	static int sub(int a, int b)
	{
		cout << a << "-" << b << "=" << a - b << endl;
		return a - b;
	}
};
class T2
{
public:
	int operator()(int a, int b)
	{
		cout << a << "*" << b << "=" << a * b << endl;
		return a * b;
	}
};
int main(void)
{
	// 绑定一个普通函数
	function<int(int, int)> f1 = add;
	// 绑定以静态类成员函数
	function<int(int, int)> f2 = T1::sub;
	// 绑定一个仿函数
	T2 t; // t本身就只T2类仿函数
	function<int(int, int)> f3 = t;

	// 函数调用
	f1(9, 3);
	f2(9, 3);
	f3(9, 3);
	return 0;
}
/*
9+3=12
9-3=6
9*3=27
*/
```
## 作为回调函数使用
> 回调函数本身是通过函数指针实现的，使用对象包装器可以取代函数指针的使用
> 通过包装器std::function非常方便的将仿函数转换为一个函数指针，通过进行函数指针的传递，在其他函数的合适位置就可以调用这个包装好的仿函数。使用std::function作为函数的传入参数，可以将定义不相同的可调用对象进行统一的传递，这样大大添加了程序的灵活性。

```cpp
#include <iostream>
#include <functional>
using namespace std;
class A
{
public:
	//构造函数参数是一个包装器对象
	A(const function<void()>& f) :callback(f)
	{
	}
	void notify()
	{
		callback(); //调用通过构造函数得到的函数指针
	}

private:
	function<void()> callback;
};
class B
{
public:
	void operator()()
	{
		cout << "仿函数包装的函数" << endl;
	}
};

int main(void)
{
	B b;
	A a(b); //仿函数通过包装器对象进行包装

	cout << "调用回调函数之前" << endl;
	
	a.notify();  //调用回调函数

	return 0;
}
/*
调用回调函数之前
仿函数包装的函数
*/
```
# 绑定器
> `std::bind`用来将`可调用对象与其参数`一起进行绑定。绑定后的结果可以使用`std::function`进行保存，并延迟到任何需要的时间执行

* 将可调用`对象与其参数`一起绑定成一个`仿函数`
* 将`多元`(参数个数为`n，n>1`)可调用对象转换为`一元`或者`(n-1)元`可调用对象，即只绑定`部门参数`
## 使用语法

> 使用std::bind绑定器返回的是一个仿函数类型，得到的返回值可以直接赋值给std::function，在使用的时候，并不需要关心返回值类型，使用auto进行自动类型推导即可。`placeholders::_1 `是一个占位符,代表位置将在函数调用时被传入的第一个参数所替代，`placeholders::_2` `placeholders::_3`分别代表第二个第三个，且顺序如参数相对应。

```cpp
// 绑定非类成员函数/变量
auto f = std::bind(可调用对象地址，绑定的参数/占位符)
// 绑定类成员函/变量
auto f = std::bind(类函数/成员地址,类实例对象地址,绑定的参数/占位符)
```

## 绑定案例

```cpp
#include <iostream>
#include <functional>
using namespace std;

//传参，和函数
void callFunc(int x, const function<void(int)>& f)
{
	if (x % 2 == 0)
	{
		f(x);  //此f(x)为包装器的回调函数，callFunc的参数
	}
}
void output(int x)
{
	cout << x << " ";
}
void output_add(int x)
{
	cout << x + 10 << " ";
}
int main(void)
{
	// 使用绑定器可调用对象和参数
	auto f1 = bind(output, placeholders::_1);  //_1 可依次累加，代表第几个参数
	for (int i = 0; i < 10; ++i)
	{
		callFunc(i, f1); //i参数会在callfunc函数中传递给f1
	}
	cout << endl;

	auto f2 = bind(output_add, placeholders::_1);
	for (int i = 0; i < 10; i++)
	{
		callFunc(i, f2);
	}
	cout << endl;
	return 0;
}
/*
0 2 4 6 8
10 12 14 16 18
*/
```

## 绑定器与包装器配合

> `f1`的类型是`function<void(int,int)>`,通过使用`std::bind`将`Test`的成员函数`output`的地址和`对象t`绑定，并转化为一个`仿函数`并存储到`对象f1`中。
>
> 使用`绑定器绑定`的类成员变量`m_number`得到的`仿函数被`存储到了类型`function<int&(void)>`的包装器`对象f2`中，并且可以在需要的时候修改`这个成员`。并且可以在需要的时候修改这个成员。其中`int`是绑定的`类成员的类`型，并且允许修改绑定的变量，因此需要指定为变量的引用，由于没有参数因此参数列表指定为`void.`

```cpp
#include <iostream>
#include <functional>
using namespace std;
class Test
{
public:
	void output(int x, int y)
	{
		cout << "x: " << x << "y: " << y << endl;
	}
	int m_number = 100;
};
int main(void)
{
	Test t;
	//绑定类成员函数
	function<void(int, int)>f1 = 
		bind(&Test::output, &t, placeholders::_1, placeholders::_2);
	// 绑定类成员变量(公共)
	function<int& (void)> f2 = bind(&Test::m_number, &t);
	//调用
	f1(520, 1314);
	f2() = 2323;
	cout << "t.m_number: " << t.m_number << endl;
	return 0;
}
/*
x: 520y: 1314
t.m_number: 2323
*/
```

