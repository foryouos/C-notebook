# 面向对象三大特征
>* `封装`是指将数据和行为组合成一个整体，对外部`隐藏`内部的`实现细节，`只提供必要的`接口`。封装可以`保护`数据的`安全性`，`降低`代码的`复杂度`，`提高`代码的`可维护性`。C++通过`private,protected，public`关键字来`控制`成员变量和成员函数的`访问权限`
>* `继承`是指`子类可以继承父类`的属性和方法，并且可以`添加或修`改自己特有的属性和方法。`继承`可以提高`代码的复用性`；`提高`代码的`扩展性`；同时也是`多态的前提`
>* `多态`是指`不同类型的对象`对`同一消息`可以做出`不同的响应`。多态可以分为`编译时`和`运行时`多态。`编译`时多态是指通过`重载实现`的多态，即在同一类中定义了相同名称但不同参数的方法，根据调用时传递的参数不同而执行不同方法，`运行时多态`是指通过`重写实现的多态`，即在`子类中重新定义父类中已有的方法`，根据调用时`使用的对象不同`而`执行不同的方法`。多态可以实现`接口的同一`，增加`程序`的`灵活性`和可`扩展性`。

# C++类型转换

* `static_cast`：明确指出`类型转换`，没有动态类型检查，`上行转换`(派生类到基类)`安全`，`下行转换`(基类到派生类)`不安全`。
* `dynamic_cast`: 用于`有条件`的转换，`动态类型`检查，运行时检查类型安全(转换失败返回NULL)，只能用于`多态类型`的`指针或引用`
* `const_cast`：用于改变运算对象的`底层const属性`，`不能改`变其`顶层const属性`
* `reinterpret_cast`: 用于`无关类型之间`的转换，如`整型和指针，不同类型的指针`等。

# STL常见的容器

## 顺序容器

* `vector` : 可变`大小数组`，支持`快速随机访问`。在尾部之外的位置`增删元素可能很慢`
* `deque` ：双端队列。支持`快速随机访问`。在`尾部之外`的位置增删元素`可能很慢`
* `list` : `双向链表`。只支持`双向顺序访问`，在任何位置增删元素都能在`常数时间`完成。不支持随机存取
* `forward_list` ：`单向链表`，只支持单向顺序访问，在链表的任何位置增删元素都能在常数时间内完成，由于没有了size操作以及简化了增删元素的链节点操作，速度相比双向链表更快，不支持随机存取
* `strin`g :字符串。与vector相似，但专门用于保存字符。随机访问块，子尾部增删元素块
* `array`：`定长数组`。支持`快速随机访问`，不能添加和删除元素

## 关联容器

* `map` ： 关联容器。保存键值对 
* `set` ：关键字取值，即只`保存关键字`的容器,--`底层红黑树`
* `multimap` : 关键字`可`重复的map
* `multiset` ：关键字`可`重复出现的set
* `unordered_map` : 用`hash函数`组织的`map`
* `unordered_set` : 用`hash函数`组织的`set`
* `unordered_multimap` : 用`hash函数`组织的`map`，关键字`可重`复出现
* `unordered_multiset` : 用`hash函数`组织的`se`t，关键字`可重`复出现

# 简述vector实现原理

> `vector`是一种`动态数组`，在内存中具有连续的存储空间，支持`快速随机访问`。
>
> 由于具有连续的存储空间，所以在插入和删除操作方面，效率较低。当`vector`的大小和容器`相等`(`size == capacity`)，如果再向其添加元素，那么vector就需要`扩容`，vector容器扩容的过程需要经历三步
>
> * `完全摒弃`现有的内存空间，`重新申请`更大的内存空间
> * 将`旧内存空间`中的`数据`，按`原有顺序`移动到`新的内存空间`
> * 最后将`旧的内存空间释放`，`vector`扩容`非常耗时`，为了降低再次分配内存空间时的成本，每次`扩容时vector`都会申请比用户`需求量更多`的内存空间(这也就是vector容量的由来，即`capacity>=size`)，以便后期使用
>
> 不同的编译器在扩容时所采用的扩容因子可能不同，比如`MSVC`的`扩容因子`为`1.5`，即`每次扩容`时容量变为`原来的1.5倍`。

# `unordered_map`实现原理

> `unordered_map` 是一种`无序的关联容器`，它存储了`键值对`的集合，其中每个`键都是唯一`的
>
> `unordered_map` 的实现原理是基于`hash表`，通过把`关键码`映射到hash表中的一个位置来访问记录
>
> unordered_map 中的元素没有按照他们的键值或映射值的任何顺序排序，而是根据他们的`散列`值`组织成桶`允许它们的键值直接`快速访问`单个元素(通常平常平局时间复杂度)
>
> 当两个元素具有相同的`散列值`时，会发生`hash冲突`，为了解决这个问题，unordered_map采用了`链地址法`，即每个桶中存储一个链表，链表中存放所有散列值相同的元素。

# 简述map实现原理，各操作的时间复杂度

>* `map`是一个模板类，它的模版参数是键值对的类型和比较函数。比较函数用来定义键值对之间的大小关系，从而确认键值对在红黑树中的位置
>* map的底层数据结构也是红黑树，它与set的红黑树相同，只是每个节点存储的不是单个元素，而是一个`pair对象`，包含一个`key`和一个`value`
>* map的插入操作是先在红黑树中找到合适的位置，然后创建一个新节点，并将其颜色设为红色。如果新节点的父节点也是红色，那么就需要进行旋转和变色操作来回复平衡
>* map的删除操作是先在红黑树中找到要删除的节点，然后其后继或前屈替换它，并释放原来的节点。如果被删除或替换的节点是黑色，那么就需要进行旋转和变色操作来回复平衡。
>* map的查找操作是沿着二叉搜索树的路径向下查找，直到直到目标键值对或者未空为止
>
>由于红黑树保证了高度平衡，因此各操作的 时间复杂度均为`O(log n)`



## 简述map和unordered_map区别

> map基于红黑树实现，该结构具有中排序功能，因此map内部的所有元素都是有序的，红黑树的每一个节点都代表着map的一个元素。因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行这样的操作，红黑树的效率决定了map的效率，其增删改查时间复杂度O(log n)
>
> 而unordered_map内部实现了一个hash表，因此其元素的排列顺序是杂乱的，无序的。且增删改查时间复杂度为`O(1)`



# 迭代器遍历容器

>键盘输入 5 个整数，将这些数据保存到 vector 容器中，采用正向迭代器和反向迭代器分别遍历 vector 中的元素并输出。
>
>使用正向迭代器和反向迭代器分别遍历输出 vector 中的元素，元素之间使用空格隔开，两次遍历之间换行。
>
>例如：
>
>1 2 3 4 5
>
>5 4 3 2 1

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
// write your code here......

using namespace std;

int main() {

    // write your code here......
    vector<int> list;
    int a;
    while (cin >> a) //循环读取用户输入，当输入ctrl+c结束输入
    {
        list.push_back(a); //将元素添加到容器末尾
    }
    vector<int>::iterator iter = list.begin(); //获取起始的迭代迭代器
    while (iter != list.end())
    {
        cout << *iter << " " << endl;
        iter++;
    }
    cout << endl;
    while (iter != list.begin())
    {
        iter--;
        cout << *iter << " " << endl;
    }
    cout << endl;
    return 0;
}
```



# vector二维数组

```cpp
//定义2*3的二维数组，并初始化为零
// vector<vector<int>> a(row, vector<int>(col,0)); //初始化为0
vector<vector<int>> a(2,vector<int>(3,0));
//可以通过行列的形式赋值
a[0][0] =1;
//获取行数
int row = a.size();
//获取列数
int col = a[0].size(); //列数

```

# 深拷贝和浅拷贝

> * 浅拷贝就是将源对象的值拷贝给当前对象，两者指向的还是同一地址，对一个对象的修改可能会影响到另一个对象。

```cpp
class MyArray {
public:
    // 构造函数
    MyArray(int size) {
        this->size = size;
        data = new int[size];
    }
 
    // 拷贝构造函数，实现浅拷贝
    MyArray(const MyArray& other) {
        size = other.size;
        data = other.data; // 浅拷贝，将指针复制给新对象
    }
 
private:
    int size;
    int* data;
};
```

> * 深拷贝是 一种对象拷贝，它会创建一个新的对象，并将原始对象中的所有数据成员复制到新的对象中，包括多态分配内存。原始对象和新的对象是完全独立的，对一个对象的修改不会影响另一个对象。

```cpp
#include <iostream>
#include <cstring>
#pragma warning(disable : 4996)
using namespace std;

class Person {

    public:
        char* name; // 姓名
        int age;    // 年龄

        Person(const char* name, int age) {
            this->name = new char[strlen(name) + 1];
            strcpy(this->name, name);
            this->age = age;
        }

        // write your code here......
        Person(const Person& p)//拷贝构造函数
        {
            this->name = new char[strlen(p.name) + 1];
            strcpy(this->name, p.name);
            this->age = p.age;
        }
        
        void showPerson() {
            cout << name << " " << age << endl;
        }
        ~Person() {
            if (name != nullptr) {
                delete[] name;
                name = nullptr;
            }
        }

};

int main() {
    char name[100] = { 0 };
    int age;
    // 用户输入姓名和年龄
    cin >> name;
    cin >> age;
    // 
    Person p1(name, age);
   	// 实现浅拷贝
    Person p2 = p1;

    p2.showPerson();

    return 0;
}
```

>区别：
>
>* 对象中含有指针类型的成员变量时需要用深拷贝构造，否则用浅拷贝构造
>* 编译器默认的拷贝构造函数是浅拷贝构造函数
>* 如果对象中含有指针用了浅拷贝构造，那么会导致两个指针变量指向同一块地址空间，如果没有创建内存的操作就是浅拷贝，否则就是深拷贝
>* 深拷贝可以用于创建独立的副本，对于需要完全独立的对象的情况，必须在修改副本时不影响原始对象的状态。而浅拷贝通常用于创建共享对象，当需要多个共享相同的数据时，可以使用浅拷贝来减少内存占用和提高性能。

# 友元函数

> `友元函数`只是一个普通函数，并不是该类的类成员函数，它可以在`任何地方调用`，友元函数中通过`对象名`来`访问`该类的`私有或保护成员`。

```cpp
friend <类型> <友元函数名>(<参数表>);

#include <iostream>
 
using namespace std;
 
class A
{
public:
    A(int _a):a(_a){};
    friend int geta(A &ca);  ///< 友元函数
private:
    int a;
};
 
int geta(A &ca) 
{
    return ca.a;
}
 
int main()
{
    A a(3);    
    cout<<geta(a)<<endl;
 
    return 0;
}
```

# 友元类

> ```cpp
> friend class <友元类名>
> ```
>
> 类B是类A的友元，那么类B可以直接访问A的私有成员
>
> * 友元关系没有继承性
> * 友元关系没有传递性

```cpp
#include <iostream>
using namespace std;
class A
{
public:
    A(int _a):a(_a){};
    friend class B;
private:
    int a;
};
class B
{
public:
    int getb(A ca) {
        return  ca.a; 
    };
};
int main() 
{
    A a(3);
    B b;
    cout<<b.getb(a)<<endl;
    return 0;
}
```

