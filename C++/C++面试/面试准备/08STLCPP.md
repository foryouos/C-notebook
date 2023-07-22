# 面向对象三大特征

>* `封装`是指将数据和行为组合成一个整体，对外部`隐藏`内部的`实现细节，`只提供必要的`接口`。封装可以`保护`数据的`安全性`，`降低`代码的`复杂度`，`提高`代码的`可维护性`。C++通过`private,protected，public`关键字来`控制`成员变量和成员函数的`访问权限`
>* `继承`是指`子类可以继承父类`的属性和方法，并且可以`添加或修`改自己特有的属性和方法。`继承`可以提高`代码的复用性`；`提高`代码的`扩展性`；同时也是`多态的前提`
>* `多态`是指`不同类型的对象`对`同一消息`可以做出`不同的响应`。多态可以分为`编译时`和`运行时`多态。`编译`时多态是指通过`重载实现`的多态，即在同一类中定义了相同名称但不同参数的方法，根据调用时传递的参数不同而执行不同方法，`运行时多态`是指通过`重写实现的多态`，即在`子类中重新定义父类中已有的方法`，根据调用时`使用的对象不同`而`执行不同的方法`。多态可以实现`接口的同一`，增加`程序`的`灵活性`和可`扩展性`。

# C++类型转换

* `static_cast`：明确指出`类型转换`，没有动态类型检查，`上行转换`(派生类到基类)`安全`，`下行转换`(基类到派生类)`不安全`。
* `dynamic_cast`: 用于`有条件`的转换，`动态类型`检查，运行时检查类型安全(转换失败返回NULL)，只能用于`多态类型`的`指针或引用`
* `const_cast`：用于改变运算对象的`底层const属性`，`不能改`变其`顶层const属性`
* `reinterpret_cast`: 用于`无关类型之间`的转换，如`整型和指针，不同类型的指针`等。

# `STL`常见的容器

![STL对比](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbucZCeIYhyYp7UQbMMwAwkjicmDYTickhqsdXd6NTgRN144dKic0kYCqWIRz74mSoR3aibclGfZxJoKUw/640?wx_fmt=png "STL对比")

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
* `multiset` ：关键字`可`重复出现的set (上四个皆红黑树)
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
> `不`同的`编译器`在扩容时所采用的`扩容因子`可能不同，比如`MSVC`的`扩容因子`为`1.5`，即`每次扩容`时容量变为`原来的1.5倍`。

# `unordered_map`实现原理

> `unordered_map` 是一种`无序的关联容器`，它存储了`键值对`的集合，其中每个`键都是唯一`的
>
> `unordered_map` 的实现原理是基于`hash表`，通过把`关键码`映射到hash表中的一个位置来访问记录
>
> unordered_map 中的元素没有按照他们的键值或映射值的任何顺序排序，而是根据他们的`散列`值`组织成桶`允许它们的键值直接`快速访问`单个元素(通常平常平均时间复杂度)
>
> 当两个元素具有相同的`散列值`时，会发生`hash冲突`，为了解决这个问题，unordered_map采用了`链地址法`，即每个桶中存储一个链表，链表中存放所有散列值相同的元素。

# 简述map实现原理，各操作的时间复杂度

>* `map`是一个模板类，它的模版参数是键值对的类型和比较函数。比较函数用来定义键值对之间的大小关系，从而确认键值对在红黑树中的位置
>* map的底层数据结构也是红黑树，它与set的红黑树相同，只是每个节点存储的不是单个元素，而是一个`pair对象`，包含一个`key`和一个`value`
>* `map的插入`操作是先在红黑树中找到合适的位置，然后创建一个新节点，并将其颜色设为红色。如果新节点的父节点也是红色，那么就需要进行旋转和变色操作来回复平衡
>* map的删除操作是`先`在红黑树中`找`到要`删除的节点`，然后`其后继或前屈`替换它，`并释放`原来的节点。`如果`被删除或替换的节点是`黑色`，那么就`需`要`进行旋转`和`变色操作`来`恢复平衡`。
>* `map`的查找操作是`沿着二叉搜索树`的路径`向下查找`，直到直到目标键值对或者未空为止
>
>由于红黑树保证了`高度平衡`，因此各操作的 时间复杂度均为`O(log n)`



## 简述map和unordered_map区别

> map基于红黑树实现，该结构具有中排序功能，因此`map内部`的所有元素都是`有序`的，红黑树的每一个节点都代表着map的一个元素。因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行这样的操作，`红黑树的效率`决定了`map的效率`，其`增删改查`时间复杂度`O(log n)`
>
> 而unordered_map内部实现了一个`hash表`，因此其元素的排列顺序是`杂乱的`，`无序的`。且增删改查时间复杂度为`O(1)`

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

## 迭代器分类

> 根据`输出`的`不同`，使用`不同`的`迭代器`。

```cpp
// 正向迭代器
容器类名::iterator 迭代器名;  //依次向下遍历
// 反向迭代器
容器类名::reverse_iterator 迭代器名; //依次向上遍历
// 常量正向迭代器
容器类名::const_iterator 迭代器名;
// 常量反向迭代器
容器类名::const_reverse_iterator 迭代器名;
```

## 逆向迭代器

> 当使用逆向迭代器时，注意`逆向迭代器`的位置，如果输出`其值`，需要` -1`
>
> ![逆向迭代器](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbucZCeIYhyYp7UQbMMwAwkjuLM5NrbEP8ZeAUavfb0xkb58SBOXHfmh6sUVRUgoW6ibmibMTAw15utg/640?wx_fmt=png "逆向迭代器")

> 根据`输出`的`不同`，`选择`不同的`迭代器`

```cpp
/*
给出一个包含n个整数的数组a,使用vector实现倒序输出数组的最后k个元素。
*/
#include <iostream>
#include <vector>
using namespace std;
int main() {
	int n, k;
	vector<int>a;
	// write your code here......
	cin >> n >> k;
	int vel;
	for(int i = 0;i<n;i++)
	{
		cin >> vel;
		a.emplace_back(vel);
	}

	// cout << a[0] << endl;
	//使用逆向迭代器，一吃输出
	for (vector<int>::reverse_iterator iter = a.rbegin(); iter != a.rend(); iter++)
	{
		cout << *iter << " ";
		k--;
		if (k == 0)
		{
			break;
		}
	}
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

# 数组系列

> 支持`下标访问`

> `cbegin`在`begin`迭代器的`基础上`，`添加`了`const属性`，`不能`用于`修改`元素。

>* `vector`动态`可变`
>* 可`存储任何类型`数据
>* `连续存储空间`(扩容和中间插入效率低)
>* 大小`动态改`变，会被容器自动处理
>
>> 与`array`区别：
>>
>> * array `固定`大小，不能调整大小
>> * array编译时就已经分配好内存
>> * array适合存储大小已知并且`大小不会改变`的数据。

## 插入语法

```cpp
//1, 在迭代器pos指定的位置之前插入一个新元素elem,并返回新插入元素的迭代器
iterator insert(pos,elem);
//2, 在迭代器pos指定的位置之前插入n个元素elem，并返回表示第一个新插入元素位置的迭代器
iterator insert(pos,n,elem);
//3, 在迭代器pos指定的位置之前，插入其它容器(不仅限于vector)中位于[first,last]区域的所有元素
// 并返回表示第一个新插入元素位置的迭代器
iterator insert(pos,first,last);
//4, 在迭代器pos指定的位置之前，插入初始化列表(用大括号{}括起来的多个元素，中间有逗号隔开)中所有的元素，
// 并返回表示第一个新插入元素的迭代器
iterator insert(pos,initilist);
```

## emplace_back和push_back区别

* `push_back()` 向容器尾部添加元素时
  * `创建`元素
  * `拷贝/移动`到容器中
  * 事后`销毁第一步`创建的元素
* `emplace_back()`
  * `直接`在`容器尾部`创建元素
  * `省`去`拷贝和移动`的过程，在使用中`效率更高`

> * `c.emplace_back(t)` : 在`c`的`尾部`创建一个`值为t`的元素
> * `c.emplace_front(t)` : 在`t的头部`创建一个值为`t`的元素
> * `c.emplace(p,t)`  : 在`迭代器p`所指向的`元素之前`创建一个`值为t`的元素，`返回`指定`新添加元素`的`迭代器`。

## 使用

```cpp
#include <iostream>
#include <vector>
#include <set>
#include <algorithm>
using namespace std;
void print_container(const std::vector<int>& c)
{
	for (int i : c)
		std::cout << i << " ";
	std::cout << '\n';
}
int main(void)
{
	//1，创建动态数组
	vector<int> v;
	//2，添加元素
	//v.push_back(1); //在vector容器尾部添加元素
	v.emplace_back(2); // 在vector容器的尾部添加一个元素 效率更高
	// 2.1 预留元素空间
	v.reserve(100); //预留空间
	
	// 2.2 插入元素 -返回新插入位置的迭代器
	// insert(插入位置前面的迭代器;插入值;first,last插入范围，主要指其它容器;ilist，要插入来源的initializer_list
	//1，
	v.insert(v.end(), 5);
	//2,
	v.insert(v.cbegin(), 9, 5);
	//3,
	set<int> s = { 1,2,3,4,5 };
	v.insert(v.end(), s.begin(), s.end()); //在动态数组末尾，插入s的元素
	//4,
	v.insert(v.begin(), { 9,9,9,9 });
	// 3，删除元素
	print_container(v);
	//3.1 删除头节点
	v.erase(v.begin());
	//3.2 删除特定区间的节点
	v.erase(v.begin(), v.begin() + 2);
	print_container(v);
	//3.3 删除所有偶数
	for (vector<int>::iterator it = v.begin(); it != v.end();)
	{
		if (*it % 2 == 0)
		{
			it = v.erase(it);
		}
		else
		{
			++it;
		}
	}
	print_container(v);
	//3.4移除末尾元素
	v.pop_back();
	// 4,查考容量 遍历查找
	//  顺序遍历，逆序遍历
	for (const auto& el : v)
	{
		cout << el << " ";
	}
	cout << endl;

	// 返回最大值
	cout << v.size() << endl; //返回容纳的原数数
	// 返回容纳的元素数
	cout << v.max_size() << endl; //理论上可容纳的最大值(根据系统)
	cout << v.capacity() << endl; //返回当前存储空间可容纳的最大值

	// 相关操作
	// 指定值填充
	// 能够实现排序  --- 使用算法，左右迭代器实现算法排序
	sort(v.begin(), v.end());
	print_container(v);

	// 交换两个元素的所有内容
	vector<int> B{1, 3, 4, 5};
	v.swap(B);

	// 清空元素
	v.clear();
	//判断是否为空
	if (v.empty())
	{
		cout << "vector为空" << endl;
	}
	// 通过释放未使用的内存较少空间
	v.shrink_to_fit();
	return 0;
}
```

# 双端队列`deque`

> * 可以进行`下标访问`的`顺序容器`
> * 允许在它的`首尾`两端`快速`插入及删除
> * 与`vector相反`，`不`一定`相邻存储`。
> * 时间复杂度 :  `随`机访问 `O(1)`，在`结尾`或`起始`插入或移除元素 -- 常数`O(1)`，`插`入或`移除`元素 --线性 `O(n)`
> * 

```cpp
/*
请设计一个排队程序，用户有普通客人和 VIP 客人之分，VIP 客人不排队（即 VIP 客人在队列头部），请将已有的guest1和guest2放入队列中（guest1排在guest2前），并将VIP客人新增至队列头部。
*/

#include <iostream>
#include <deque>
using namespace std;

class Guest {
public:
    string name;
    bool vip;
    Guest(string name, bool vip) {
        this->name = name;
        this->vip = vip;
    }
};
void in_queue(deque<Guest>& deque,Guest guest) // 引用地址才能改变deque
{
    if(guest.vip)
    {
        //vip插入队列头部
        deque.emplace_front(guest);
    }
    else
    {
        //普通用户
        //普通客户插入到队列尾部
        deque.emplace_back(guest);
    }
}
int main() {

    Guest guest1("张三", false);
    Guest guest2("李四", false);
    Guest vipGuest("王五", true);
    deque<Guest> deque; // 双端队列队列

    // write your code here......
    in_queue(deque,guest1);
    in_queue(deque,guest2);
    in_queue(deque,vipGuest);
    //除数队列排序
    for (Guest g : deque) {
        cout << g.name << " ";
    }

    return 0;
}
```

# `set`系列

> 特点
>
> * `默`认`升序`排序(`可降序`)
> * 内部使用`红黑树`
> * `不含`重复元素(自动`去重`)
> * 不能下标访问。

```cpp
#include <iostream>
#include <set>
using namespace std;
int main(void)
{
	//1,创建set
	set<int> st; //默认升序相当于set<int,less<int> > s;
	//set<int, greater<int>> st; //设置降序
	//2,插入数据 并自动递增排序且去重
	st.insert(3);
	st.insert(2);
	st.insert(5);
	st.insert(1);
	st.insert(6);
	st.insert(7);
	st.insert(9);
	st.insert(71);
	st.insert(17);
	st.insert(55);
	st.insert(5); //重复的元素将会被省略

	//3，查找元素 find,返回set对应值为value的迭代器
	set<int>::iterator it = st.find(3);
	cout << "返回查询的值: " << *it << endl;
	//4,删除元素
	//使用erase可以删除单个元素，也可以删除一个区间内的所有数据
	st.erase(7); //删除7元素
	it = st.find(6);
	st.erase(it, st.end()); //删除6之后的所有元素

	//5,返回元素个数O(1)
	cout << "元素个数为:" << st.size() << endl;

	//6，输出
	for (it = st.begin(); it != st.end(); it++)
	{
		cout << *it << endl;
	}
	// 7 清空所有元素O(N)
	st.clear();
	//  8 返回set是否是空
	if (st.empty())
	{
		cout << "空" << endl;
	}
	return 0;
}
```

> * `unordered_set `： 使用``散列``取代`红黑树`，实现`只去重不排序`。速度`快于set`
> * `multise`t : `不去重`，但`排序`
> * `unordered_multiset` : `不排序`，`不去重`

# map

> map存储的pair对象，也就是用pair类模版创建的键值对。
>
> 默认根据`键`来进行升序排序。并`去重`
>
> `map`存储的都是`pair类型`的`键值对`元素
>
> `支`持`下标`访问

```cpp
#include <iostream>
#include <map>
#include <algorithm>
#include <string>
using namespace std;
//输出map
auto print = [](auto const comment, auto const& map)
{
	cout << comment << "{";
	for (const auto& pair : map)
	{
		//first键，second对应的值，map默认根据键进行升序排序
		cout << "{" << pair.first << ":" << pair.second << "}";
	}
	cout << endl;
};
int main(void)
{
	//默认为升序
	// map<int, string,less<int>> m;
	// map<int, string> m;
	// 将升序设置为降序
	map<int, string, greater<int>> m;
	cout << m[10] << endl;
	//添加
	m[10] = "张三";
	m[6] = "李四";
	m[19] = "麻花";
	m[3] = "王五";
	m[3] = "王五";
	print("键值对:",m);
	// 插入键值对
	m.insert(pair<int, string>(25, "刘强"));
	m.insert(make_pair(29, "王宝强"));
	//当键值对成功插入到对应的map容器中，其返回的迭代器指向该新插入的键值对，
	//插入失败是，表明map容器中存在相同的键值对，此时返回的迭代器指向具有相同键的键值对，
	// 同时bool变量的值为false
	pair<map<int, string>::iterator, bool> ret;

	ret = m.emplace(9, "清华"); 
	cout << "1." << ret.first->first << " " << ret.first->second << ", " << ret.second << endl;
	print("键值对:", m);

	// 使用emplace_hint()
	// 该方法不仅要创建键值对所需要的数据，
	// 还需要传入一个迭代器作为参数,指明要插入的位置
	// 新键值对会插入到该迭代器指向的键值对的前面
	// 该方法的返回值是一个迭代器，不再是pair对象，
	// 当成功插入新键值对时，返回迭代器指向新插入的键值对
	// 反之若插入失败，则表明map容器中有相同键的键值对，
	// 返回的迭代器就指向这个键值对
	map<int, string>::iterator iters;
	iters = m.emplace_hint(m.begin(), 15, "北大");
	cout << iters->first << " : " << iters->second << endl;

	// 删除同样使用key值
	m.erase(10);
	// 使用迭代器删除 ,使用查找，返回主键对应的迭代器
	map<int, string>::iterator iter = m.find(6);
	m.erase(iter);  
	//同样可以使用迭代器的形式，删除某一个区间范围内的值
	print("删除后:", m);

	//查找
	// lower_bound(key) 返回指向第一个键大于或等于key的键值对的迭代器
	iter = m.lower_bound(30);
	cout << "lower_bound: " <<iter->first << iter->second << endl;
	// upper_bound(key) 返回的是指向第一个键大于key的键值对的迭代器
	iter = m.upper_bound(5);
	cout << "upper_bound: " << iter->first << iter->second << endl;
	// 返回范围的键值对equal_range
	// 创建一个pair对象，来接受equal_range()的返回值
	pair<map<int, string>::iterator, map<int, string>::iterator> mypair;
	mypair = m.equal_range(19);
	for (auto iter = mypair.first; iter != mypair.second; iter++)
	{
		cout << iter->first << " " << iter->second << endl;
	}
	//返回当前容器可以容纳的最大元素个数
	cout << "容器的最大存储量:(与机器有关)"<< m.max_size() << endl;
	cout << "容器中键值对的个数:" << m.size() << endl;
	cout << "元素个数: " << m.count(29) << endl; //map不允许重复值，为1
	//清空map
	m.clear();
	//判断当前map容器是否为空
	cout << m.empty() << endl;
	return 0;
}
```

# list

> 双向带头循环链表 
>
> * 允许在`常数范围内`的`任意位置`进行`插入和删除`  --`不支持随机下标`访问
> * `链表`是插入元素，`不需要`提前`扩容`，`没有reserve操作`，`可`以使用`empty`判空，使用size返回大小
> * 由于其双向迭代器的特征，list只能使用list提供的sort排序即：`list.sort()`.`链表`排序`效率较低`，`排序`尽量`使用vector`不要使用list
>
> ![list](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbucZCeIYhyYp7UQbMMwAwkjQ3rusOhCrSr81CEicyRfw6rNGhaicylJRmSOzcr7rohBowk9XGyeOobw/640?wx_fmt=png)

##  forward_list

> 支持从`容器`中的`任何位置`快速`插入和移除`元素的容器，`不`支持`快速随机`访问，实现方式为`单链表`。与list相比，此容器不需要双向迭代时提供`更有效地利用空间`的存储。

```cpp
#include <iostream>
#include <list>
#include <algorithm>
#include <string>
using namespace std;
void print(list<int> l)
{
	cout << " l = { ";
	for (int n : l)
	{
		cout << n << ", ";
	}
	cout << "};" << endl;
}
int main(void)
{
	//双向循环列表初始化
	list<int> l = {7,5,16,8};
	
	// 添加元素
	// 添加元素到list开头
	l.push_front(25);
	// 添加元素到list结尾
	l.push_back(13);
	// 数据插入
	// 查找插入位置之前的迭代器
	auto it = find(l.begin(), l.end(), 16); //返回16前一位的迭代器
	// 插入数据
	if (it != l.end())
	{
		l.insert(it, 42);
	}
	// 遍历打印list的值
	print(l);
	// 只能使用list提供的方法进行排序
	l.sort();
	print(l);
	// 将元素逆置
	l.reverse();
	print(l);
	// 交换两个容器的内容
	list<int> li = { 1,2,3,4,5 };
	l.swap(li);
	print(l);
	print(li);
	//重新分配内容
	// assign函数用于将新内容分配给容器，替换其当前的内容
	// 1,将n个值为val的数据分配给容器
	// 2,将所给迭代器区间当中的内容分配给容器
	l.assign(li.begin(), li.end());
	print(l);
	// 移除元素
	l.remove(1); // 移除所有等于1的元素
	l.remove_if([](int n) { return n > 25; }); //移除全部大于10的元素
	print(l);
	// 使用erase移除
	// 1,要移除元素的迭代器
	// 要移除的元素迭代器范围
	l.erase(l.begin());
	return 0;
}
```

# stack

>   ` FILO`（`先进后出`）`数据结构`

```cpp
#include <iostream>
#include <stack>
#include <algorithm>
#include <string>
using namespace std;
int main(void)
{
	// 初始化先进后出
	stack<int> s;
	// 插入元素
	s.emplace(1); //先栈顶插入元素
	for (int i = 0; i < 9; i++)
	{
		s.emplace(i);
	}
	s.push(2);    //上同
	s.pop();      // 删除栈顶元素
	cout << s.size() << endl; //返回栈的大小
	//访问栈顶元素
	cout << s.top() << endl;

	return 0;
}
```

# queue

> 队列 ： `先进先出`的`数据结构`.
>
> queue在底层容器`尾端推入`元素，在`首端弹出`元素。

```cpp
#include <iostream>
#include <queue>
#include <algorithm>
using namespace std;
int main(void)
{
	queue<int> q;
	//1，在尾部构造元素
	q.push(9);
	for (int i = 0; i < 6; i++)
	{
		q.emplace(i);
	}
	//2,访第一个元素 //队头
	cout << q.front() <<endl;
	//3，访问最后一个元素 // 队尾
	cout << q.back() << endl;

	// 4,删除首个元素
	cout << q.size() << endl; //返回当前元素
	q.pop(); //删除了0
	// 5,判断当前容器是否为空
	
	// 交换内容
	queue<int> qu;
	q.swap(qu); //q和空的qu队列交换
	if (q.empty())
	{
		cout << "空" << endl;
	}
	return 0;

}
```

# 联系

## 统计字符串中各字母字符对应的个数

> 键盘输入一个字符串，统计字符串中各个字母字符的个数。例如：键盘输入"Hello World!"，上述字符串中各个字母字符的出现的次数为：
>
> H:1
>
> e:1
> l:3
> o:2
> W:1
> r:1
>
> d:1
>
> 要求使用map实现，键的排序使用map默认排序即可。

```cpp
#include <cctype>
#include <cstring>
#include <iostream>
#include <map>
// write your code here......
using namespace std;
int main() {

    char str[100] = { 0 };
    cin.getline(str, sizeof(str));

    // write your code here......
    //输入字符串，根据字符依次加入map，主键是char,值为个数
    map<char, int> m;
    // 遍历一个字符串
    for (int i = 0; i < strlen(str); i++) {
        // if (isalpha(str[i])) {
        //     if ( m.count(str[i])) { //是字符并且存在
        //           m[str[i]] = m[str[i]]+1;//若没有查到返回
        //         //判断对应的字符是否存在，重复会被替换
        //     } else {
        //         m[str[i]] = 1;
        //     }
        // }
        if (isalpha(str[i])) 
        {
            m[str[i]] ++; //若没有查找，会插入，找到之后将对应的值++
        }
        
    }
    // 输出
    for (const auto& pair : m) {
        cout << pair.first << ":" << pair.second << endl;
    }

    return 0;
}
```

## 查找

> 给出一个大小为*n*的数组*a*，有m*次询问，每次询问给出一个x*，你需要输出数组a*中大于x*的最小值，如果不存在，输出-1。
>
> 要求使用set实现。

```cpp
#include<bits/stdc++.h>
#include <set>
using namespace std;
int main(){
	int n,m; //set数组大小为n,m
	cin>>n>>m;
	set<int>s;
	int q;
	int i;
	for(i = 0;i<n;i++)
	{
		cin>>q;
		s.insert(q);
	}
	//所有数据已经插入到集合中
	//进行m次询问
	for(i=0;i<m;i++)
	{
		cin>>q;
		//输出大于x的最小值 upper_bound 返回指向首个大于给定键的元素的迭代器
		set<int>::iterator it = s.upper_bound(q); 
		// 如果不存在，返回end迭代器，end是空读取不到的
		if(it == s.end())
		{
			cout<<-1<<endl;
		}
		else {
			cout<<*it<<endl;
		}
		
	}
	return 0;
}
```

## 对vector排序

> 键盘输入 5 个整数，使用 vector 进行存储，使用 STL 排序算法对元素进行排序（从大到小），再使用 STL 遍历算法输出元素。（元素和元素之间使用空格隔开）

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <vector>
// write your code here......

using namespace std;

int main() {

    int num;
    vector<int> v;
    for (int i = 0; i < 5; i++) {
        cin >> num;
        v.push_back(num);
    }

    // write your code here......
    //默认是升序，使用greater<int>()调整为降序
    sort(v.begin(), v.end(),greater<int>());
    for(const auto &e:v)
    {
        cout<<e<< " ";
    }
    cout<<endl;
    return 0;
}
```

# 资料

> 公众号：瓶子的跋涉  
>
> * 在对话框输入`STL`
> * 即可获得`STL范例大全`，看看学习，用法。
