### C++11新特征与MFC
#### `auto`

* 可以从初始化表达式中判断出变量的类型
* 属于编译器特性，不影响最终机器码质量，不影响运行效率
```cpp
auto i = 10; //int 
auto str = "C++"; //const char*
auto p = new Person(); //Person *
```
####  `decltype`

> 可以获取`变量的类型`
####  `nullptr`

* 可以解决`NULL`的二义性问题
```cpp
#include <iostream>
using namespace std;

void func(int v)
{
	cout << "func(int) - " << v << endl;
}
void func(int *v)
{
	cout << "func(int *) - " << v << endl;
}
int main(void)
{
	auto a = 10;
	func(NULL);//NULL为宏定义0
	int* p = nullptr; //空指针使用nullptr
	func(p);
	return 0;
}
```
#### 快速遍历
```cpp
//快速遍历
	int array[] = { 11,22,33,44,55 };
	for (int item : array)
	{
		cout << item << endl;
	}
```
##### 更加简介的初始化方式
```cpp
int array[]{11,22,33,44,55};
```
####  `lambada`表达式

> 类似于`JavaScript的闭包`，`IOS`中的`Block`本质上就是函数
```cpp
//完整结果
[capture list](params list) mutable exception ->return type{ function body}
```
* `capture list`:捕获外部变量列表
* `params list`:形参列表，不能使用默认参数，不能省略参数名
* `mutable`:用来说用是否可以修改捕获的变量
* `exception`:异常设定
* `return type`:返回值类型
* `function body`:函数体
**有时可以省略部分结果**
```cpp
[capture list](params list)  ->return type{ function body}
[capture list](params list){ function body}
[capture list]{ function body}
```
```cpp
	//最简单的调用此表达式
	([]
		{
			cout << "funvc()----" << endl;
		}
	)();
void (*p)() = []
		{
			cout << "funvc()----" << endl;
		};

	p(); //调用lambda表达式
	//计算两个整数和
	auto q = [](int a,int b) -> int
	{
		 return a + b;
	};

	cout << q(10,20) << endl;

```
****
```cpp
#include <iostream>
using namespace std;

int exec(int v1, int v2, int (*func)(int, int))
{
	return func(v1, v2);
}
int main(void)
{

	cout << exec(20, 10, [](int v1, int v2)
		{
			return v1 + v2;
		}) << endl;
	cout << exec(20, 10, [](int v1, int v2)
		{
			return v1 - v2;
		}) << endl;
	cout << exec(20, 10, [](int v1, int v2)
		{
			return v1 * v2;
		}) << endl;
	return 0;
}
```
#### 变量捕获
```cpp
#include <iostream>
using namespace std;

int exec(int v1, int v2, int (*func)(int, int))
{
	return func(v1, v2);
}


int main(void)
{
	int a = 10;
	//int b = 10;
	//auto func = [a, b]
	//{
	//	cout << a << endl;
	//	cout << b << endl;
	//};
	//a = 20;
	//func(); //值捕获，并没有获取最新的a值

	//auto func = [&a] //使用地址捕获，可以保证获取最新的a值
	//{
	//	cout << a << endl;
	//};

	//a = 70; 
	//func(); //地址捕获获取最新值

	//值捕获不能加加
	//auto func = [a]
	//{
	//	a++; //值捕获不能++
	//};
	//func();
	//地址捕获可以
	//auto func = [&a]
	//{
	//	a++;
	//};
	//func();
	// cout << a << endl; //值a+1
	//mutable 
	auto func = [a] () mutable //内部a可以相加
	{
		int b = a;
		b++;
		cout << "lambda = " << b << endl;
	};
	func();
	cout << a << endl;


	return 0;
}
```
#### 智能指针(`Smart Pointer`)

##### 传统指针的问题
* 需要手动管理内存
* 容易发生内存泄露(忘记释放，出现异常等)
* 释放之后残生野指针
* 智能指针就是为了解决传统指针存在的问题
##### 智能指针
* `auto_ptr`:属于C++98标准，在C++11中已经`不推荐`使用
* `shared_ptr`:C++11标准
* `unique_ptr`:C++11标准

##### 使用`auto_ptr`自适应指针

* `不`能用于数组
```cpp
#include <iostream>
using namespace std;
class Person {
	int m_age;
public:
	Person() {
		cout << "Person()" << endl;
	}
	Person(int age) :m_age(age) {
		cout << "Person(int)" << endl;
	}
	~Person() {
		cout << "~Person()" << endl;
	}
	void run() {
		cout << "run() - " << m_age << endl;
	}
};
//创建简单的智能指针
template <typename T>
class SmartPointer
{
private:
	T *m_obj;
public:
	SmartPointer(T* obj) : m_obj(obj)
	{

	}
	~SmartPointer()
	{
		if (m_obj == nullptr) return;
		delete m_obj;
	}
	T* operator->()
	{
		return m_obj;
	}
};
int main()
{
	/*Person *p = new Person(20);
	p->run();*/

	// 可以理解为：智能指针p指向了堆空间的Person对象
	cout << 1 << endl;
	{
		auto_ptr<Person> p(new Person(20));
		p->run();
	}
	cout << 2 << endl;

	/*Person p(20);
	p.run();*/
	return 0;
}
```
##### `share_ptr`

> 设计理念:多个`share_ptr`可以指向同一个对象，当`最后`一个`shared_ptr`在作用域范围内结束时，对象才会被自动释放
* 一个`shared_ptr`会对一个对象产生强引用(`strong reference`)
* 每个对象都有个与之对应的强引用计数，记录着当前对象被多少个`share_ptr`强引用着
* 可以通过`shared_ptr`的`use_count`函数获得强引用计数
* 当有一个新的`shared_ptr`指向对象时，对象的强引用计数就会+1
* 当有一个`shared_ptr`销毁时(比如作用域结束)对象的强引用就会-1

```cpp
#include <iostream>
using namespace std;
class Person {
	int m_age;
public:
	Person() {
		cout << "Person()" << endl;
	}
	Person(int age) :m_age(age) {
		cout << "Person(int)" << endl;
	}
	~Person() {
		cout << "~Person()" << endl;
	}
	void run() {
		cout << "run() - " << m_age << endl;
	}
};
//创建简单的智能指针
template <typename T>
class SmartPointer
{
private:
	T *m_obj;
public:
	SmartPointer(T* obj) : m_obj(obj)
	{

	}
	~SmartPointer()
	{
		if (m_obj == nullptr) return;
		delete m_obj;
	}
	T* operator->()
	{
		return m_obj;
	}
};
int main()
{
	cout << 1 << endl;
	{
		//可以
		//shared_ptr<Person> p(new Person(20));
		//若为数组
		//shared_ptr<Person[]> p(new Person[5]);
		//当多个智能指针指向一个对象
		shared_ptr<Person> p4; //当最后一个对象销毁后对象才会销毁
		{
			shared_ptr<Person> p1(new Person(10));
			cout << p1.use_count() << endl; //返回使用强引用的数量
			shared_ptr<Person> p2 = p1;
			cout << p2.use_count() << endl;
			shared_ptr<Person> p3 = p2;
			cout << p3.use_count() << endl;
			p4 = p3;
			cout << p4.use_count() << endl;
		}
		cout << 2 << endl;
		cout << p4.use_count() << endl;
	}
	cout << 3 << endl;
	return 0;
}
/*
1
Person(int)
1
2
3
4
2
1
~Person()
3

*/
```
##### 循环引用
* `weak_ptr`可以解决`share_ptr`循环引用导致强引用
```cpp
#include <iostream>
using namespace std;
class Person;
class Car
{
public:
	weak_ptr<Person> m_person; //弱引用解决循环引用问题
	Car()
	{
		cout << "Car()" << endl;
	}
	~Car()
	{
		cout << "~ Car()" << endl;
	}
};
class Person
{
public:
	shared_ptr<Car> m_car;
	Person()
	{
		cout << "Person()" << endl;
	}
	~ Person()
	{
		cout << "~ Person()" << endl;
	}
};
int main()

{
	shared_ptr<Person> person(new Person());
	shared_ptr<Car> car(new Car()); 
	//两个强引用循环引用，导致对象无法销毁
	person->m_car = car;
	car->m_person = person;
	return 0;
}
/*
Person()
Car()
~ Person()
~ Car()
*/
```
#### `unique_ptr`

> `unique_ptr`也会对一个对象产生强引用，它可以确保同一时间只有一个指针指向对象
* 当`unique_ptr`销毁时(作用域结束时)，其指向的对象也就自动销毁了
* 可以使用`std::move`函数转移`unique_ptr`的所有权
```cpp
//ptr1强引用着Person对象
unique_ptr<Person> ptr1(new Person());
//转移之后，ptr2强引用着Person对象
unique_ptr<Person> ptr2 = std::move(ptr1);
```