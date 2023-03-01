#### 面向对象

> 面向对象语言的`出发点`：更直接客观地描述客观世界中存在的事物（对象）以及它们之间的关系。

#### 面向对象的基本概念

* 对象：系统中用来描述客观事物的一个`实体`，它是用来构成系统的一个`基本单位`。对象是由一组属性和一组行为构成。属性是用来描述对象静态特征的数据项，行为时用来描述对象动态特征的操作序列
* 类：具有`相同属性和服务`的一组对象的集合

* 封装：把对象的属性和服务结合成一个`独立的系统单元`，并尽可能隐蔽对象的内部细节。
* 继承：特殊类的对象拥有其一般类的全部属性与服务，称作特殊类对一般类的继承。
* 多态性：指一般类中定义的`属性或行为`，被特殊类继承之后，可以具有不同的数据类型或表现出`不同的行为`。

#### 面向对象优点

> 使程序能够直接地反应问题域的本来面目，软件开发人员能够利用人类认识事物所采用的一般思维方法来进行软件开发。

#### 面向过程的结构化程序设计方法：

- 设计思路:`自顶向下，逐步求精`，采用`模块分解与功能抽象`，`自顶向下，分而治之`。

- 程序结构

- - 按功能划分为若干个基本模块
  - 各模块间的关系尽可能简单，功能上相对独立，每一模块内部均由顺序，选择，循环三种基本结构组成
  - 其模块化实现的具体方法是使用子程序

- 优点：有效地将一个较复杂的程序设计任务分解成许多易于控制和处理的子任务，便于开发和维护。

- 缺点：可重用性差，数据安全性差，难以开发大型软件和图形界面的应用软件。



#### 面向对象的方法

- 将数据及对数据的操作方法封装在一起，作为一个相互依存，不可分离的整体----`对象`

- 对同类型对象抽象出其`共性`，形成`类`

- 类通过一个简单的外部接口，与外界发生关系

- 对象与对象之间通过信息进行通信

- 优点：

- - 程序模块间的关系更为简单，程序模块的独立性，数据的安全性就有了良好的保障
  - 通过继承与多态性，可以大大提高程序的可重用性，使得软件的开发和维护都更为方便。
  - - - 静态特征：可以用某种数据来描述
      - 动态特征：对象所表现的行为或具有的功能。

#### 面向对象的软件工程

面向对象的软件工程是面向对象方法在软件工程领域的全面应用，

`包含`：

- 面向对象的分析（`OOA`）：抽象系统必须做什么
- 面向对象的设计（`OOD`）：概要设计，详细设计等
- 面向对象的编程（`OOP`）：
- 面向对象的测试（`OOT`）：
- 面向对象的软件维护（`OOSM`）等主要内容。

#### 基本术语

- 源程序：用源语言（C/C++/Java等）写的，有待翻译的程序
- 目标程序：也称为`结果程序`，是源程序通过翻译程序加工以后所生成的程序。：二进制代码
- 翻译程序:是指把一个源程序翻译成等价的目标程序的程序。



#### 三种不同类型的翻译程序

- 汇编程序：把汇编语言写成的源程序，翻译成机器语言形式的目标程序
- 编译程序：若源程序是用高级程序设计语言所写，经翻译程序加工生成目标程序，该翻译程序就称为“编译程序”。
- 解释程序：也是一种翻译程序，但与编译程序的不同是：`解释程序边翻译边执行，即输入一句，翻译一句，执行一句，直到将整个源程序翻译并执行完毕`.

#### 程序的开发过程：

- `编辑`：将源程序输入到计算机中，生成后缀为`cpp`的磁盘文件
- `编译`：将程序的源代码转换为机器语言代码
- `连接`：将多个源程序文件以及库中的某些文件连在一起，生成一个后缀为`exe`的可执行文件。
- `运行调试`

#### C++的特点：

- 尽量`兼容`C
- 支持面向对象的方法
- 保持C的简洁，高效和接近汇编语言等特点
- 支持泛型程序设计方法：使程序更加的通用。

#### C++程序示实例

```cpp
/*
* project: C++输出已经响铃实例
* environment：VS2022
*/
#include <iostream>  //预处理指令iostream中声明了程序所需要的输入和输出操作有关信息
using namespace std;  //命名空间
int main(void)  //main主函数，执行入口，int返回值类型
{
  //cout 输出流对象
  cout << "Hello!C++" << endl;
  cout << "Hello Word " <<endl;  
  //endl表示换行符

  printf("Hello word\n\a"); /*    \a响铃 */
  return 0;  //0意味着正常结束，非0返回，意味着程序异常结束
}
```



#### C++关键字：C++预先声明的单词

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyow5jy3R5YgKIK5a1TcGwvicClLYqH0ovjrQyfw7MXovvmDV4WdDCeUhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### C++的基本数据类型

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyoSuOywP1l0GRtvqDkZlbkGtf2lDpibibibCTKBYMeG6UQ8KzibhEGSFTTcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

C++的基本数据类型：

> `bool`（布尔型），`char`（字符型），`int`（整型），`float`（浮点型，表示实数），`double`（双精度浮点型，简称双精度型）。

#### C++预定义的转义序列

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyo7HEl6yjRAViaQZv937vjx2CRw6VAJibnW0KfAWwts67F9x6sHg1BBywg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 变量的存储类型

- `auto`存储类型:采用堆栈方式分配内存空间，属于暂时性存储，其存储空间可以被若干变量多次覆盖使用。
- `register`存储类型：存放在通用寄存器中
- `extern`存储类型：在所有函数和程序段中都可引用
- `static`存储类型：在内存中是以固定地址存放的，在整个程序运行期间都有效。

#### 符号常量

```cpp
const 数据类型说明符 常量名=常量值
//或者
数据类型说明符 const 常量名=常量值
```

> 符号常量在`声明时一定要赋初值`，而在程序中间不能改变其值，

#### 变量初始化

```cpp
int a = 0;
int a(0);
int a = {0};
int a{0};
```

其中使用大括号的初始化称为`列表初始化`，列表初始化时不允许信息的丢失。

####  `auto`类型与`decltype`类型

```cpp
auto val= val1+val2;
//让编译器通过初始值自动推断变量的类型
decltype(i)j=2;
//表示j以2位初始值，类型与i一致。
```

#### 位运算

- 按位与（&）：将两个操作数对应的每一位分别进行`逻辑与`操作

  

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyo7Y145AVbMFPLfhMWwkQ5icjUf5hnTyxqHxb67DusC0S6brOic4K9JeCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 按位或（|）：将两个操作数对应的每一位分别进行`逻辑或`操作

  

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyowLSQI8H2Elwx1hvkIeWoAWUgXgmj6GE3EfhPAIDcSOpUa6OlvdFjqg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



- 按位异或（^）：将两个操作数对应的`每一个位进行异或`：若对应位`相同`，则该位的运算结果为`0`，若对应位`不同`，则该位的运算结果为`1`.

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyoJyr6uJNadGJXRZDiaQtiaR8DkBdFibCrPM9Eic99epI1GO9iaMmzxvmUTXA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 按位取反（~）：一个单目运算符，是对一个`二进制数`的每`一位`取`反`

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyoiallBVgY31bjeicjMQPYHUyVMB1X6Yia1F4BgG42UCTzZkgyqicwtafliaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### C++类型转换

```cpp
类型说明符 （表达式）
（类型表达式） 表达式
const_cast <类型说明符>(表达式)
dynamic_cast <类型说明符>(表达式)
reinterpret_cast <类型说明符>(表达式)
static_cast <类型说明符>(表达式)
```



#### 数据的输入与输出

> 将数据从一个对象到另一个对象的流动抽象为`流`。流在使用前要被建立，使用后要背删。从流中获取数据的操作称为提取操作，向流中添加数据的操作称为插入操作。数据的输入与输出是通过I/O流来实现的，`cin`和`cout`是预定义的流类对象。`cin`用来处理标准输入，即键盘输入。`cout`用来处理标准输出，即屏幕输出。

```cpp
cout<< 表达式1<< 表达式2   
//输出
cin>>表达式1>>表达式2      
//输入，两数用空格隔开
```



#### 常用的I/O流类操作符

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvnKicrhvdQdibJrfD4FaYHyoz5E92AbY5Z7WibuSkR9FmCnVaWfljfCm4z94uNJ9KXnZ5cIY8eic1a4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
cout << setw(5) << setprecision(3) <<3.1415;
```

#### 算法控制的基本结构：

> 顺序结构，选择结构和循环结构`if ...else... while....for... `与C完全相同

#### 反编译

反编译：是将其语言代码转换为与之对应的汇编语言代码的过程。汇编语言与机器语言的指令具有一一对应的关系。

#### 函数

>  C++继承了C语言的全部语法，也包含函数的定义与使用方法。

```cpp

#include <iostream>
using namespace std;

double power(double x, int n)
{
  double val = 1.0;
  while (n--)
  {
    val = val * x;
  }
  return val;
}


int main(void)
{
  int n;
  cout << "Please input:";
  cin >> n ;
  cout << n << "的立方:" << power(n, 3) << endl;
  return 0;
}
```



#### 内联函数

`内联函数`不是在调用时发生控制转移，而是在编译时将函数体嵌入在每一个调用处，这样就节省了参数传递，控制转移等开销

```cpp
inline 类型说明符 函数名(含类型说明的形参表)
{
	语句序列
}
```

`注意`：

- 内联函数体内`不能`有循环语句和`switch语句`
- 内联函数的定义必须出现在内联函数`第一次`被调用之前
- 对内联函数`不`能进行异常接口声明



#### 函数重载

> 两个以上的函数，具有`相同`的函数名，但是形参个数或者类型不同，编译器根据实参和形参的类型及个数的最佳匹配，自动确定调用哪一个函数，这就是函数的重载。

* C++允许功能相近的函数在相同的作用域以相同函数名定义，从而形成重载，方便使用，方便记忆。
* 注意：重载函数的形参必须不同：个数不同或者类型不同,不以形参名或者返回值来区分。

#### 求整数和实数的平方和

```cpp
#include <iostream>
using namespace std;

double power(double x, int n)
{
  double val = 1.0;
  while (n--)
  {
    val = val * x;
  }
  return val;
}
int power(int x, int n)
{
  int val = 1;
  while (n--)
  {
    val = val * x;
  }
  return val;
}

int main(void)
{
  int n;
  double m;
  cout << "Please input:";
  cin >> n ;
  cout << n << "的立方:" << power(n, 3) << endl;
  cin >> m;
  cout << n << "的立方:" << power(m, 3) << endl;
  return 0;
}
```

####  `rand`函数

```cpp
int rand(void);  //函数原型
//所需头文件<cstdlib>
//功能和返回值：求出并返回一个伪随机数
```

#### `srand`函数

```cpp
void srand(unsigned int seed); //函数原型
//参数：seed产生随机数的种子
//所需头文件<cstdlib>
//功能：为使rand()产生一序列伪随机整数而设置起始点。
// 使用1作为seed参数，可以重新初始化rand()
```

#### 函数的参数传递

- 在函数被调用时才分配形参的存储单元
- 实参可以是常量，变量或表达式
- 实参类型必须与形参相符或可隐式转换为形参类型
- 值传递是传递参数值，即单向传递
- 引用传递可以实现双向传递
- 常引用作参数可以保障实参数据的安全

#### C++系统函数

```
#include <cmath>
```

#### 参考资料：

- 《C++语言程序设计》第四版郑莉
-  B站郑莉课堂[^1]

[^1]:https://space.bilibili.com/702528832/channel/collectiondetail?sid=35258&ctype=0