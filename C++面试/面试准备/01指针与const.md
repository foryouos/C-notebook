# 特别

> `while`一般而言，所有`非零值`都视为`真`，只有`0`被视为`假`。

```cpp
#include <iostream>
using namespace std;
int main(void)
{
	printf("%d\n", 1);
	int i = -6;
	// while 一般而言，所有非零值都视为真，只有0被视为假
	while (i) 
	{
		printf("%d\n", i);
		i++;
	}
	return 0;
}
```

# 指针与`const`

> 指针在初始化时，引用了，其它变量的值，无论对指针的地址还是对指针值`+ const`。修改变量的值，都将影响到指针
* 对`*`后面加入const 代表将地址设置为常量，`地址不可变`，但是`对应的值`可以改变。
* 第`*`前面的值加const，对应的值`*p`不能改变，但是可以`改变p`的地址，通过此形式改变了`*p`对应的值。
* 对`*`的前后`都`添加了`const`，代表`对应的地址`，和`值都不能改变`，但是如果在`const初始化`时是`引用其它变量`的值，就可以修改`其它变量的值`，来`间接修改此值`。

```cpp
/*
	关于const与指针之间的运用问题
*/
#include <iostream>
using namespace std;
int main(void)
{
	int a = 6;
	int i = 1;
	// *p常量化
	const int* p = &a;
	cout << "a: " << *p <<endl;		//6
	p = &i;
	cout << "p = &i : " << *p << endl;		//1
	//*p = 10;  //*p为const值不可改变
	cout << "*p: " << *p << endl;   // 
	// 对a的值进行修改，间接的修改*p的值
	a = 7;
	// 通过对a值的修改，间接修改了 指向p地址的p
	cout << "*p: " << *p << endl;   // 7
	int b = 8;
	//*p固定值不能改变，但是p的地址可以改变
	p = &b;  //通过赋值新的地址p
	cout << "*p: " << *p << endl;
	
	// t地址常量
	int* const T = &a;   
	cout << "const p: " << *T << endl; //值为7
	// T = &b;  //地址为cosnt常量，不可进行修改
	//当忍让可以通过修改a的值，间接修改引了a地址的T
	a = 9;
	cout << "const p: " << *T << endl;

	// 地址和值都是常量
	const int* const S = &a; //初始化为a的9值
	// S = &b;  //地址值不再允许改变
	// 通过a的值可以 改变S的值
	
	//
	a = 10; //a仍然为变量，仍然可以进行改变

	cout << "const int* const S ：" << *S << endl;

	//对*S以及S的地址，在初始化后不可再次改变

	return 0;
}
```
# 数组a[]中a与&a区别

* `值相同`
* `a`指的是`a[0]`的`地址`
* `&a`指的是`数组a`的`地址`
* 数组名代表数组第一个元素的地址，`&数组名`代表`整个数组`的地址，从而导致`a+1`和`&a+1有本质`的区别

# char*  与char a[]
> `char *a = "abcd" ` 
> `char a[20] = "abcd" `

* 读写能力
> 字符串数据存放在`常量存储区`，通过指针只可以访问字符串常量，而不可以改变它
> 数组数据存放在`栈`，可以通过`指针`去`访问`和`修改`数组内容

* 赋值时刻
> 指针编译时就已经确认了，因为是`常量`
> 数组运行时确认

* 存取效率
> 存于`常量存储区`，在`栈上的数组`比`指针所指向字符串`快
> `数组`存放在`栈上`，因此`块`

> `strlen` 不计`\0`,但是`sizeof`计算字符串容量时算`\0`

# `读取位置`:sailboat:

```cpp
// 不加*代表真个字符串地址，加*对应地址的单个 字符
const char* a = "helloworld"; //常量字符串
//在没有取单独值时，就是指针移动，指针移动的是对应的字符对应的字节
cout << a + 1 << endl;        //起始地址右移，变为elloworld
char b[] = "morning";  
cout << b+1 << endl;          //同样地址右移，变为orning
//其相等，取地址位置的一个字符
cout << *(a + 1) << endl;   //取固定值
cout << b[1] << endl;;
printf("%c\n", *(a + 1));
printf("%c\n", b[1]);
```
# 为`char*`申请`固定空间`

> `char*`代表`字符串指针`，`单个`，为其`申请空间`，要取其`字符串指针的地址`即`二级指针`进行`空间申请`。

```cpp
void getMemory(char** n)
{
	*n = (char*)malloc(100);
}
char* n = NULL;
getMemory(&n);
strcpy(n, "nihaohaohao");
cout << n << endl;
```

# 整型在内存中的存储及运算规则

> 有符号`二进制`表示`正反`的方式，在`首位`
>
> * `1为负数`
> * `0为正数`
>
> 数据以`补码`的形式保存在内存中。
>
> `正数`的`反码`和`补码`是`其本身`
>
> > 在类型转换的过程中直接`保存低位`即可
> >
> > 在`int转char型`，-2对应的补码`1111 1110`转为4位，直接`截断`得到四位，即1110，这个数对应的还`是-2`；
> >
> > 在还原时：-1，再求反码即得原码得到原数
>
> > `char转int`，若`高位是0`，`正数`，`高位补0`，若为 `负数`高位`补1`即可

![正负保存形式](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkk1W9XmyXrKAVXE4cEoTzulMtu2xiauDBxwyk6Onh6Ytc88riaZMVavJFA/640?wx_fmt=png)



## 截断

> 当将不同类型元素混合赋值且指向内存空间大小不一样时，就会`发生截断`。截断会`高位截断`，保留低位数据，当`-1`整型`存入字符串数据中`，就会`发生截断`

## 整型提升

> `整型提升`的`意义`在于：表达式的整型运算要在CPU的相应运算器件内执行，CPU内整型运算器(ALU)的操作数的字节长度一般就是int的字节长度，同时也是CPU的通用寄存器的长度。因此，即使两个char类型的相加，在CPU执行时实际上也要先转换为CPU内整型操作数的标准长度。通用CPU（general-purpose CPU）是难以直接实现两个8比特字节直接相加运算（虽然机器指令中可能有这种字节相加指令）。所以，表达式中各种长度可能小于int长度的整型值，都必须先转换为int或unsigned int，然后才能送入CPU去执行运算。

```c
int main()
{
	char a = -1;  //-1截断后存储到a中
	//10000000000000000000000000000001	-1的原码
	//11111111111111111111111111111110	-1的反码
	//11111111111111111111111111111111  -1的补码
	//11111111 - a   截断后char a中a所存的补码
	
	signed char b = -1;
	//11111111111111111111111111111111  -1的补码
	//11111111 - b    b的补码
	//

	unsigned char c = -1;
	//11111111111111111111111111111111  -1的补码
	//11111111 - c    c的补码。
	//
	//整型提升
	printf("a=%d,b=%d,c=%d", a, b, c);
	//-1 -1 
	//11111111111111111111111111111111
	//11111111111111111111111111111110
	//10000000000000000000000000000001

	//11111111
	//00000000000000000000000011111111

	return 0;
}
```

# 八道笔试题集合

> 指针层级`过深时`，通过`画关系`的`形式表达`出来。

## `sizeof小练`

> 牢记：sizeof的计算在编译时刻，把它当`常量表达式`使用，且会`忽略掉表达式`内部的`各种运算`，指针在`32位系统`中占`4`个字节，在`64位系统`中占`8`个字节

* `sizeof`计算字符串容量时算`\0` 与此同时: `sizeof("\0")=2`;
* 在`编译阶段处理`，`sizeof`作用范围内的内容`不能被编译`，所以`sizeof()`内的`运算不被执行`
* `sizeof(函数) `= `sizeof(返回值类型)`
* 联合体：最长成员的大小对齐
* `sizeof(2.5+3.14)`实际上是 `sizeof(double)`  切记需要识别出`其类型`
* `sizeof`可以对`函数调用`求值，实际上是对`返回值类型求值`
```cpp
int func(char s[5])
{
	cout << endl;
	return 1;
}
sizeof(func("1234"));  //得出返回值类型int的大小，4个字节
```
### sizeof不可用
* `不能对函数名`求值
* `不`能第`不确定返回值`的类型求值，如`void`

```cpp
#include <stdio.h>
int main(void)
{
	short num = 20;
	int a = 1;
	printf("%d\n", sizeof(num = a+5));
	printf("%d\n", num);
	return 0;
}
```



## 笔试1

```cpp
#include<stdio.h>
int main()
{
    int a[5] = { 1, 2, 3, 4, 5 };
 	//a为数组a[0]的地址与数组a地址重合，&a取的就是数组a的地址
    int* ptr = (int*)(&a + 1);    // 取了数组a之后的地址空间
    printf("%d,%d", *(a + 1), *(ptr - 1)); // a的地址空间减去一个sizeof(int) 得出结果: 2,5
    return 0;
}
```
> `指针的类型`决定了`指针+1时`的`步长`，指针的类型决定了对指针进行`解引用操作`时，`访问的空间大小`

![数组指针引用](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkvpbC0pQRyicJHfR1RnWofhwtXibPiabZfWIlDwZ9b2YGItZjEK2d4sXaA/640?wx_fmt=png "数组指针引用")



## 笔试2

> 指针移动，移动对应类型的`sizeof(类型)` 的字节数，注意转换关系，要根据`转换关系`灵活`应用`
```cpp
#include<stdio.h>
//此结构体的大小是20个字节
struct Test
{
	int Num;
	char* pcName;
	short sDate;
	char cha[2];
	short sBa[4];
}*p;
 
int main()
{
	p = (struct Test*)0x100000;//假设p的值为0x100000。  //p的地址0x0000000000100000
	//%p输出地址
	//p为结构体类型，每次指针移动一位，就是移动的strct Test的大小，和数组移动时一样
	// 移动了一位，就是内存地址移动了20个字节
	printf("%p\n", p + 0x1);   //0x100020
	// 使用了类型转换,无符号长整型 对应的是值 +1
	printf("%p\n", (unsigned long)p + 0x1); //0x100001
	// 使用无符号整型，那整型+1就是int,对应的是指针类型，要加4个字节就是4个字节
	printf("%p\n", (unsigned int*)p + 0x1);  //0x100004
	return 0;
}
```
![结构体添加](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkBN1ricluEwq82P2z8BYgoSP0R7EnMCok9o4icRmuwNaPjPQcH1JRjicSg/640?wx_fmt=png "第一问")

![第二问](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkhygpVw5NuAU6blDLSfAv4WAibzOPdVhfY28RdWQ2rCibe9rk4vw6Wicicw/640?wx_fmt=png "添加长整型")

![第三问](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkiba0AaiaTEJVFzqnCP4xosaJ7phic6j89b6g6ASO6bdtEmAueZyO5nCYg/640?wx_fmt=png "添加整型")



## `笔试3`

```cpp
#include<stdio.h>
int main()
{
    int a[4] = { 1, 2, 3, 4 };
    int* ptr1 = (int*)(&a + 1); //上通4 &a取的是整个数组a的地址，+1即跳过一个int(*)[4]类型，然后再将它强制转换为int *类型
    int* ptr2 = (int*)((int)a + 1);
    // 
    printf("%x,%x", ptr1[-1], *ptr2); //%x以16进制打印
    return 0; 
}
```
![ptr2的地址](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkF4tgMeHUic6TzcogYlb5ek40yYvFNFwsz2TE9eTPXVia1BNgVS1gzG6g/640?wx_fmt=png "ptr2地址")

![第一问](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkQkEN0uxWv96c6g9r3EdiaiagADiaIrq88xUC7lAJeA3xia8Ww3L13Vufmg/640?wx_fmt=png)

![第二问](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkR3kVc5r0icJTEySUFTdMUjC7ytUZZuM9FRTN88DibRTXbez1hoHSiaicmg/640?wx_fmt=png)

![第三问](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkNua7DkHKqhxDvePJSA25202SX1U78wiaKM3mV9eRscyILnSIiaWwx0Sg/640?wx_fmt=png)

## 笔试4

### 逗号表达式

* 逗号表达式的运算过程为：从左往右逐个计算表达式
* 逗号表达式作为一个整体，它的值为最后一个表达式(也即表达式n)的值
* 逗号运算符的优先级别在所有运算符中最低

> ```cpp
> main()
> {
>     int x,y,z;
>     x=1;
>     y=1;
>     //由于后面没有括号，且逗号的优先级是最低的，z就等于x++了，后面照样运行
>     z=x++,y++,++y;
>     printf("%d,%d,%d",x,y,x);
> }
> ```

```cpp
#include<stdio.h>
int main()
{
    // (0, 1) 逗号运算符，取最后一个
    // { {1,3},{5,0},{0,0} }
    int a[3][2] = { (0, 1), (2, 3), (4, 5) };
    int* p;
    p = a[0];
    printf("%d", p[0]); //1
    return 0;
}
```
## `笔试5`

```cpp
#include<stdio.h>
int main()
{
    int a[5][5];
    // p是一个数组指针，指向一个有4个整型元素的数组
    int(*p)[4];
    // a是数组名，数组名代表的是数组首元素的地址，二维数组的首元素是第一行元素，
    p = a;
    printf("%p,%d\n", &p[4][2] - &a[4][2], &p[4][2] - &a[4][2]);
    return 0;
}
```
![指针赋值](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkWC50jHSOYchS28sKDRick2T8S1gjwAzjj0rABgCudEg9Z0FAw7g4HLA/640?wx_fmt=png "指针赋值详细")

![详解](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkk5ZwZ1xOIGhXhjsh0X9Jg5ibyIuxUYeGkRfajQcgD4BmuxtKvpj8iaRUQ/640?wx_fmt=png "详解")

## 笔试6

```cpp
#include<stdio.h>

int main()
{
    int aa[2][5] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    int* ptr1 = (int*)(&aa + 1); //取了数组aa之后的内存
    int* ptr2 = (int*)(*(aa + 1)); // 取的是aa[1][0]的地址
    printf("%d,%d", *(ptr1 - 1), *(ptr2 - 1)); //10，5
    return 0;
}

```
![第六题](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkDopUnK4qcuy3jxLQxHYf9LibnBGRfEEePyeDo8lxMN1mpgqYkdgcYJw/640?wx_fmt=png "6解法")

## 笔试7

```cpp
#include<stdio.h>

int main()
{
	// 代表多个字符串a[0],a[1],a[2]   char变成了字符串
	const char* a[] = { "work","at","alibaba" };
	// a的地址默认是a[0]的地址，
	const char** pa = a;
	pa++;   //a[1]
	printf("%s\n", *pa);  //at
	return 0;
}
```
![07解答](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkibtOByQiagGLQxl7De0tYlbaiatcdIB4BuNVO2XNvwC8LbUxLackAlMBg/640?wx_fmt=png "07题")



## 笔试8

```cpp
#include <stdio.h>
int main()
{
	char* c[] = { "ENTER","NEW","POINT","FIRST" };
	char** cp[] = { c + 3,c + 2,c + 1,c };
	char*** cpp = cp;
	printf("%s\n", **++cpp);
	printf("%s\n", *-- * ++cpp + 3);
	printf("%s\n", *cpp[-2] + 3);
	printf("%s\n", cpp[-1][-1] + 1);
	return 0;
}
```

![第一步](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkIyNicOXzib6kMjIqWDEUU5Io8D2EIq0mmVVJrHKoHLZIyjcfSyRLMK9A/640?wx_fmt=png)

![第二步](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkX4giay6zIDHkspFkGqDXeOADgh3kSp62ibiaHDqzIf4RPSsmF1U8Rs6XQ/640?wx_fmt=png "第二步")

![第三步](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkUQdqtE1qKibf6cotwuC0mnGQJg29r3NNCJxanWPibWZ66st7QuBaQ8mQ/640?wx_fmt=png)

![第四步](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkuwRyicxDJnoqwBDo1HyMPH5NFF4URR1XOsQrEz1NHaX30UMkKib7h9EA/640?wx_fmt=png )

![第五步](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtWag4RMZqqBh24Iv7t2wkkPCozmcwR1rQ3eIZ7FibpBywSNcZe2XdqQ2peicXj1GpuQ5WzTcCzHrcw/640?wx_fmt=png)

# 参考资料

* [关于指针的笔试题](https://blog.csdn.net/m0_62692838/article/details/127338454)

* [`sizeof`详解](https://blog.csdn.net/qq_32693119/article/details/86617654)
* [截断](https://blog.csdn.net/m0_64579278/article/details/127418670)

* [入坑无符号类型转换](https://blog.51cto.com/u_13682052/2980047)