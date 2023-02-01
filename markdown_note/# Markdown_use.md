# Markdown

## 目录：
* Markdown简介
* Markdown语法
* Markdown高级技巧
* Markdown公众号插件
* 工具Typora使用及破解

### Markdown简介
 Markdown是一种轻量级的标记语言，语法简单，多数网站支持Markdown编译器。Markdown从写作到完成，导出格式随心所欲，同时也可导出HTML格式发布网站，也可导出PDF。
 #### 优点
 * 专注文本内容而不是排版样式，安心写作
 * 轻松导出HTML，PDF，和本身的.md文件
 * 纯文本内容，兼容所有文本编译器与字处理软件
 * 随时修改文章排版
 * 可读，直观，学习成本低
****
 ### Markdown语法
 #### 强调
 Markdown使用星号\* 和下划线 \_ 作为标记强调字词的符号
```markdown
* 强调
_ 强调
```

```biff
* 强调
_ 强调
```
***
#### 斜体
将内容用\*或\_包裹起来，包围的字词会被转换为表前包围，会显示斜体
```Markdown
*斜体内容*
_斜体内容_
```
*斜体内容*
_斜体内容_
##### 注意

* 用什么符号开启标签，就用什么符号结束
* \*和\_两边都有空白的话，它们就只会被当成普通的符号
* 如果要在文字前后直接插入普通的星号或下划线，可以用反斜杠\\*\\_
****
#### 粗体
用两个\*或\_包起来的话，则会被转成
```Markdown
**粗体内容**
__粗体内容__
```
**粗体内容**
__粗体内容__
****
#### 删除线
使用两个\~可以给内容加~~删除线~~
```Markdown
~~删除的内容~~
```
~~删除的内容~~
****
#### 标题
在首行插入1到6个\#(最多支持6级标题)对应标题1到6
```Markdown
# 标题1
## 标题2
#### 标题4
###### 标题6
```
# 标题1

## 标题2
#### 标题4
###### 标题6

****
#### 文本高亮

```markdown
`html` `css` `javascript`
```
`html` `css` `javascript`

#### 带下划线

```markdown
<u>带下划线文本</u>
```

<u>带下划线文本</u>

#### 注脚
```Markdown
注脚百度[^1]
[^1]:www.baidu.com   //放结尾，可链接可文字
```

注脚百度[^1]

****
#### Emoji图表

:100: :grinning:

#### 链接

##### 行内形式链接
行内形式的链接是在方括号后面接括号并插入链接即可，如果想要加上链接的alt体会文字，只要在网址后面，用双引号把alt文字包起来即可，其格式为 \[内容\]\(http_url "alt提示"\)
```Markdown
[内容](http_url "alt提示")
[foryouos](https://foryouos.github.io/ "foryouos页面")
```
[foryouos](https://foryouos.github.io/ "foryouos页面")

##### 参考形式链接
参考形式的链接使用另外一个方括号在连接文字的括号后面，而在第二个方括号里面填入要以辨识链接的标签：\(两个括号之前也可加空白\)
```Markdown
[内容][1]瓶子的跋涉[1]:https://foryouos.github.io/ "foryouos页面"
```
[foryouos][1] 瓶子的跋涉[1]:<https://foryouos.github.io/> "foryouos页面"
****
#### 图片

##### 行内形式图片
```Markdown
![Allt text](/path/to/img.jpg "optional title")
```
##### 参考形式图片
```markdown
![Alt text][id]
[id]:url/to/image "optional title attribute"
```

微信卡片阅读方式
```Markdown
[![Mark](https://files.mdnice.com/dance.gif)](https://mp.weixin.qq.com/s/lM808MxUu6tp8zU8SBu3sg)
```
[![Mark](https://files.mdnice.com/dance.gif)](https://foryouos.github.io/)

****
#### 列表

##### 无序列表
无序列表使用\*,\+或是-作为列表标记
```Markdown
* 香蕉
* 苹果
* 桃子
```
* 香蕉
* 苹果
* 桃子
##### 有序列表
```Markdown
1.第一天
2.第二天
3.第三天
```
1.第一天
2.第二天
3.第三天
##### 任务列表
任务列表的语法格式为 \-\[ \]todo,其中\[ \]\(带空格的中括号\)表示未完成的任务,\[X\]\(带字母x的中括号\)表示已经完成的任务
列表之前可以相互嵌套
```Markdown
- [x] 起床
- [x] 吃饭
- [x] 跑步
- [ ] 工作
```
- [x] 起床
- [x] 吃饭
- [x] 跑步
- [ ] 工作

##### 注意：
Markdown在下面这些符号前加上反斜杠来帮助插入普通的字符
```Markdown
	\   反斜杠
	`   反引号
	*   星号
	_   底线
	{}  大括号
	[]  方括号
	()  括号
	#   井字号
	+    加号
	-    减号
	.   英文句点
	!   惊叹号
```
****
#### 分隔线
在一行中用三个或以上的\*,\-,\_来创建一个分隔线，行内不能有其它东西，也可以在型号中间插入空白
****
#### 引用

```markdown
> 区块引用
>> 嵌套引用
>>> 三嵌套引用
>>>> 四嵌套引用
```

> 区块引用
>> 嵌套引用
>>> 三嵌套引用
>>>> 四嵌套引用
***
#### 表格

* 第一行包含表头并用"竖线"\(|)分隔
* 第二行将标题与单元格分开，并且必须包含三个或更多破折号
* 第三行以及随后的任何行均包含单元格值
##### 注意
* 不能在Markdown中将单元格分隔成多行，他们必须保持为单行，如果需要，还可以使用HTML \<br>标记对内容进行强制换行
* 第二行单元长短与标题不需要保持一致，但必须用竖线 \(|)分隔
* 可以有空白的单元格
###### 通过冒号指定每一列的文本对齐方式
* \:--: 两端都有冒号表示内容和标题栏居中对齐
* \:--- 左侧冒号表示内容和标题栏居左对齐
* \---: 右侧冒号表示内容和标签栏居右对齐
```Markdown
| 表头 | 表头 |
| ---- | ----|
|单元格 |单元格|
|单元格 | 单元格|
```
| 表头 | 表头 |
| ---- | :--:|
|单元格 |单元格|
|单元格 | 单元格|

****
### Markdown高级技巧
####支持的HTML元素
不在Markdown覆盖范围之内的标签，都可以直接在文档里面用HTML写
目前支持的HTML元素
```HTML
<kbd>
<b>
<i>
<em>
<sup>
<sub>
<br>
```
#### 公式
Markdown Preview Enhance使用  [Katex](https://github.com/Khan/KaTeX)和[MathJax](https://github.com/mathjax/MathJax)来渲染数学表达式。katex拥有比MathJax更快的性能，但缺少很多MathJax拥有的特性
```Markdown
$$
h_\theta(x)=\theta_0+\theta_1x
$$
```
$$
h_\theta(x)=\theta_0+\theta_1x
$$
##### 常见符号的代码
* 上下标，正负无穷
* 加减法，分式，根号，省略号
* 三角函数
* 矢量，累加累乘，极限
* 希腊字母

![上下标](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtx1lS5fMaqmNwV87NSnmSyFAIicUSDEg8BtRcHvw529VEAicocqicQzHbhE4ias87VjPlxIUa0zTtPDA/0?wx_fmt=png  "上下标正负无穷")

![加减法](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtx1lS5fMaqmNwV87NSnmSyeABopNmGheicskwtTEngzFezb4ianVXQOM59liaqUzQvibtJZehNSJMu7w/0?wx_fmt=png "加减法，分式根号，省略号")

![三角函数](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtx1lS5fMaqmNwV87NSnmSypvDGR56JHomIaahoCyOZJchmQ3TRa9c1J2mc2hMp4Piaia8g1EmpCMBA/0?wx_fmt=png "三角函数")

![矢量](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtx1lS5fMaqmNwV87NSnmSylLUOKPwpP4SEicY5OciclHZ166qLmVf7Dy5PrxOVicQHBps2sZYPThUtg/0?wx_fmt=png "矢量")

![希腊字母](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtx1lS5fMaqmNwV87NSnmSyBW5mZfyfD90IeZFp3IwJbDbFBNbUoMgLgS7X1SpSAT2AxMXQHNibIJQ/0?wx_fmt=png "希腊字母")
****
#### 矩阵
##### 简单矩阵
使用\\begin\{matrix}...\\end\{matrix}生成,每一行以\\\结尾表示换行，元素间以&间隔，式子的表示序号\\tag\{1}\(右边的序号)
- matrix 矩阵
- Bmatrix 大括号
- bmatrix 中括号
- vmatrix 行列式
- 
```markdown
$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 
\end{matrix} \tag{1}
$$
```
$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 
\end{matrix} \tag{1}
$$
##### 带左右括号的矩阵(大众小括号)
在\\begin{}之前和\end{}之后添加左右括号代码
****
```markdown
$$
\left\{
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 
\end{matrix}
\right\} \tag{1}
$$
```
$$
\left\{
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 
\end{matrix}
\right\} \tag{1}
$$
****
改变\\begin\{matrix}和\end\{maatrix}中的matrix
```markdown
$$
\begin{bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 
\end{bmatrix} \tag{1}
$$
```
$$
\begin{bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9 
\end{bmatrix} \tag{1}
$$
****
##### 包含希腊字母与省略号

行省略号\\cdots,列省略号\vdots,斜向省略号(左上至右下)\\ddots
```Markdown
$$
\begin{Bmatrix}
1         & 2          &\cdots        & 5 \\
6         & 7          & \vdots       & 10 \\
\vdots    &\vdots      & \ddots       & \vdots \\
\alpha    & \alpha+1   & \cdots       & \alpha+4 
\end{Bmatrix}
$$
```
$$
\begin{Bmatrix}
1         & 2          &\cdots        & 5 \\
6         & 7          & \vdots       & 10 \\
\vdots    &\vdots      & \ddots       & \vdots \\
\alpha    & \alpha+1   & \cdots       & \alpha+4 
\end{Bmatrix}
$$

****
#### 表格
```markdown
$$
\begin{array}{|c|c|c|}
            \hline 2&9&4\\
            \hline 7&5&3\\
            \hline 6&1&8\\
            \hline
\end{array}
$$
```
$$
\begin{array}{|c|c|c|}
            \hline 2&9&4\\
            \hline 7&5&3\\
            \hline 6&1&8\\
            \hline
\end{array}
$$
##### 注意
* 开头结尾：\\begin{array},\\end{array}
* 定义式：例{|c|c|c|},其中c l r分别代表居中，左对齐，右对齐
* 分隔线：
	- 竖直分割线：在定义式中插入 \|,\(\||代表两条竖直分割线)
	- 水平分隔线：在下一行输入前插入\\hline，
	- 其它：每行元素均要插入 \&,每行元素以\\\结尾
****
#### 真值表
```Markdown 
$$
\begin{aligned}
a &= b + c \\
  &= d + e + f
\end{aligned}
$$
```
$$
\begin{aligned}
a &= b + c \\
  &= d + e + f
\end{aligned}
$$

#### 多行等式对齐

```Markdown
$$
\begin{cases}
3x + 5y +  z \\
7x - 2y + 4z \\
-6x + 3y + 2z
\end{cases}
$$

```
$$
\begin{cases}
3x + 5y +  z \\
7x - 2y + 4z \\
-6x + 3y + 2z
\end{cases}
$$



#### 方程组，条件表达式

```Markdown
$$
f(n)=
\begin{cases}
n/2, & \text{if }n\text{ is even}\\
3n+1, & \text{if }n\text{ is odd}
\end{cases}
$$
```
$$
f(n)=
\begin{cases}
n/2, & \text{if }n\text{ is even}\\
3n+1, & \text{if }n\text{ is odd}
\end{cases}
$$


#### 间隔
* 紧贴 \\!
* 无空格 小空格\\,中空格\\;打空格\\
* 真空格\\quad 双真空格\\qquad 
****
### Typora

[Typora Markdown编辑器](https://typoraio.cn/ "Typora Markdown编辑器")

```tiff
https://typoraio.cn/
```

### typora1.4.8激活工具

[蓝奏云](https://foryouos.lanzoul.com/ic2dX0mc3nbc "typoral激活工具蓝奏云盘")[^2]

[百度网盘](https://pan.baidu.com/s/1uELB0Y7-I9lD9N8jWmwgRg?pwd=fh9w "typoral激活工具百度网盘")[^3]😊

****

[^1]:www.baidu.com
[^2]:https://foryouos.lanzoul.com/ic2dX0mc3nbc
[^3]:https://pan.baidu.com/s/1uELB0Y7-I9lD9N8jWmwgRg?pwd=fh9w

****

参考资料：

* [Typora](https://zhuanlan.zhihu.com/p/261750408 "typora")

* [技能树](https://edu.csdn.net/skill/markdown)
* [NoteZ个人博客](http://chart.zhenglinglu.cn/pages/3ae8de/)

