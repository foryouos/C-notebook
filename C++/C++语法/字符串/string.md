# string 

> C++字符串类，容器类，可以吧字符串当做普通类型使用

# 构造函数



```cpp
string s();  //生成空的字符串s
string s(str); //拷贝构造函数
string(const string& str); // 生成str的复制品
//将字符串str中从pos到长度为strlen,作为str初值
string s(const string& str, size_type pos,strlen); 
//舒婷str的前n个字符串初始化为字符串s的初值
string s(const char* cstr, size_type n);
//生成一个字符串包含num个c字符
string s(int num,char c);
string s(beg,end);//以区间beg,end内的字符作为字符串s的初值
s.~string();  //销毁所有字符，释放空间
    
```

# 操作函数

```cpp
=，assign();  //赋以新值
//assign第一个参数为复制的自付出啊，第二第三位返回，开始和结束范围
swap();//交换两个字符串的内容
+=，append(),push_back(); //在尾部添加字符
//push_back()在结尾添加单个字符，仅一个
insert();  //插入字符
//需要指定插入的索引insert(1,"hello");
erase(int nStart,int nEnd); //删除start-End的字符
clear();        //删除全部字符
replace(); //替换字符
//需要添加索引
s.replace(1,2,"hello"); 
//将索引1开始到2替换为后面的hello
+   //串联字符串
==,!=,<,<=,>,>=,compare(); //比较字符串
size(),length();   //返回字符串数量
max_size(); //返回字符串饿可能最大个数
empty(); //判断字符串是否为空
capacity(); //返回重新分配之前的字符容量,会为string重新分配内存
reserve();  //保留一定量内容，以容纳一定数量字符
[],at();   //存取单一字符
// []并不检查 索引是否有效，有效索引0~str.length()
>>,getline(); //从stream读取某值
<<         //将某值写入stream
copy();   //将某值复制为一个c_string
to_str();  //将内容以以c_string返回
data();   //将内容以字符数组形式返回
substr(); //返回某个子字符串，提取部分字符串
// 添加开始和结束索引
s.substr(5,6); //5~6的字符串
s.substr(11);  //从11往后的子串
// 搜索
// 搜索符合条件的字符区间，返回第一个字符的索引，没找到目标返回npos --多数为size大小
find();
rfind();
find_first_of();;
find_last_of();
find_first_not_of();
find_last_not_of();

begin(),end(); //提供类似STl的迭代器支持
rbegin(),rend(); //逆向迭代器
get_allocator();  //返回配置器
```

# 使用STL算法

```cpp
//初始化值
string name = "marius";
// 将字符串全部转为大写
trandform(name.begin(),name.end(),name.begin(),toupper);
// 升序排列字符串
sort(name.begin(),name.end());
//字符串反转
reverse(name.begin(),name.end());
// 删除空白字符
bool iswhitespace(char ch)
{
    return ch ==''|| ch == '\t'|| ch=='\v'|| ch=='\r'|| ch =='\n';
}
newend = remove_if(name.begin(),name.end(),iswhitespace);
```