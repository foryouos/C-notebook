###  Linux基础命令1关机文件类操作

> Linux主要是通过命令行来进行运行管理的操作系统

#### 比图像化Windows的优点：

- 快速
- 批量化
- 自动化
- 智能化的处理业务

进入root权限(首次）

```sh
sudo passwd   // 首次需要设置密码//输入密码su   // 就如root用户权限
```
#### 常见关机，重启和注销指令:

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbswkxBUQxd5lWI8gV177foWAR72moHpkVYfY9v5BFrUeCOlAQNb1BJoXFVtjvksEt0tt576xSy9YA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### Linux各文件夹含义及用途：

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbswkxBUQxd5lWI8gV177foWUnglAOpHTduT98iaInccOFOrQbXCbjK5ibZNYKIricc8TmrG0icWWMuwjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsYHvxfYOdPtq8846ywUeha8qLyoe9m5egrfmiaQzaoTQ6oShC5AjGbEZsyjFR1pTeia2q3KKZVBWZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1. /boot 该目录默认存放Linux的启动文件和内核，包含可引导的Linux内核和引导装载（boot loader）配置文件（GRUB）
2. /initrd : boot loader initialized RAM disk就是boot loader初始化的内存盘。在Linux内核启动前，boot loader会将存储介质（一般是硬盘）中的initrd文件加载到内存，内核启动时会在访问真正的根文件系统前先访问该内存中的initrd文件系统
3. /bin 存放linux的常用命令，例如ls,sort,date和chmod，
4. /sbin 存放 系统管理员使用的管理程序，包含管理命令和守护进程
5. /var 存放经常被 修改的文件，包括各种日志，数据数据文件。这里放置作为FTP服务器（/var/ftp），web服务器（/var/www）共享文件，它还包含所有系统日志文件(/var/log)
6. /etc 存放系统管理 时要 用到的各种配置文件和子目录，例如：网络配置文件，文件系统，x系统配置文件 ，设备配置信息，设置用户信息
7. /dev 包含了Linux系统中使用的所有外部设备，它实际上是访问这些外部设备的端口，访问这些外部设备与访问一个文件或一个目录没有区别
8. /mnt 临时将别的文件系统挂在该目录下
9. /root 超级用户的主目录
10. /home 如果创建一个名为“XX”的用户，那么在/home目录下就有一个对应的“/home/xx”路径，用来存放该目录的主目录
11. /usr 用户的应用程序和文件几乎都存放在该目录下
12. /lib 存放系统动态链接共享库，几乎所有应用程序都会 用到该目录下的共享库
13. /opt 第三方软件在安装时默认安装目录（警惕删除）
14. /tmp 用来存放不同程序执行时产生的临时文件，该目录会被系统自动清理干净
15. /proc 可以在该目录下获取系统信息 ，这些信息是在内存中由系统自己产生的，该目录的内容不在硬件上而在内存里
16. /misc 可以让多用户堆积和临时转移自己的文件
17. /lost+found 该目录在大多数情况下都是空的，当突然停电，或者非正常关机后，有些文件就临时存放在这里
18. 文件颜色的含义：蓝色为文件夹，绿色为可执行文件，浅蓝色是链接文件，红框文件是加了SUID位，任意限权；红色为压缩文件，褐色为设备文件。

#### 文件和目录操作命令

- pwd（print working directory）::显示当前工作目录的绝对路径,查看当前所在路径

```
pwd [option]
pwd [选项]
// 必须有空格，通常情况下不需要带任何参数
pwd -L
//logical的首字符缩写，显示逻辑路径（忽略软链接文件）
pwd -P
// physical首字符缩写，显示物理路径，
//如果是软链接文件，则会显示软链接文件对应的源文件
```

- cd（change directory） 切换目录

```sh
//语法格式
cd [option] [dir]
cd [选项] [目录]
//默认情况下，单独执行cd命令，可切换到当前登陆用户的家目录
//（由系统环境变量HOME定义）
help cd  //查看系统帮助

//用法
cd -  //切换到上一次所在的目录路径
cd ~  //切换到当前用户的家目录所在的路径
cd ..  //切换到当前目录的上一级目录所在的路径，一个点为当前目录
cd -P  //若切换的目标目录hi软链接，则直接指向真正的物理目标目录
cd -L  //直接切换啊软链接所在的目录
```

#### TAb键具有自动补齐功能

注意:

```sh
绝对路径从 “/” 根开始的路径，如: /data/
相对路径不从斜线开始   如: data/
//进入当前目录的父目录的父目录
cd ../../
```
#### tree
- tree:以树形结构显示目录下的所有内容，包括所有文件，子目录及子目录里的目录和文件
```sh
tree [option] [directory]
tree [选项] [目录]
//tree命令后若不接选项和目录就会默认显示当前所在路径目录的目录结构
```

| 参数选项 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| -a       | 显示所有文件，包含隐藏文件（以“.”点开头的文件）              |
| -d       | 只显示目录                                                   |
| -f       | 显示每个文件的全路径                                         |
| -i       | 不显示树枝，常与-f参数配合使用                               |
| -L level | 遍历目录的最大层数，level为大于0的正整数                     |
| -F       | 在执行文件，目录，Socket,符号连接，管道名称等不同类型文件的结尾，各自加上"*","/","=","@","\|"号，类似ls命令的-F选项 |

```sh
tree    //显示当前目录的结构
tree -a   //以树形结构显示目录下的所有内容
tree -L 1   //-L参数后接数字，表示查看目录的层数，不带-L选项默认显示所有层数
tree -d /etc/    // -d参数表示只显示目录
tree -L 1 -f /boot/   //-f显示内容的完整路
tree -L 1 -F /boot/    //使用-F参数会在幕后后面添加“/”，方便区分目录
```
#### mkdir
> mkdir （make directories）创建目录,默认情况下，如果要创建的目录已存在，则会提示此文件已存在，而不会继续创建目录
```sh
mkdir [option] [directory]
mkdir [选项] [目录]
// mkdir 命令可以同时创建多个目录，格式为mkdir，dir1，dir2
```

#### mkdir使用及说明

| 参数选项 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| -p       | 1，递归创建目录，递归：父目录及其子目录及其子目录及其子目录的子目录2，即使要创建的目录事先已存在也不会报错提示目录已存在 |
| -m       | 设置新创建目录的默认目录对应的权限                           |
| -v       | 显示创建目录的过程                                           |

```sh
mkdir data  //在当前目录下创建data目录//再此执行创建命令会提示目录已经存在
```
> 注意：window下的目录路径样式为D:\data\test,而Linux样式的路径样式为/data/test,它们的目录顶点和分隔符均不同

```sh
mkdir -m 333 dir2
// 创建目录时指定333的数字权限
ls -ld dir2
//可以看到权限已经发生变化
```

#### touch命令:

- 创建新的空文件
- 改变已有文件的时间戳属性

```sh
touch [option] [file]
touch [选项] [文件]
```

注意；touch是创建新文件，medir是创建空目录，touch虽不能创建目录，但是可以修改目录的时间戳

| 参数选项  | 解释说明                                                     |
| --------- | ------------------------------------------------------------ |
| -a        | 只更改指定文件的最后访问时间                                 |
| -d STRING | 使用字符串STRING代表的时间作为模板设置指定文件的时间属性     |
| -m        | 只更改指定文件的最后修改时间                                 |
| -r file   | 将特定文件的时间属性设置为与模板文件file时间属性相同         |
| -t STAMP  | 使用[[CC]YY]MMDDhhmm[.ss]格式的时间设置文件的时间属性，格式的含义从左到右依次为：世纪，年，月，日，时，分，秒 |

使用:

```
mkdir /test    //在根下新建test目录
cd /test/       //切换到此目录
touch oldboy.txt   //创建空文件oldboy.txt
ls                //查看文件表的文件
touch a.txt b.txt   //同时创建多个文件
touch stu{01..05}    //利用{}输出的字符序列批量创建文件
stat oldboy.txt      //查看文件的时间戳属性
touch -a oldboy.txt   // 更改最后访时间
touch -m oldboy.txt    //更改最后修改时间
ls -lh oldboy.txt      //修改文件修改时间
touch -d 20202001 oldboy.txt   // 指定创建文件后的修改时间
touch -r a.txt oldboy.txt     //让oldboy.txt的时间属性和a.txt一致
```

#### GNU/Linux
> 文件有三种类型的时间戳
```sh
Access   最后访问文件的时间
Modify   最后修改文件的时间
Change   最后改变文件状态的时间
```
```sh
ls -lt  // 最后修改时间
ls -lc  // 状态改变时间
ls -lu  //最后访问时间
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsYHvxfYOdPtq8846ywUehaR2Gl4JASCZRNw4vGB5D7LyJg9OI35YpvHwu0R3RePn6utl2sBz3UZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsYHvxfYOdPtq8846ywUehaCRArCXCTrhxdib1PtzUhR0Pfe2tI4Ipvfu9ibbKVjJbtUoDKI77RoajA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
#### ls
> ls ：列出目录的内容及其内容属性信息（list directory contents）

```
ls [option] [file]
ls [选项] [<文件或目录>]
// 命令后面的选项和目录文件可以省略，表示 查看当前路径的文件信息
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsYHvxfYOdPtq8846ywUehaOZzAiaUUhtadGHJx86WRkK78tWKBQOunIV82DpPAcCgTJiaQic1U0HSNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```sh
touch .file4.txt   //再创建一个隐藏文件
//在linux系统中以“.”点开头的文件就是隐藏文件
ls -l --time-style=long-iso //以long-iso方式显示时间，当遇到目录时间显示不一致 s时
ls --full-time //用于显示完整的时间，等同于
ls -l --time-style=full-iso
date  //显示当前系统时间
```

#### cp
> 复制文件或目录
```
cp [option] [source] [dest]
cp [选项] [源文件] [目标文件]
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtPJDhG2drZFicBoHEoSN6I5fm7sSn2FAib4E68UcCNl4nLMoVuxYZZvyGkULZI5PujaJeyO2KbQXVw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
cp file1.txt file2.txt    //复制file1.txt为file4.txt
cp -i file1.txt file2.txt  //若出现重复，会提示是否覆盖
//备份操作
cp /etc/ssh/sshd_config{,.ori}
//对大括号的展开操作  /etc/ssh/sshd_config{,.ori}展开成/etc/ssh/sshd_config /etc/ssh/sshd_config.ori 再传给cp命令
```
#### mv: （move）移动或重命名文件

```sh
mv [option] [source] [dest]
mv [选项] [源文件] [目标文件]
```

| 参数选项 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| -f       | 若目标文件已经存在，则不会询问而是直接覆盖                   |
| -i       | 若目标文件已经存在，则会询问是否覆盖                         |
| -n       | 不覆盖已经存在的文件                                         |
| -t       | 指定mv的目标文件，适用于移动多个源文件到一个目录的情况，此时目标文件在前，源文件在后，和cp命令的-t选项功能一致 |
| -u       | 在源文件比目标文件新，或目标文件不存在时才进行移动           |

```
mv file6.txt file7.txt //若file7.txt不存在则将file6.txt重命名为file7.txt
//若file7.txt存在，则将file5.txt覆盖为file7.txt
mv file.txt dir1/  
//dir1位目录且存在，则移动file.txt到dir1下，若dir1不存在，则重命名为dir1的普通文件
```
#### rm
> rm：删除一个多多个文件或目录（remove files or directories）
```
rm [option] [file]rm [选项] [<文件或目录>]
```

| 参数选项 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| -f       | 强制删除。忽略不存在的文件，不提示确认                       |
| -i       | 在删除前需要确认                                             |
| -I       | 在删除超过三个文件或者递归删除前要求确认（一个大文件下小文件一起删除） |
| -r       | 递归删除目录及其内容                                         |
```sh
rm -f ./*     //加上“./”
//禁止使用rm -fr /oldboy/* ,这个命令如果多了空格可能会带来在灾难
rm -fr /oldboy/*
// "*"的前面不小心多了空格，会删除当前目录的所有内容，
rm -fr /oldboy/ *    //把当前目录（根）下面的目录全部删除
```
#### rmdir
> rmdir ：用于删除空目录(remove empty directories)，当目录不为空时，命令不起作用。
```sh
rmdir [option] [directory]
rmdir [选项] [目录]
```

| 参数选项 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| -p       | 递归删除目录，当子目录删除后其父目录为空时，也一并删除。如果整个路径被删除，或者由于某些原因保留了部分路径，则系统在标准输出上显示对应的信息 |
| -v       | 显示命令的执行过程                                           |

#### In硬链接与软连接

> 功能:创建文件间的链接（make links between files）

```
ln [option] [source] [target]
ln [选项] [源文件或目录] [目标文件或目录]
```

| 参数选项 | 解释说明             |
| -------- | -------------------- |
| 无参数   | 创建硬链接           |
| -s       | 创建软连接(符号链接) |

硬链接：通过索引结点（Inode）来进行链接。在Linux文件系统中，所有文件都有一个独有的inode编号。多个文件指向同一索引结点（inode），硬链接相当于文件的另外一个入口
```
ln /etc/hosts hard_link
//给/etc/hosts 文件做硬链接
//通过ls查看硬链接数值
ls -i /etc/hosts hard_link
rm -f /etc/hosts  //删除源文件
ln hard_link /etc/host   //再次链接回来
```
##### 硬链接：

- 具有相同inode节点号的多个文件互为硬链接文件
- 删除硬链接文件或者删除源文件任意之一，文件实体并未被删除
- 只有删除了源文件以及源文件所对应的硬链接文件，文件实体才会被删除
- 硬链接文件就是文件的另外一个入口
- 可以通过设置硬链接文件，来防止重要文件被误删
- 硬链接文件可以用rm命令删除
- 对于静态文件（没有进程正在调用的文件）来讲，当对应硬链接数为0（i_link）时，文件就会被删除，i_link的查看方法是ls-lih,
- 很多硬件设备的快照功能，就是利用了硬链接



###### 软链接
> 类似windows里的快捷方式，软链接是真正的链接文件

- 软链接文件inode值和源文件，硬链接文件都不同
- 软链接文件的文件类型是l （字母l）
- 软链接类似一个文本文件，里面存放的是源文件的路径，指向源文件实体
- 即使删除了源文件，软链接文件也依然存在，但是无法访问指向的源文件路径内容
- 失效的时候一般白字红底闪烁提示

注意：

- 对于目录，不可以创建硬链接，但是可以创建软连接
- 目录的硬链接文件不能跨越文件系统（硬链接需要相同的inode值）
#### readlink
> readlink :查看符号链接文件的真实内容
```sh
readlink [option] [file]readlink [选项] [文件]
```

| 参数选项 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| -f       | 一直跟随符号链接，直到非符号链接的文件位置，但要保证最后必须存在一个非符号链接的文件 |