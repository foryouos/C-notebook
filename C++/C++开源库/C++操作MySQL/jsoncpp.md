# JSONcpp

## 开发环境配置

### 现成版百度网盘

[jsoncpp现成配置环境VS2022开发环境x64](../../C++%E5%BC%80%E6%BA%90%E5%BA%93/C++%E6%93%8D%E4%BD%9CMySQL/%E9%93%BE%E6%8E%A5%EF%BC%9Ahttps:/pan.baidu.com/s/1-DjqbRVVSmDizcD8wbEGFg)

> 链接：https://pan.baidu.com/s/1-DjqbRVVSmDizcD8wbEGFg?pwd=5213 提取码：5213

### 环境配置版

#### 下载Jsoncpp

```sh
git clone https://github.com/open-source-parsers/jsoncpp
```

#### cmake进行编译

#### 下载cmake

cmake下载链接 ![cmake下载文件](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYA1hiaSsHibx5wxf3SZa7oFTibVcJEsd8ia6iampzdjrRHvjGRO6duSHN7gg/0?wx\_fmt=png)

#### 解析出动态

* 安装`cmake`默认安装即可
* 将`jsoncpp所在文件夹，以及生成`jsoncpp.dll和jsoncpp.lib\`等动态文件的文件夹 ![填入jsoncpp文件夹](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYNbweGGibvwcvUknfulqk893zicuQ5tWeCGqF4YC7xWoXiaZyKoP9xw5iag/0?wx\_fmt=png)
* 第一个为VS的版本，第二个默认`x64`不用填 ![版本配置](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYY8pibHGcUPtWKamg0EEZ8OM02SntdWsiaoIg5BQrsR0asicqrfZjBQZPLQ/0?wx\_fmt=png)

***

* 配置不动，点击`生成`即可，会发现生成文件逐步完成 ![生成](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYS1icpcckPtrk3tHj3wfibrcNXOLOz3sRVWxF34TLlYyJxNlwDKoBI4jA/0?wx\_fmt=png)

***

* `打开`生成文件的项目 ![打开生成文件的项目](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYkpUxzHgBjy7k0DcmVXkEGic5oWhiaBPb1YZXFhOOxcgAIeoJa1CfazRg/0?wx\_fmt=png)

***

* 将`jsoncpp_dll`设置为`启动项`，并右键点击第一个`生成`即可完成静态文件的生成 ![静态文件生成](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYG3ogN50qSibbnl2VWwonazExaDkWjJ0SSiceW3PMdibLoYxicAbcmVMm8g/0?wx\_fmt=png)

***

* 可以在C盘`新建`一个存放环境的位置cpp
* 将下载`clone`的jsoncpp文件下的include文件全部放入C盘文件夹cpp中(此文件为jsoncpp的头文件)
* 将生成文件之后项目文件夹中`bin\Debug`下的`jsoncpp.dll`复制到C盘文件夹cpp下与`include同级`的`lib`文件夹(自己创建)下
* 将生成文件之后项目文件加中lib\Debug下的`jsoncpp.lib`复制到C盘cpp文件夹`lib`文件下下 ![文件夹](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYrbJL3c8nVpqefTuicABAR2Fb76kFpySaiax770ic3z70AOhN3N5Wia1b2w/0?wx\_fmt=png)

***

#### VS2022环境配置

* 右键点击项目进入属性配置 ![VS配置jsoncpp](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYcH0ia4wcLI7ozIMmXYqmicIylF3QPrPoylev5IXNial7FjWjX2T0PQLnA/0?wx\_fmt=png)

![VS配置jsoncpp](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYpUO34ZBQQ4RMDTcQflVoXgRQp6icfFDZSfPy8b2eMNz7SMyd7W4ibVdA/0?wx\_fmt=png)

***

#### 最后配置

> 运行下面的实例代码很可能出现“运行程序无法启动的问题”,将(刚才C盘中)`jsoncpp.dll`文件复制到`main.cpp`对应的资源文件夹下即可 ![VS运行最后配置](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYRjfdyU261d0YPzGD4Of8RlfQibjrGeLMKPliaicqhnKDv90PZVZuFfhFA/0?wx\_fmt=png)

## 实例代码

```cpp
#include <json/json.h>
#include <fstream> 
#include <iostream>
using namespace std;
using namespace Json;

//对于Json数组而言，内部的顺序是有序的，这个顺序就是添加数据的顺序
void writeJson()
{
    //定义Value对象
    Value root;
    //使用append方法向Value包装器li填充数据
    root.append("luffy");
    root.append(19);
    root.append(170);
    root.append(false);

    //又定义一个subArray的Value对象
    Value subArray;
    subArray.append("ace");
    subArray.append("sabo");
    //将subArray对象增加到root里面
    root.append(subArray);

    Value obj;
    //定义键值对
    obj["sex"] = "man";
    obj["girlfriend"] = "Hancock";
    //将obj对象的键值对增加到root里面
    root.append(obj);
#if 1
    //对上面数据格式序列化使用toStyledString方法得到带格式的字符串
    string json = root.toStyledString();
#else
    //得到不带格式的字符串
    FastWriter w;
    //通过write方法得到不带换行符的字符串   
    string json = w.write(root);
#endif
    //写文件
    //write  ->  ostream 
    //read   ->  ifstream   
    //将json数据写入test.json
    ofstream ofs("test.json");
    //写入格式化后带格式的字符串
    ofs << json;
    //关闭文件
    ofs.close();
}
/*保存到的json数据
[
    "luffy",
    19,
    170,
    false,
    [
        "ace",
        "sabo"
    ],
    {
        "girlfriend" : "Hancock",
        "sex" : "man"
    }
]
*/

void readJson()
{
    //读，打开test.json
    ifstream ifs("test.json");
    Reader rd;
    Value root;
    //将json数据解析到root里面
    rd.parse(ifs, root);
    //判断解析的root value类是否是Array
    if (root.isArray())
    {
        //size方法判断
        for (unsigned i = 0; i < root.size(); ++i)
        {
            Value item = root[i];
            //对类型进行判断，并对对应的类型进行输出
            if (item.isInt())
            {
                cout << item.asInt() << " ,";
            }
            else if (item.isString())
            {
                cout << item.asString() << " ,";
            }
            else if (item.isBool())
            {
                cout << item.asBool() << " ,";
            }
            else if (item.isArray())
            {
                for (unsigned j = 0; j < item.size(); ++j)
                {
                    cout << item[j].asString() << ", ";
                }
            }
            else if (item.isObject())
            {
                //Return a list of the member names.
                Value::Members keys = item.getMemberNames();
                for (int k = 0; k < keys.size(); ++k)
                {
                    cout << keys.at(k) << "," << item[keys[k]];
                }
            }
            cout << endl;
        }
    }
}

int main()
{
    writeJson();
    printf("json写入完毕");
    readJson();
   

    return 0;
}
```

## 具体用法

### 声明命名空间

```cpp
using namespace Json;
```

### jsoncpp的三个类

#### `Value类`

> `Value类`:封装`Json`支持的所有类型，方便后序处理数据

**Value检测类型**

> 使用`Value.isNull()`判断是够为空，输出`bool值`

```cpp
  bool isNull() const;
  bool isBool() const;
  bool isInt() const;
  bool isInt64() const;
  bool isUInt() const;
  bool isUInt64() const;
  bool isIntegral() const;
  bool isDouble() const;
  bool isNumeric() const;
  bool isString() const;
  bool isArray() const;
  bool isObject() const;
```

**将`Value`转换为`对应类型`**

```cpp
  Int asInt() const;
  UInt asUInt() const;
#if defined(JSON_HAS_INT64)
  Int64 asInt64() const;
  UInt64 asUInt64() const;
#endif // if defined(JSON_HAS_INT64)
  LargestInt asLargestInt() const;
  LargestUInt asLargestUInt() const;
  float asFloat() const;
  double asDouble() const;
  bool asBool() const;
```

#### FastWriter类

> 将`Value`对象中的数据序列化为`字符串`

```cpp
std::string Json::FastWriter::write(const Value& root);
```

* 将数据格式化，不带换行符，编程单行

#### Reader类

> 反序列化，将`json字符串`解析为`Value类型`

**`parse`方法**

```cpp
bool Json::Reader::parse(const std::string& decument,Value& root,bool collectComments=true);
```

参数:

* `document:json`格式字符串，`打开`的json文件
* root：传出参数，`存储`了json字符串`解析`出的数据
* collectComments：是否保存`注释信息`，默认`True`

```cpp
 bool parse(const char* beginDoc, const char* endDoc, Value& root,bool collectComments = true);
```

参数：

* bneginDoc和endDoc`指针定位`json字符串，解析`部分`json字符串

```cpp
  //brief Parse from input stream.
  //see Json::operator>>(std::istream&, Json::Value&).
bool parse(IStream& is, Value& root, bool collectComments = true);
```
