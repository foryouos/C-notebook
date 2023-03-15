#JSONcpp
## 开发环境配置
### 现成版
[jsoncpp现成配置环境VS2022开发环境x64]()
### 环境配置版
#### 下载Jsoncpp
```sh
git clone https://github.com/open-source-parsers/jsoncpp
```
#### cmake进行编译
#### 下载cmake
cmake下载[链接](https://cmake.org/download/)[^1]
![cmake下载文件](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYA1hiaSsHibx5wxf3SZa7oFTibVcJEsd8ia6iampzdjrRHvjGRO6duSHN7gg/0?wx_fmt=png "cmake下载文件")
#### 解析出动态
* 安装`cmake`默认安装即可
* 将jsoncpp所在文件夹，以及生成`jsoncpp.dll和jsoncpp.lib`等动态文件的文件夹
![填入jsoncpp文件夹](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYNbweGGibvwcvUknfulqk893zicuQ5tWeCGqF4YC7xWoXiaZyKoP9xw5iag/0?wx_fmt=png "填入jsoncpp所在文件夹")
* 第一个为VS的版本，第二个默认`x64`不用填
![版本配置](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYY8pibHGcUPtWKamg0EEZ8OM02SntdWsiaoIg5BQrsR0asicqrfZjBQZPLQ/0?wx_fmt=png "版本配置")
****
* 配置不动，点击生成即可，会发现生成文件逐步完成
![生成](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYS1icpcckPtrk3tHj3wfibrcNXOLOz3sRVWxF34TLlYyJxNlwDKoBI4jA/0?wx_fmt=png "生成文件")
****
* 打开生成文件的项目
![打开生成文件的项目](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYkpUxzHgBjy7k0DcmVXkEGic5oWhiaBPb1YZXFhOOxcgAIeoJa1CfazRg/0?wx_fmt=png "打开生成文件的项目")
****
* 将`jsoncpp_dll`设置为`启动项`，并右键点击第一个`生成`即可完成静态文件的生成
![静态文件生成](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYG3ogN50qSibbnl2VWwonazExaDkWjJ0SSiceW3PMdibLoYxicAbcmVMm8g/0?wx_fmt=png "静态文件的生成")
****
* 可以在C盘新建一个存放环境的位置cpp
* 将下载`clone`的jsoncpp文件下的include文件全部放入C盘文件夹cpp中(此文件为jsoncpp的头文件)
* 将生成文件之后项目文件夹中`bin\Debug`下的`jsoncpp.dll`复制到C盘文件夹cpp下与`include同级`的`lib`文件夹(自己创建)下
* 将生成文件之后项目文件加中lib\Debug下的`jsoncpp.lib`复制到C盘cpp文件夹`lib`文件下下
![文件夹](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYrbJL3c8nVpqefTuicABAR2Fb76kFpySaiax770ic3z70AOhN3N5Wia1b2w/0?wx_fmt=png "文件夹存放")
****
#### VS2022环境配置
* 右键点击项目进入属性配置
![VS配置jsoncpp](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYcH0ia4wcLI7ozIMmXYqmicIylF3QPrPoylev5IXNial7FjWjX2T0PQLnA/0?wx_fmt=png "VS配置jsoncpp")

![VS配置jsoncpp](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYpUO34ZBQQ4RMDTcQflVoXgRQp6icfFDZSfPy8b2eMNz7SMyd7W4ibVdA/0?wx_fmt=png "VS配置jsoncpp")
****
#### 最后配置
> 运行下面的实例代码很可能出现“运行程序无法启动的问题”,将\(刚才C盘中)`jsoncpp.dll`文件复制到`main.cpp`对应的资源文件夹下即可
![VS运行最后配置](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbt72TibPibqUOnXvuG7dQ3WYYRjfdyU261d0YPzGD4Of8RlfQibjrGeLMKPliaicqhnKDv90PZVZuFfhFA/0?wx_fmt=png)

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
    Value root;
    root.append("luffy");
    root.append(19);
    root.append(170);
    root.append(false);


    Value subArray;
    subArray.append("ace");
    subArray.append("sabo");
    root.append(subArray);

    Value obj;
    obj["sex"] = "man";
    obj["girlfriend"] = "Hancock";

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
    ofstream ofs("test.json");
    ofs << json;
    ofs.close();
}

void readJson()
{
    //读
    ifstream ifs("test.json");
    Reader rd;
    Value root;
    rd.parse(ifs, root);
    if (root.isArray())
    {
        for (unsigned i = 0; i < root.size(); ++i)
        {
            Value item = root[i];
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

[^1]:https://cmake.org/download/