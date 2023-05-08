https://i.imgtg.com/2023/04/24/IeepG.jpg

# 静态库
> 对源文件进行汇编操作`(使用参数-c)`得到二进制的目标文件`(.o 格式)`,然后通过`ar工具`将目标文件打包就可以得到`静态库文件``(libxxxx.a)`

## `ar`参数

* 参数c:创建一个库，不管库是否存在
* 参数s:创建目标文件索引，在创建较大库时加快时间
* 参数r:在库中插入模块(替代)。默认新的成员添加在库的结尾处，若该模块已经存在，则替换同名模块

## 生成静态链接库
* 对源文件进行汇编，得到`.o`文件
```sh
gcc 源文件(*.c) -c
```
* 将得到的.o进行打包，得到静态库
```sh
ar rcs 静态库名字(libxxx.a) 原材料(*.0)
```
* 发布静态库
```sh
#发布静态库
1,提供头文件**.h
2,提供制作出来的静态库 libxxxx.a
```

## 静态库测试

> 通过对函数生成对应的静态库`.a`文件，然后根据其头文件以及对应的`静态库文件`，即可让其它函数进行调用

* `add.c`
```c
#include <stdio.h>
#include "head.h"

int add(int a, int b)
{
    return a+b;
}
```
* `sub.c`
```c
#include <stdio.h>
#include "head.h"

int subtract(int a, int b)
{
    return a-b;
}
```
* `mult.c`
```c
#include <stdio.h>
#include "head.h"

int multiply(int a, int b)
{
    return a*b;
}
```
* `div.c`
```c
#include <stdio.h>
#include "head.h"

double divide(int a, int b)
{
    return (double)a/b;
}
```
* `head.h`
```c
#ifndef _HEAD_H
#define _HEAD_H
// 加法
int add(int a, int b);
// 减法
int subtract(int a, int b);
// 乘法
int multiply(int a, int b);
// 除法
double divide(int a, int b);
#endif
```
* 调用文件`main.c`
```c
#include <stdio.h>
#include "head.h"

int main()
{
    int a = 20;
    int b = 12;
    printf("a = %d, b = %d\n", a, b);
    printf("a + b = %d\n", add(a, b));
    printf("a - b = %d\n", subtract(a, b));
    printf("a * b = %d\n", multiply(a, b));
    printf("a / b = %f\n", divide(a, b));
    return 0;
}
```
### 生成静态库
```sh
# 生成对应的汇编.o 文件
$ gcc add.c div.c mult.c sub.c -c
# 打包成静态库
ar rcs lib名称.a add.o div.o mult.o sub.o
//即可生成静态库
```
![静态库](https://i.imgtg.com/2023/04/24/Ie4fs.png "静态库")

# 动态库
> 动态库时`运行时`加载的库，是多个程序可以共用的`共享库`。动态链接库是目标文件的集合，使用相对地址，其真实地址是在应用程序加载动态库时形成的。`Linux`以`.so`作为后缀。`libxxx.so`.   `window`使用以`lib`作为前缀，以`dll`作为后缀，`libxxx.dll`.
> 动态链接库直接使用`gcc`命令需要添加` -fPIC (-fpic)` 以及`-shared`参数，其作用时使得gcc生成的代码与位置无关，也就是使用相对位置，`-shared`参数是`告诉`编译器生成一个`动态链接库`

## 生成动态链接库
* 将源文件进行汇编操作，需要使用`-c`，还需要添加额外参数`-fpic / -fPIC`
```sh
# 得到汇编.o 文件
gcc 源文件(*.c) -c -fpic/
# 将得到的.o文件打包成动态库使用参数-shared指定动态库
gcc -shared 有关的目标文件(*.o) -o 动态库(libxxx.so)
# 发布动态库和头文件
# 1,提供头文件  xxx.h
# 2,提供动态库: libxxx.so
```
### 无法加载问题
> 将库路径添加到环境变量 `LD_LIBRARY_PATH`中

#### 方法一
> 1,找到相关的配置文件
> 	* 用户级别: `~/.bashrc —> `设置对当前用户有效
>
>   * 系统级别:` /etc/profile `—> 设置对所有用户有效
> 2,使用`vim`打开配置文件，在文件最后添加
```sh
# 自己把路径写进去就行了
export LIBRARY_PATH=$LIBRARY_PATH:动态库的绝对路径 
```
3, 让配置文件生效
* 修改用户级别的配置文件，关闭当前终端，即可
* 系统级别，重启
* 或者
```sh
source 作用让文件内容被重新加载
source ~/.bashrc          (. ~/.bashrc)
source /etc/profile       (. /etc/profile)
```

#### 方法二
> 更新`/etc/ld.so.cache`文件
* 找到动态库所在的绝对路径:`/home/robin/Library/`
* 使用vim修改`/etc/ld.so.conf`文件，将上面的路径添加，独占一行
* 更新`/etc/ld.so.conf`中的数据到`/etc/ld/so.cache`中
```sh
sudo ldconfig
```

#### 方法三
> 拷贝拷贝动态库文件到系统库目录 `/lib/` 或者` /usr/lib `中 (或者将库的软链接文件放进去)
```sh
# 库拷贝
sudo cp /xxx/xxx/libxxx.so /usr/lib
# 创建软连接
sudo ln -s /xxx/xxx/libxxx.so /usr/lib/libxxx.so
```

## 原理
> 静态库:在程序编译的最后一个阶段--链接阶段，提供静态库被打包到可执行程序中，当可执行程序执行，静态库中的代码也会一并加载到内存中
> 动态库: 使用参数-L 查看库文件是否存在。在程序执行先检测是否动态库被加载，当动态库的函数在程序中被调用，这个动态库被加载到内存，不掉用不加载。动态库的检测和内存操作都是由动态连接器来完成

## 对比
* 静态库:
	* 优点:
	* 静态库被打包到应用程序中加载速度快
	* 发布程序无需提供静态库，移植方便
	* 缺点:
	* 相同的库文件数据被加载多份，消耗资源，浪费内存
	* 库文件更新需要重新编译项目文件，生成新的可执行程序浪费时间
* 动态库
	* 优点
	* 不同进程资源共享
	* 动态库升级，只需替换库文件，可以控制何时加载动态库
	* 缺点:
	* 加载速度慢，
	* 发布程序需要依赖动态库
# Makefile

> make是一个命令工具，是一个解释makefile中指令的命令工具。
> Makefile---自动化编译，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。

## 规则
```sh
target1,target2...: depend1,depend2, ...
	command
	......
```
* `command`：命令，一般情况下这个动作就是一个shell命令
* `depend`: 规则所必需的依赖条件，
* `target`: 目标，目标和规则在命令中是对应的。


> 在调用`Makefile`命令编译程序时，`make`会首先找到Makefile文件中的第一个规则，分析并执行相关的动作

> 根据上面的代码

![image-20230424123317849](E:\学习笔记\C-notebook\Linux系统编程\assets\image-20230424123317849.png)

> 运行结果

![image-20230424123338546](E:\学习笔记\C-notebook\Linux系统编程\assets\image-20230424123338546.png)

## 自动推导
> 当源文件的编译规则`不明确`时，make会使用`默认`的编译规则，按照默认规则完成对`.c`文件你的编译，`生成`对应的`.o`文件，它使用`cc -c` 编译`.c`源文件。

## 变量
> 为了使`makefile`进行规则设定时更加的`灵活`。

### 自定义变量
> 在对makefile进行规则设定时，用户可以自己定义变量，用户自定义变量。其变量没有类型，直接赋值即可
```sh
# 定义变量名,定义 多个中间用空格即可
变量名=变量值
# 取出变量名
$(变量名)
```
#### 自定义变量自动推导

```sh
obj = add.o sub.o mult.o div.o main.o
target = calc
$(target) : $(obj)
        gcc $(obj) -o $(target)
```
### 预定义变量
> 此变量为Makefile已经预先定义的，变量名 都是大写，可以直接使用这些变量。

| 变 量 名 | 含 义                        | 默 认 值 |
| -------- | ---------------------------- | -------- |
| `AR `     | 生成静态库库文件的程序名称   | `ar`       |
| `AS `      | `汇编`编译器的名称             | `as`       |
|` CC `      | `C` 语言编译器的名称           | `cc  `     |
| `CPP `     | `C `语言预编译器的名称         | `$(CC) -E` |
| `CXX`      | `C++` 语言编译器的名称         | `g++`      |
| `FC `      | `FORTRAN` 语言编译器的名称     | `f77`      |
| `RM`       | 删除文件程序的名称           | `rm -f`    |
| `ARFLAGS ` | 生成静态库库文件程序的选项   | 无默认值 |
| `ASFLAGS`  | 汇编语言编译器的编译选项     |          |
| `CFLAGS`   | C 语言编译器的编译选项       |          |
| `CPPFLAGS` | C++ 语言编译器的编译选项     |          |
| `FFLAGS `  | FORTRAN 语言编译器的编译选项 |          |

### 自动变量
> 自动变量用来代表这些规则中的目标文件和依赖文件，并且他们只能在规则的命令中使用

| 变量 | 含 义                                                        |
| :--- | ------------------------------------------------------------ |
| `$*`   | 表示目标文件的名称，不包含目标文件的扩展名                   |
| `$+ `  | 表示所有的依赖文件，这些依赖文件之间以空格分开，按照出现的先后为顺序，其中可能 包含重复的依赖文件 |
|` $< `  | 表示依赖项中第一个依赖文件的名称                             |
| `$? `  | 依赖项中，所有比目标文件时间戳晚的依赖文件，依赖文件之间以空格分开 |
|` $@  ` | 表示`目标文件`的`名称`，包含`文件扩展名 `                          |
| `$^  ` | 依赖项中，所有`不重复`的`依赖文件`，这些文件之间以`空格`分开       |

```sh
calc2 : add.o sub.o mult.o div.o main.o
        $(CC) $^ -o $@
```
## 模式匹配
> 模式匹配相当于一个模版，第一个规则中依赖的生成，都需要基于此规则来完成，得到所有的依赖，模式规则被执行了5次，模式规则中`%` 对应的文件名是`时时变化`的，因此命令的依赖的名字，必须要使用`自动变量`。

```sh
# 模式匹配 -> 通过一个公式，代表若干个满足条件的规则
# 依赖有一个，后缀为.c 生成的目标是一个.o的文件，%是一个通配符，匹配的是文件名
%.o : %.c
	gcc $< -c
```
```sh
calc2 : add.o sub.o mult.o div.o main.o
        $(CC) $^ -o $@
%.o:%.c
        gcc &< -c
```
## 函数
> 函数格式为:`$(函数名 参数1,参数2,...)` 目的为快速方便的得到`函数返回值`
### withcard获取文件名

```sh
# 参数只有一个，但此参数可以分为若干个部分，通过空格间隔
# 参数为某个目录，搜索这个路径下指定类型的文件: *.c
$(withcard PATTERN...)
```
#### 应用案例
> 搜索三个不同目录下`.c`格式的源文件
```sh
src = $(withcard /home/robin/a/*.c /home/robin/b/*.c *.c)
# 返回值得到一个大的字符串，里面有若干个满足条件的文件名，文件名之间使用空格间隔
```
### patsubst 替代指定文件后缀
> 按照`指定的模式`替换指定的文件名的后缀，函数原型为
```sh
$(patsubst <pattern>,<replacement>,<text>)
```
* 参数
* `pattern `：需要指定要替换的文件名的`后缀`是什么，使用`%.c`
* `replacement`：模式投资富川，指定参数`pattern`中的后缀，`最终要替换`为什么。
* `text`:该参数中存储这要被替换的原始数据
* `返回值`：该参数中存储这要被替换的原始数据
```sh
src = a.cpp b.cpp c.cpp e.cpp
# 把变量src中的所有文件名的后缀从.cpp替换为.o
obj = $(patsubst %.cpp,%.o,$(src))
# obj的值为a.o b.o c.o e.o
```

## Makefile完整案例
```sh
# 使用函数搜索当前目录下的源文件
src = $(wildcard *.c)
# 将源文件的后缀替换为.o
obj = $(patsubst %.c,%.o,$(src))

target = calc

$(target):$(obj)
        gcc $(obj) -o $(target)
%.o : %.c
        gcc $< -c

# 添加规则，删除生成文件，*.o可执行程序
# 声明clean为伪文件
.PHONY:clean

clean:
        # shell命令前的 - 表示强制执行这条指令，如果执行失败也不会终止
        - rm $(obj) $(target)
        echo "hello"
```



> 参考资料:
>
> ​	爱编程的大丙

