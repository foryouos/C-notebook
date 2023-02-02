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



~~~md
::: 
```yaml
- name: 《静夜思》
  desc: 床前明月光，疑是地上霜。举头望明月，低头思故乡。
  bgColor: '#F0DFB1'
  textColor: '#242A38'
- name: Vdoing
  desc: 🚀一款简洁高效的VuePress 知识管理&博客(blog) 主题
  link: https://github.com/xugaoyi/vuepress-theme-vdoing
  bgColor: '#DFEEE7'
  textColor: '#2A3344'
```
~~~





### 参考资料

* [GiT-book中文](https://git-scm.com/book/zh/v2)[^1]
* [Git笔记](http://chart.zhenglinglu.cn/pages/635088/)[^2]

[^1]:https://git-scm.com/book/zh/v2
[^2]:http://chart.zhenglinglu.cn/pages/635088/