# Git2

#### Git

**删除分支**

```sh
git branch -d <分支名>
```

**修改分支名**

```sh
git branch -m <原分支名> <新分支名>
```

***

**查看简明状态说明**

* git目录中的文件状态包含:是否跟踪，是否修改，是否已存入暂存区
* 参数的一个横杠表示缩写，两个横杠表示全程

```sh
git status -s # 查看简明状态说明
#
 M README # 已修改，但未暂存(M的位置靠右，红色)
 MM Rakefile # 已修改，暂存后又作了修改（有暂存和未暂存)
 A lib/git.rb # 新添加到暂存区，未提交
 M lib/simplegit.rb #已修改，已暂存
 ？？ LICENSE.txt #新添加，未跟踪
```

**忽略文件**

添加一个名为`.gitignore`的文件，列出要忽略的文件的模式

```sh
*.[oa] #忽略以.o或.a结尾的文件(一般这类文件在编译过程出现)
*~     #忽略以~ 结尾的文件(一般是文本编辑软件保存的副本)
```

**`.gitignore`文件格式规范**

* 所有空行或者以 **#** 开头都会被Git忽略(注释符号)
* 可以使用标准的glob模式（shell所使用的简化正则）匹配，它会递归整个工作区
* 匹配模式可以以(**/**)开头防止递归
* 以 (**/**) 结尾指定目录
* 要忽略指定模式以外的文件或目录，可以在模式前加上叹号(**!**)取反
* 星号(\*)匹配零个或多个任意字符
* \[`abc`]匹配任何一个列在括号中的字符
* 文件(**？**)只匹配一个任意一个字符
* \[0-9]表示匹配所有0到9的数字，在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配
* 使用两个(\*\*\*\***)表示匹配任意中间目录，比如a/**/z可以匹配a/z,a/b/z,a/b/c/z等
* 详情

```sh
*.a   # 忽略所有的.a文件
!lib.a   # 跟踪所有的lib.a，基表你在前面忽略了.a文件
/TODO   # 只忽略当前目录下的TODO文件，而不忽略subdir/TODO
build/  # 忽略任何目录下名为build的文件夹
doc/*.txt  # 忽略doc/notes.txt 但不忽略深层次的
doc/**/*.pdf  # 忽略doc/ 目录及其所有子目录下的.pdf 文件
```

***

**查看修改的具体内容**

```sh
git diff # 比较修改之前还没有暂存起来的变化内容
git diff --staged # 查看已暂存的将要添加到下次提交里的内容
```

`注意`:`git status`只能查看文件变化的状态，并不能查看具体修改了哪些内容。使用`git diff`可以查看具体变化的内容 ![git diff](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbsY0YabUMSXt2xbpITduhdCdE1VrpZsiabp9K5nViaB8ugo6JqPXqL0VxXFzc2Z7kBBpa8yNQUicP0lg/0?wx\_fmt=png)

***

**查看提交历史**

不传入任何参数的默认情况先，`git log`会按时间先后顺序列出所有的提交，最新的更新排在最上面

```sh
git log
git log -p -2 # -p显示差异，-2显示最近的提交次数
git log --stat # 显示每次提交的差异统计
#使用不同的默认格式展示提交历史
#oneline把每个提交放在一行显示，还有short，full，fuller
git log --pretty=oneline  
# --pretty=format:"%h - %an,%ar:%s"定制记录的显示格式
git log --pretty=format:"%h - %an,%ar:%s"

```

![git log](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbsY0YabUMSXt2xbpITduhdCK3V2TY33PTgJhlsYSgYfHGV4FcqynIOWknRnXUVxD9MM0RcxicRvxUA/0?wx\_fmt=png) 常见的format选项

***

**撤销操作**

```sh
git commit --amend  #重新提交，且只有一次提交记录
```

**远程仓库中抓取与拉取**

```sh
get fetch <remote>
```

此命令只会将数据下载到本地仓库--它并不会自动合并或者修改你当前的工作，必须手动将其合并

```sh
git pull
```

此命令会自动抓取后合并该远程分支到当前分支 默认情况下，`git clone`会自动设置本地master分支跟踪克隆的远程仓库的`master`分支(或其它名字的默认分支)。运行`git pull`通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支

***

**打标签**

```sh
git tag  #完整标签列表
git tag -l "v2.0"  # 只显示包含v2.0的标签。注意加星号(*)
git tag -list "v2.0"
```

* 轻量标签:本质上是将提交和存储到一个文件中---没有保存任何其它信息

```sh
git tag v1.4-lw  # 不需要添加选项
git show v1.4-lw  #查看标签信息，没有任何额外信息
```

* 附注标签:是存储在Git数据库中的一个完整对象，它们是可以被校验的，其中包含打标签者的名字，电子邮箱地址，日期时间，此外还有一个标签信息，并且可以使用GNU Privacy Guard签名并验证。通常会建议创建附注标签。

```sh
git tag -a v1.4 -m "my version 1.4" # -a表示add，-m表示附件信息
```

**共享标签**

git push命令并不会传送标签到远程仓库服务器上，在创建完标签后必须显式地推送标签到贡献服务器上，这个过程就像共享远程分支一样

```sh
git push origin v1.5 # 显式地推送标签到远程仓库
git push origin --tags #一次性推送所有不在远程仓库上的标签
```

**删除标签**

```sh
git tag -d v1.4-lw  # 删除一个轻量标签，但并不会从远程仓库移除
git push <remote> :refs/tags/v1.4-lw  #更新远程仓库
# 第二行或者
git push <remote> --delete <tagname>
```

**命令别名**

```sh
 git config --global alias.co checkout
 git config --global alias.br branch
 git config --global alias.ci commit
 git config --global alias.st status

```

***

**修改分支名**

首先保证本地代码是最新的

```sh
git branch -m oldbranchName newbranchname #修改本地分支名
git push origin :oldbranchName  #删除远程分支
git push origin --delete oldbranchName  #或者删除远程分支

#改名后的本地分支推送到远程
git push --set-uostream origin newbranchname
```

***

* Workspace ：工作区
* Index/Stage：暂存区
* Repository:仓库区(或本地仓库)
* Remote：远程仓库 ![Git命令](https://mmbiz.qpic.cn/mmbiz\_png/ORog4TEnkbsY0YabUMSXt2xbpITduhdCmRbmErzicx7hkKfAXmOeNB9dOthFSKeicnDwLhzY23DpDaQkZIZbjvNQ/0?wx\_fmt=png)

***

#### 改名文件并将这个改名放入暂存区

```sh
git mv [file-original] [file-renamed]
```

***

#### 参考资料

* **GiT-book中文**
* Git笔记

1. http://chart.zhenglinglu.cn/pages/635088/
2. https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2#pretty\_format
