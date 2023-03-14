# C MySQL API详解

## 普通调用

### 开发环境
> 需要引用头文件`#include <mysql.h>`
* 默认已经安装了MySQL环境
* 配置VS2022项目属性
	![MySQL开发环境目录地址](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv2VGCVVXKvuxwgrnANSZyHqeZDCa87ibzcZwLibvE23rslmo9ORjmBy7vQQDJr6gMp1xthYEFriaA6w/0?wx_fmt=png "MySql系统安装目录以及需要的文件夹")
	* 右键项目，进入属性
	* 需要在`属性`的VC++目录中包含目录和引用目录中分别添加MySQL安装文件夹下的include文件夹和lib文件夹
		![VS2022属性配置](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv2VGCVVXKvuxwgrnANSZyHP6Mcv1GtEglB2cIRmn0M0j1ia3WUlQg97oAXkPUC68bWMbftWD3GTBA/0?wx_fmt=png "VS2022属性配置")
	* 在属性中输入目录下的附加依赖项添加MySQL加密动态库
	![加密动态库位置](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv2VGCVVXKvuxwgrnANSZyHdp4bK9MEHAP9aaicllw6iadGuBk8jOFksrlYT5XDgZ4N9Y3jy2B9rZBw/0?wx_fmt=png "加密动态库系统位置")
	![添加MySQL加密动态库](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv2VGCVVXKvuxwgrnANSZyHN21ZEJEVWVwwafwPU5iarsm1Kow0ChWQct8dibRp0L2yK5Bsgiax9Uqbg/0?wx_fmt=png "加载MySQL加密动态库")
### 代码测试
```cpp
#include <stdio.h>
#include <mysql.h>

int main(void)
{
	printf("MySQL Environment Successful\n");
	return 0;
}
//引用头文件不报错，即环境匹配成功
```
## `MySql.h`用法
### 运行流程图
![C++调用MySQL流程图](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbv2VGCVVXKvuxwgrnANSZyHMUu5icNADLDCWTp2YEB4BiarKBy5G5Jj4hzL6xDGvCv4OqvMxTuG6ZbQ/0?wx_fmt=png "C++调用MySQL流程图")
### C连接MySQL语法详解
> 一共`两`个部分，一个通过一个小程序实例，通过注释的形式将MySQL API吃透，另一种是分解版逐步吃透。其为`MySQL.h内部头文件`
#### 合集版
##### 本地数据先进行的操作
```sql
show databases;
create database cpp;
use cpp;
show TABLES;

-- 创建dept部门表
CREATE TABLE dept ( 
d_id INT PRIMARY KEY,
d_name VARCHAR ( 20 ),
d_des VARCHAR ( 50 ) 
);

SHOW TABLES;

DESC dept;

INSERT INTO dept VALUES(01,'陆军','河南');
INSERT INTO dept VALUES(02,'第二陆军','开封');
INSERT INTO dept VALUES(03,'第三陆军','西安');
INSERT INTO dept VALUES(04,'第四陆军','山东');
INSERT INTO dept VALUES(05,'第五陆军','菏泽');
INSERT INTO dept VALUES(06,'第六陆军','商丘');
INSERT INTO dept VALUES(07,'第七陆军','郑州');
INSERT INTO dept VALUES(08,'第八陆军','洛阳');
SELECT * FROM dept;
```
##### C调用合集
```cpp
#include <stdio.h>
#include <mysql.h>

int main(void)
{
	printf("MySQL Environment Successful\n");

	//初始化运行环境
	//该函数将分配初始化，并返回新对象，并通过返回这个对象去连接MySQL服务器
	MYSQL* mysql = mysql_init(NULL);

	//判断是否初始化成功
	if (mysql == NULL)
	{
		printf("mysql_init() error");
		return -1;
	}
	printf("数据库初始化成功\n");
	//连接MySQL服务器
	mysql = mysql_real_connect
	(
		mysql, //mysql_init()函数返回值
		"localhost", //mysql的IP地址
		"root", //myslq的用户名
		"5211314", //mysql的密码
		"cpp",   //使用的数据库名称
		3306,     //监听端口，0为默认3306端口
		NULL,   //本地套接字，不指定为NULL
		0       //默认0
	);
	//检查数据库连接情况
	if (mysql == NULL)
	{
		printf("mysql_real_connect() error\n");
		return -1;
	}
	printf("数据库连接成功\n");


	//输出MySQL API使用的编码,参数为数据库对象
	printf("mysql api使用的默认编码:%s\n", mysql_character_set_name(mysql));

	//设置数据库编码为utf8,
	//参数：数据库对象，编码
	mysql_set_character_set(mysql, "gbk"); //将编码改为gbk才可以进行数据插入，报错，乱码
	printf("mysql api修改后的编码为:%s\n", mysql_character_set_name(mysql));

	//执行sql语句
	// 查询cpp数据库下的dept部门表
	const char* sql = "select * from dept";
	//执行sql语句
	//参数：数据库对象，const char*的数据库执行语句
	//执行成功返回0
	int ret = mysql_query(mysql, sql);
	if (ret != 0)
	{
		//mysql_error返回mysql数据库对象错误描述
		printf("mysql_query()失败了，原因：%s\n", mysql_error(mysql));
		return -1;
	}

	//取出结果,参数mysql数据对象
	//MYSQL_RES对应一块内存八寸执行之后的结果集
	//若错误，返回NULL
	MYSQL_RES* res = mysql_store_result(mysql);
	if (res == NULL)
	{
		printf("mysql_store_result()失败了，原因是:%s\n", mysql_error(mysql));
		return -1;
	}

	//得到结果集中的列数
	//参数：传入mysql_store_result()得到的返回值
	int num = mysql_num_fields(res);

	//得到所有列的名字，并且输出
	//参数传入mysql_store_result()得到的返回值
	//MYSQL_FIELD对应的是一个结构体
	MYSQL_FIELD* fields = mysql_fetch_fields(res);
	//遍历输出mysql的name
	for (int i = 0; i < num; i++)
	{

		printf("%s\t\t", fields[i].name);
	}
	printf("\n");
	
	//遍历结果集中的所有行
	//参数：mysql_store_result()得到的返回值
	// 成功得到当前记录中每一个字段的值
	// 失败/数据读完 返回NULL
	MYSQL_ROW row;
	while ((row = mysql_fetch_row(res)) != NULL)
	{
		// 将当前列中的每一列信息读出
		for (int i = 0; i < num; ++i)
		{
			printf("%s\t\t", row[i]);
		}
		printf("\n");
	}

	//释放资源 -结果集
	mysql_free_result(res);

	//写入数据
	//MySQL默认自动提交数据库
	//设置事务为手动提交
	mysql_autocommit(mysql, 0);
	//执行成功返回0
	int ret1 = mysql_query(mysql, "insert into dept values(10,'第一海军','海南')");
	int ret2 = mysql_query(mysql, "insert into dept values(11,'第二海军','福建')");
	int ret3 = mysql_query(mysql, "insert into dept values(12,'第三海军','辽宁')");

	printf("ret1=%d  ret2 = %d  ret3 = %d\n", ret1, ret2, ret3);

	if (ret1 == 0 && ret2 == 0 && ret3 == 0)
	{
		printf("执行成功，正在提交事务\n");
		//提交事务
		mysql_commit(mysql);
	}
	else
	{
		printf("执行失败，正在回滚....\n");
		printf("执行失败原因: %s\n", mysql_error(mysql));
		mysql_rollback(mysql);
	}
	//释放数据库资源
	mysql_close(mysql);

	return 0;
}
```
#### 分解头文件
##### 初始化运行环境
```cpp
//该函数将分配初始化，并返回新对象，并通过返回这个对象去连接MySQL服务器，得到一块内存并保存
MYSQL* mysql_init(MYSQL * mysql);
//参数NULL
```
##### 连接数据库
```cpp
MYSQL * mysql_real_connect(
MYSQL *mysql,   //mysql_init()函数的返回值即指向的指针
const char *host, //主机地址ip地址，localhost,NULL代表本地连接
const char *user, //服务器用户名
const char *passwd,//连接服务器密码
const char *db,   //要使用数据库名称
unsigned int port,  //监听的端口号，若为0，则默认mysql的3306端口
const char *unix_socket, //本地套接字,不指定为NULL
unsigned long clientflag //通常指定为0
);
```
##### 执行SQL语句

```cpp
int mysql_query(MYSQL *mysql,const char *query);
```
参数:
* mysql:mysql_real_connect()的返回值
* query : 可以执行的sql语句，不要`;`
返回值:
* 成功返回`0`就，结果集在MySQL对象中
* 错误返回`非0`
##### 获取结果集
* 将结果集从MySQL(参数)对象中取出
* MYSQL_RES对应一块内存，保存着查询之后的结果集
* 返回具有多个结果的MYSQL_RES结果集合
* 若错误，返回NULL
```cpp
MYSQL_RES *mysql_store_result(MYSQL *mysql);
```
##### 获取结果集的列数
```cpp
unsigned int mysql_num_fields(MYSQL_RES *result);
```
参数
* 调用mysql_store_result()得到的返回值
返回值
* 结果集中的列数
##### 获取结果集中所有列的名字
```cpp
MYSQL_FIELD *mysql_fetch_fields(MYSQL_RES *result);
```
参数
* mysql_store_result()得到的结果集
返回值
* `MYSQL_FIELD*` 指向的一个结构体
###### 结构体定义
```c
typedef struct MYSQL_FIELD {
  char *name;               /* Name of column  */
  char *org_name;           /* Original column name, if an alias */
  char *table;              /* Table of column if column was a field */
  char *org_table;          /* Org table name, if table was an alias */
  char *db;                 /* Database for table */
  char *catalog;            /* Catalog for table */
  char *def;                /* Default value (set by mysql_list_fields) */
  unsigned long length;     /* Width of column (create length) */
  unsigned long max_length; /* Max width for selected set */
  unsigned int name_length;
  unsigned int org_name_length;
  unsigned int table_length;
  unsigned int org_table_length;
  unsigned int db_length;
  unsigned int catalog_length;
  unsigned int def_length;
  unsigned int flags;         /* Div flags */
  unsigned int decimals;      /* Number of decimals in field*/
  unsigned int charsetnr;     /* Character set */
  enum enum_field_types type; /* Type of field. See mysql_com.h for types */
  void *extension;
} MYSQL_FIELD;
```
##### 获得结果集中字段的长度
```c
unsigned long *mysql_fetch_lengths(MYSQL_RES *result);
```
参数:
* result:通过查询得到的结果集

返回值
* 无符号长整数的数组表示各列的大小，若错误返回NULL

##### 遍历结果集

```C
MYSQL_ROW mysql_fetch_row(MYSQL_RES *result);
```
参数:
* result：通过查询得到的结果集
返回值:

* 成功:得到了当前记录中每个字段的值

* 失败/数据已经读完：NULL，

###### MYSQL_ROW

```C
typedef char** MYSQL_ROW;
```
* 返回值二级指针，`char**`指向一个指针数组，类型是数组，里面的元素仍未指针，`char*`类型，
* `char* []`：数组中的字符串对应的一列数据
* 如果想要遍历整个结果集，需要对函数MYSQL_ROW进行循环调用
##### 资源回收
```c
//释放结果集
void mysql_free_result(MYSQL_RES *result);
//关闭MySQL实例
void mysql_close(MYSQL *mysql);
```
##### 字符编码
###### 返回当前编码

```C
// 获取API默认使用的字符编码(当前连接返回默认的字符集)
const char * mysql_character_set_name(MYSQL *mysql);
```
###### 设置编码
```c
//设置API使用的字符集
int mysql_set_character_set(MYSQL *mysql,char *csname);
```
参数:
* csname为要设置的字符集:常用`utf8`
##### 事务
> MySql会默认进行事务提交，需要设置逻辑处理时，多吃写操作，开始时创建事务，结束时判断是否全部成功，再提交或处理错误
```C
my_bool mysql_autocommit(MYSQL *mysql,my_bool mode);
```
参数:
* `mode`如果模式为`1`，`启用`autocommit模式
* 如果模式为`0`，`禁止`autocommit模式
###### 事务提交
```C
my_bool mysql_commit(MYSQL *mysql);
```
返回值:
* 成功：0
* 失败: 非0
###### 数据回滚

> 回滚到事务处理前的状态

```c
my_bool mysql_rollback(MYSQL *mysql);
```
返回值:
* 成功： 0
* 失败： 非0

##### 打印错误信息

```c
//返回错误的描述
const char *mysql_error(MYSQL *mysql);
// 返回错误的编码
unsigned int mysql_error(MYSQL *mysql);
```