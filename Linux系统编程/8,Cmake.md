# `Cmake`笔记

> `Cmake`允许开发者指定整个工程的编译流程，再根据编译平台，自动生成本地化的`makefile`和工程文件，只需`make`即可`编译`。一款自动生成`Makefile`的工具



## `Cmake`优点

* 跨平台
* 能够管理大型项目
* 简化编译构建过程和编译过程
* 可扩展:可以为`cmake`编写特定功能的模块，扩充`cmake`功能

## `Cmake`使用

> `Cmake`不区分大小写。

### 注释

```cmake
# 这是一个行注释
#[[ 这是一个
快注释 ]]
```

### 测试文件
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
### 添加`CMakeLists.txt`

```cmake
# 指定使用的cmake最低版本
cmake_minimum_required(VERSION 3.0)
#[[ project 定义工程名称,
并可指定工程的版本，
工程描述，
web主页地址，
支持的语言，
# PROJECT 指令的语法是：
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>    
       [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
       [DESCRIPTION <project-description-string>]
       [HOMEPAGE_URL <url-string>]
       [LANGUAGES <language-name>...])

]]
project(CALC)
# add_executable：定义工程并生成一个可执行文件
# add_executable(可执行文件名 源文件名称)
add_executable(app add.c div.c main.c mult.c sub.c)
# 此可执行文件与project项目名无关系，文件名多个可以`;`间隔，也可不用
```
### 执行
```cmake
# cmake命令原型
cmake CMakeLists.txt文件所在路径
# 执行此命令后会生成Makefile文件
make
# 执行make执行makefile执行，即可将可执行程序编译出来
```
### 存入build
```cmake
# 进入项目文件夹
mkdie build
cd build
# 执行cmake,即可将所生成的文件存入cmake当中
# cmake在build文件内执行，文件地址在..上一级目录当中，即当前目录的上一级
cmake ..
```
### 定义变量
```cmake
# set指令语句 []中的参数为可选性
set(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
```
* VAR : 变量名
* VALUE: 变量值
```cmake
set(SRC_LIST add.c;div.c;main.c;mult.c;sub.c)
add_executable(app ${SRC_LIST})
```
### 指定使用的标准
* 普通编译使用
```sh
g++ *.cpp -std=c++11 -o app
```
* 使用`CMakeLists.txt`中通过`set`命令指定
	* 在`txt`文件中指定
```cmake
# 设置C++11标准编译
set(CMAKE_CXX_STANDARD 11)
```
	* 在执行cmake时指定
```cmake
cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=11
```
### 指定输出路径
> `Cmake`指定可自行输出的路径，对应宏:`EXECUTABLE_OUTPUT_PATH`，通过set命令进行设置值
```cmake
# 定义变量用于存储一个绝对路径
set(HOME /home/robin/linux/Sort)
# 将拼好的路径设置给 宏，若路径不存在将自动生成，使用相对路径，生成的makefile将对应makefile所在的路径
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
```
### 搜索文件
> 使用`aux_source_directory`命令或者`file`命令
```sh
aux_source_directory(< dir > < variable >)
```
* `dir`要搜索的目录
* `variable` 将从`dir`目录下搜索到的`源文件`列表存储到该变量中
```sh
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJEcT_SOURCE_DIR}/include)
# 搜索src目录下的源文件
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIE}/src SRC_LIST)
add_executable(app ${SRC_LIST})
```
#### file
```sh
file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
```
* `GLOB`:将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中
* `GLOB_RECURSE`：`递归`搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中

> 搜索当前目录的`src`目录下所有的源文件，并存储到变量中
> `CMAKE_CURRENT_SOURCE_DIR`表示当前访问的`CMakeLists.txt`文件所在的路径，关于要搜索文件路径和类型可加双引号，也可不加
```sh
file(GLOB MAIN_SRS ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
```
#### 包含头文件
```sh
include_directories(headpath)
```
#### `CMaeList.txt`文件内容

```sh
cmake_minimum_required(VERSION 3.0)
project(CALC)
set(CMAKE_CXX_STANDARD 11)
set(HOME /home/robin/Linux/calc)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin/)
# PROJECT_SOURCE_DIR 宏对应的值就是在使用cmake命令时后面紧跟的目录，一般为工程的根目录
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
add_executable(app ${SRC_LIST})
```
## 制作静态库
> Linux静态库分为三个部分: `lib` + `库名称` + `.a`
```sh
add_library(库名称 STATIC 源文件1 [源文件2] ...)
# 例如
# 设置动态库/静态库生成路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
add_library(calc STATIC ${SRC_LIST})
# 最终生成对应的静态库文件libcalc.a
```
## 制作动态库
> `Linux`动态库名称分为` lib + 库名称 + .so `
```sh
# 仅可设置动态链接库生成路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
add_library(库名称 SHARED 源文件 [源文件2] ... )
```
## 链接静态库
```sh
link_libraries(<static lib> [<static lib> ...])
```
* 参数1,指出要链接的静态库名称，可以是全名，也可以是去掉头lib和尾.a之后的名字，
* 参数2要链接的其它静态库名称

> 如果该静态库不是系统提供的，此时可以将静态库的路径指定出来
```sh
link_directories(<lib path>)
```
## 链接动态库
```sh
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
# target：指定要加载动态库的文件的名字
# PRIVATE|PUBLIC|INTERFACE：动态库的访问权限，一般无需指定默认为 PUBLIC
```
> 由于动态链接库在可执行程序中并不会被打包到可执行程序中，应该将命令写到可执行文件之后.通过命令指定要链接的动态库位置，指定静态库位置使用的命令。
```sh
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定源文件或者动态库对应的头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 指定要链接的动态库的路径
link_directories(${PROJECT_SOURCE_DIR}/lib)

# 添加并指定最终生成的可执行程序名
add_executable(app ${SRC_LIST})
# 指定可执行程序要链接的动态库名字
# app是最终生成的可执行程序的名字
# pthread可执行程序要加载的动态库，库的全名为libpthread.so
target_link_libraries(app pthread)
```
## 日志
> `CMake`的命令工具会在`stdout`上显示`STATUS`信息，在`stderr`上显示所有信息，`CMake`的GUI会在它的log区域显示所有信息，`CMake`警告和错误信息的文本显示使用一种简洁的标记语言，文本没有缩进，超过长度的行会回卷，段落之间以新行作为分隔符
```sh
message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR] "message to display" ...)
```
* (无) ：重要消息
* `STATUS` ：非重要消息
* `WARNING`：`CMake` 警告，会继续执行
* `AUTHOR_WARNING`：`CMake` 警告 (`dev`), 会继续执行
* `SEND_ERROR`：`CMake` 错误，继续执行，但是会跳过生成的步骤
* `FATAL_ERROR`：`CMake `错误，终止所有处理过程
## 变量操作
> 有时候源文件并不一定都在一个目录中，但最后要编译器到一起，通过set或list命令对变量进行拼接

### 使用set`拼接`

```sh
set(变量名1 ${变量名} ${变量名2} ... )
# 从第二个参数往后所有的字符串进行拼接，最后将结果存储到第一个参数中。如果第一个参数中原来有数据将对原数据进行覆盖
```

### 使用list`拼接`

```sh
# APPEND表示进行数据追加，
list(APPEND <list> [<element> ...])


cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
# 追加(拼接)
list(APPEND SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
message(STATUS "message: ${SRC_1}")
```
### 使用list移除
> `REMOVE_ITEM`
```sh
list(REMOVE_ITEM <list> <value> [<value> ...])

cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/*.cpp)
# 移除前日志
message(STATUS "message: ${SRC_1}")
# 移除 main.cpp
list(REMOVE_ITEM SRC_1 ${PROJECT_SOURCE_DIR}/main.cpp)
# 移除后日志
message(STATUS "message: ${SRC_1}")

```
## `list`使用

* 获取`list`的长度
```cmake
list(LENGTH <list> <output variable>)
#LENGTH：子命令 LENGTH 用于读取列表长度
#<list>：当前操作的列表
#<output variable>：新创建的变量，用于存储列表的长度。
```
* 读取列表中指定索引的元素，可以指定多个索引
```cmake
list(GET <list> <element index> [<element index> ...] <output variable>)
#<list>：当前操作的列表
#<element index>：列表元素的索引
#从 0 开始编号，索引 0 的元素为列表中的第一个元素；
#索引也可以是负数，-1 表示列表的最后一个元素，-2 表示列表倒数第二个元素，以此类推
#当索引（不管是正还是负）超过列表的长度，运行会报错
#<output variable>：新创建的变量，存储指定索引元素的返回结果，也是一个列表。
```

* 查找列表是否存在指定的元素，若未找到，返回 -1
```cmake
list(FIND <list> <value> <output variable>)
#<list>：当前操作的列表
#<value>：需要再列表中搜索的元素
#<output variable>：新创建的变量
#如果列表 <list> 中存在 <value>，那么返回 <value> 在列表中的索引
#如果未找到则返回 - 1。
```
* 将列表中的元素用连接符(字符串)连接起来组成一个字符串
```cmake
list (JOIN <list> <glue> <output variable>)
#<list>：当前操作的列表
#<glue>：指定的连接符（字符串）
#<output variable>：新创建的变量，存储返回的字符串
```
* 将元素追加到列表中
```cmake
list (APPEND <list> [<element> ...])
```
* 在list中指定的位置插入若干元素
```cmake
list(INSERT <list> <element_index> <element> [<element> ...])
```
* 在元素插入到列表的0索引为止
```cmake
list (PREPEND <list> [<element> ...])
```
* 将列表中最后元素移除
```cmake
list (POP_BACK <list> [<out-var>...])
```
* 将指定的元素从列表中移除
```cmake
list (REMOVE_ITEM <list> <value> [<value> ...])
```
* 移除列表中重复元素
```cmake
list (REMOVE_DUPLICATES <list>)
```
* 列表翻转
```cmake
list(REVERSE <list>)
```
* 列表`排序`
```cmake
list (SORT <list> [COMPARE <compare>] [CASE <case>] [ORDER <order>])
#[[ COMPARE：指定排序方法。有如下几种值可选：
STRING: 按照字母顺序进行排序，为默认的排序方法
FILE_BASENAME：如果是一系列路径名，会使用 basename 进行排序
NATURAL：使用自然数顺序排序
CASE：指明是否大小写敏感。有如下几种值可选：
SENSITIVE: 按照大小写敏感的方式进行排序，为默认值
INSENSITIVE：按照大小写不敏感方式进行排序
ORDER：指明排序的顺序。有如下几种值可选：
ASCENDING: 按照升序排列，为默认值
DESCENDING：按照降序排列 ]]
```


## 宏定义
* 在源代码中进行`宏`定义
```cpp
#define NUMBER 1
```
* 在编译的时候进行宏定义
```sh
# 通过参数-D指定要定义的宏的名称
gcc test.c -DDEBUG -o app
```
* 在`CMake`中
```cmake
add_definitions(-D宏名称)
```



## 预宏定义列表

| 宏   | 功能 |
| ---- | ---- |
|`PROJECT_SOURCE_DIR`| 使用` cmake `命令后紧跟的目录，一般是工程的根目录 |
|`PROJECT_BINARY_DIR`	|执行 `cmake `命令的目录|
|`CMAKE_CURRENT_SOURCE_DIR`	|当前处理的 `CMakeLists.txt `所在的路径|
|`CMAKE_CURRENT_BINARY_DIR`| `target `编译目录 |
|`EXECUTABLE_OUTPUT_PATH`|	重新定义目标二进制可执行文件的存放位置|
|`LIBRARY_OUTPUT_PATH`	|重新定义目标链接库文件的存放位置|
|`PROJECT_NAME`	|返回通过` PROJECT` 指令定义的项目名称|
|`CMAKE_BINARY_DIR`	|项目实际构建路径，假设在` build `目录进行的构建，那么得到的就是这个目录的路径|

## 参考资料:
* 爱编程的大丙
