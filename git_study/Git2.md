### Git

#### 删除分支

```sh
git branch -d <分支名>
```
#### 修改分支名

```sh
git branch -m <原分支名> <新分支名>
```
#### 忽略文件

添加一个名为.gitignore的文件，列出要忽略的文件的模式

```sh
*.[oa] #忽略以.o或.a结尾的文件(一般这类文件在编译过程出现)
*~     #忽略以~ 结尾的文件(一般是文本编辑软件保存的副本)
```
##### .gitignore文件格式规范

* 所有空行或者以 **#** 开头都会被Git忽略(注释符号)
* 可以使用标准的glob模式（shell所使用的简化正则）匹配，它会递归整个工作区
* 匹配模式可以以(**/**)开头防止递归
* 以  \(**/**)  结尾指定目录
* 要忽略指定模式以外的文件或目录，可以在模式前加上叹号(**!**)取反
* 








### 参考资料

* [GiT-book中文](https://git-scm.com/book/zh/v2)[^1]
* [Git笔记](http://chart.zhenglinglu.cn/pages/635088/)[^2]

[^1]:https://git-scm.com/book/zh/v2
[^2]:http://chart.zhenglinglu.cn/pages/635088/