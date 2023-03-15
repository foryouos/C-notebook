# C++封装MySQL API

## 什么时候调用释放结果集
* 析构函数
* 可能会对数据库进行多次查询，每次查询一次都会得到结果集，查询是清空掉上次的结果集

## 源代码

### `MySQLConnect.h`
```cpp
#pragma once
#ifndef MYSQLCONNECT_H
#define MYSQLCONNECT_H
#include <iostream>
#include <mysql.h>
using namespace std;
class MySqlConnect
{
private:
	//什么时候调用释放结果集
	//1,析构函数2，可能会对数据库进行多次查询，每次查询一次都会得到结果集，查询是清空掉上次的结果集
	void freeResult(); //释放结果集
	MYSQL* m_conn = nullptr; //保存MySQL初始化的私有成员
	MYSQL_RES* m_result = nullptr; //报错结果集
	MYSQL_ROW m_row = nullptr; //保存着当前字段的所有列的数值
public:
	//初始化数据库连接
	MySqlConnect();
	//释放数据库连接
	~MySqlConnect();
	//连接数据库,使用默认端口可省略端口书写
	bool connect(string user, string passwd,string dbName,string ip,unsigned short port = 3306 );
	//更新数据库(插入，更新，删除)，传递字符串

	bool update(string sql);
	//查询数据库,单词query:查询
	bool query(string sql);

	//遍历查询得到的结果集,每调一次，从结果集中取出一条数据

	bool next(); 
	//得到结果集中的字段值，取记录里面字段方法
	string value(int index); 
	//事务操作，关闭自动提交
	bool transaction();
	//提交事务
	bool commit();
	//事务回滚;
	bool rollback();
};
#endif // !MYSQLCONNECT_H
```
### `MySQLConnect.cpp`
```cpp
#include "MySQLConnect.h"

void MySqlConnect::freeResult()
{
	if (m_result)
	{
		mysql_free_result(m_result);
		m_result = nullptr;
	}
}

MySqlConnect::MySqlConnect()
{
	m_conn = mysql_init(nullptr);
	mysql_set_character_set(m_conn, "utf8");
}

MySqlConnect::~MySqlConnect()
{
	if (m_conn != nullptr)
	{
		mysql_close(m_conn);
	}
	freeResult();
}

bool MySqlConnect::connect(string user, string passwd, string dbName, string ip, unsigned short port)
{
	//ip传入为string，使用.str将ip转为char *类型
	MYSQL* ptr = mysql_real_connect(m_conn, ip.c_str(), user.c_str(), passwd.c_str(), dbName.c_str(), port, nullptr, 0);
	//连接成功返回true
	//如果连接成功返回TRUE，失败返回FALSE
	return ptr!=nullptr;
}

bool MySqlConnect::update(string sql)
{
	//query执行成功返回0
	if (mysql_query(m_conn, sql.c_str()))
	{
		return false;
	};

	return true;
}

bool
MySqlConnect::query(string sql)
{
	freeResult();
	//query执行成功返回0
	if (mysql_query(m_conn, sql.c_str()))
	{
		return false;
	};
	m_result = mysql_store_result(m_conn);
	return true;
}

bool MySqlConnect::next()
{
	//如果结果集为空则没有必要遍历
	if (m_result != nullptr)
	{
		//保存着当前字段的所有列的数值
		m_row = mysql_fetch_row(m_result);
		return true;
	}
	return false;
}

string MySqlConnect::value(int index)
{ 
	//表示列的数量
	int row_num = mysql_num_fields(m_result); //函数得到结果集中的列数
	//如果查询的的index列大于总列，或小于0，是错误的
	if (index >= row_num || index < 0)
	{
		return string();
	}
	char* val = m_row[index]; //若为二进制数据，中间是有"\0"的
	unsigned long length = mysql_fetch_lengths(m_result)[index];
	return string(val,length); //传入length就不会以"\0"为结束符，而是通过长度把对应的字符转换为string类型
}

bool MySqlConnect::transaction()
{
	return mysql_autocommit(m_conn,false); //函数返回值本身就是bool类型
}

bool MySqlConnect::commit()
{
	return mysql_commit(m_conn);//提交
}

bool MySqlConnect::rollback()
{
	return mysql_rollback(m_conn);//bool类型，函数成功返回TRUE，失败返回FALSE
}
```