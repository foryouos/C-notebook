# Linux基础2
## Linux全局
> Linux中一切皆文件
![Linux运行全局](https://i.imgtg.com/2023/04/19/uKFaD.jpg)

## 查看指令所在的位置
```sh
which pwd
```
## 查看环境变量
```sh
[root@ecs-39448704 ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

## 命令行补充

| 快捷键           |           功能            |                    备注                    |
| ---------------- | :-----------------------: | :----------------------------------------: |
| Ctrl+p           | 显示输入的上一个历史命令  | 从输入的最后一个命令往前倒，也可以使用 ↑键 |
| Ctrl+n           | 显示输入的下一个历史命令  |               也可以使用 ↓键               |
| Ctrl+a           |    光标移动命命令行首     |             也可以使用 Home 键             |
| Ctrl+e           |    光标移动命命令行尾     |             也可以使用 End 键              |
| Ctrl+u           |  删除光标前的部分字符串   |                                            |
| Ctrl+k           |  删除光标后的部分字符串   |                                            |
| Backspace/Delete | 删除光标前 / 后的一个字符 |                                            |

## 双目录

```sh
[root@ecs-39448704 ~]# cd ../../
[root@ecs-39448704 /]# ls
bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  patch  proc  root  run  sbin  srv  sys  titan  tmp  usr  var  www
```

## 输出文件

```sh
//ll输出文件信息， -F是否还有子目录加/，h文件以文件大小形式输出，a总大小
[root@ecs-39448704 /]# ll -Fha
total 84K
```
## 文件类型

* -: 普通的文件，在 Linux 终端中`没有`执行权限的为`白色`，`压缩包`为`红色`，`可执行程序`为`绿色`字体
* d:` 目录` (directory), 在 Linux 终端中为`蓝色`字体，如果目录的所有权限都是开放的，有绿色的背景色
* l: `软链接`文件 (link), 相当于 windows 中的`快捷方式`，在 Linux 终端中为`淡蓝色` (青色) 字体
* c:` 字符设备` (char), 在 Linux 终端中为`黄色`字体
* b: `块`设备 (block), 在 Linux 终端中为`黄色`体
* p: `管道`文件 (pipe), 在 Linux 终端中为`棕黄色`字体
* s: 本地`套接字文件` (socket), 在 Linux 终端中为`粉色`字体
## 用户类型
> 文件类型包含:`文件所有者`，`文件所属组用户`，`其他人`
> `r `读 `w `写` x `执行-execute 
> 没有权限  `-`

* 第一个字母为文件类型 
* 第一个文件所有者，第二个为文件所属组用户，第三为其他人
```sh
[root@ecs-39448704 /]# ll
total 72
            别名个数 所有权用户  目录磁盘空间大小(不代表内部)
lrwxrwxrwx.   1 root root     7 Dec 18  2018 bin -> usr/bin
dr-xr-xr-x.   5 root root  4096 Jun 28  2022 boot
drwxr-xr-x   20 root root  3140 Apr 11 07:45 dev

```
## 删除
> rm：可以直接删除文件
> rm -r   删除目录，参数-r是递归(recursion)
> -i 删除时给提示
> -f 删除时没有任何提示

## 拷贝
> cp对`文件`可以直接拷贝
> `目录`拷贝要加参数 `-r`

## 修改文件权限

### 代码法
> chmod

```sh
chmod who [+|-|=] mod 文件名
who: 
	- u :user ->文件所有者
	- g ：group ->文件所属组用户
	- o: other ->其他
	- a: all，以上三类人 u+g+o
mod:权限
	r: 读
	w: 写
	x：执行
	- : 没有权限
对权限的操作
	+:添加权限
	-:去除权限
	=:覆盖权限
```
### 数字法
```sh
chmod [+|-|=] mod 文件名
- mod 权限描述 所有权限都放开时 7
- 4 read ,r
- 2 write,w
- 1 execute ,x
- 0:没有权限

#分解:chmod 0567 a.txt
0：八进制
5：文件所有者 r+x
6: 文件所属用户  r + w
7 其他人 r+w+x
```
### 修改文件所有者
```sh
sudo chown 新的所有者:新的组名 文件名
```
### 修改文件所属组
> 只修改文件所属的组，普通用户没有这个权限，借助管理与安全线
```sh
sudo chgrp 新的组 文件名
```

## 创建文件
```sh
mkdir 创建目录
touch 文件 //创建文件，存在更新时间，不存在创建
cat 文件   //查看文件内容
which 	//查看命令所在位置
```
## 重定向
> 将内容快速叠加到文件当中，修改输出数据的文字
> `>`将文件内容写入到指定文件中，如果文件中已有数据，则会使用新数据覆盖原数据
> `>>`将输出的内容追加到指定的文件尾部

## 添加新用户
```sh
//第一种
sudo adduser 用户名

//第二种
//cenos
sudo useradd 用户名
//ubuntu  
sudo useradd -m -s /bin/bash 用户名

```
## 删除用户
> 使用userdle命令删除系统中用户ID和所属组ID等相关信息，但是在某些Linux版本中虽然被删除了，但是家目录却没有被删除，需要手动将其删除

```sh
//-r 一并把用户的家目录页删除，不加不删除
sudo userdel 用户名 -r
```
## 添加删除用户组
> 使用groupadd添加用户组，使用groupdel删除用户组
> 验证成功，查看`/etc/group`文件，里面用户相关信息以及用户组ID

## 修改密码
```sh
passwd :修改当前用户
//修改非当前用户密码
sudo passwd 用户名
```

## 压缩命令
### 压缩tar
```sh
x:释放压缩文件内容，解压
c:创建压缩文件
z:使用gzip方式进行文件压缩.tar.gz
j：使用bzip2的方式进行文件压缩.tar.bz2
v：压缩过程中显示压缩信息，可以省略不写
f:指定压缩包的名字

# 语法
tar 参数 生成压缩包的名字 要压缩的文件(文件或目录)

# 压缩名字
	- 压缩使用gzip方式，标准后缀为.tar.gz
	- 压缩使用bzip2方式，标准后缀为.tar.bz2
	- 

```
* tar 解压
```sh
tar 参数 压缩包名 -c 解压目录
tar xzv 名称 -c 目录 
```
* zip 解压
> 使用zip压缩目录，必须添加参数`-r`，这样才能将子目录中的文件一并压缩。zi压缩会 自动生成文件后缀.zip

```cpp
zip [-r] 压缩包名 要压缩的文件
```

* rar
> 安装

```sh
wget https://www.rarlab.com/rar/rarlinux-x64-6.0.0.tar.gz
tar -zxvf rarlinux-x64-6.0.0.tar.gz 
mv ./rar /opt
ln -s /opt/rar/rar /usr/local/bin/rar
ln -s /opt/rar/unrar /usr/local/bin/unrar
```
> 使用，文件压缩，需要使用参数a ，压缩包会自动添加后缀.rar
> 如果压缩了目录，需要加参数-r
```sh
rar a 压缩包名 要压缩的文件 [-r]
```
> 解压缩

```sh
#解压参数x，默认加压到当前目录中
rar/unrar x 压缩包名字
# 解压到指定目录
rar/unrar x 压缩包姓名 解压目录
```
## xz文件
* 压缩

> 创建文件，首先将需要压缩的文件大宝`tar cvf xxx.tar files`然后对大宝文件进行压缩`xz -z xxx.tar`。这样就可以得到一个大包之后的压缩文件
> 使用`xz`工具压缩文件需要添加参数`-z`

* 解压缩
```sh
# 压缩包解压缩，得到xxx.tar
xz -d xxx.tar.xz
# 第二步，将xxx.tar中的文件释放到当前目录
tar xvf.xxx.tar
```
## 文件搜索
### find
> 主要功能时根据文件的属性，查找对应的磁盘文件\(`文件名`，`文件类型`，`文件大小`，`文件的目录深度`)。模糊查询必须要使用对应的通配符，`*`和`？`,其中`*` 可以匹配 `零`个或者`多`个字符`？`用于匹配`单`个字符。
*  `-name`
```sh
# 语法格式:根据文件名搜搜
find 搜索的路径 -name 要搜索的文件名
```

* `-type`

|    文件类型    | 类型的字符描述 |
| :------------: | :------------: |
| 本地套接字类型 |       s        |
|  普通文件类型  |       f        |
|    目录类型    |       d        |
|   软连接类型   |       l        |
|  字符设备类型  |       c        |
|   块设备类型   |       b        |
|    管道类型    |       p        |

```sh
#语法格式:
find 搜索的路径 -type 文件类型
```
* `-size`
> 根据文件大小进行搜搜，需要使用参数`-size`常见的分别有K(小写)，M(大写)，G(大写)。
```sh
#找到大约4M的
[root@ecs-39448704 bin]# find ./ -size +4M
./cpack
./gdb
./ld.gold
./ctest
```
* 基于目录层级
> `-maxdepth`：最多搜索到第多少层目录
> `-mindepth`：至少从第几层开始搜索

```sh
sudo find / -maxdepth 5 -name "*.txt"
```
* exec  (`ok`与此相同)
> exec是find的参数，可以在exec参数后添加其他需要执行的shell命令
> find添加了exec参数之后，命令的`尾`部需要添加一个后缀`{} \;`,注`{}`和`\`中间有`空格`

```sh
find 路径 参数 参数值 -exec shell 命令2 {} \;
```
* `xargs `
> 可以在find命令后直接使用`管道`完成前后命令的`数据传递`

```sh
find 路径 参数 参数值 | xargs shell 命令2
```
> 区别:
> -exec :将find查询的结果逐条传递给后边的shell命令
> -xargs：将find查询的结果一次性传递给后边的shell命令

* `grap`
> * -r：如果需要搜索目录中的文件内容，需要进行递归操作，必须指定该参数
> * -i：对应要搜索的关键字，忽略字符大小写的差别
> * -n：在显示符号样式那一行之前，标示出该行的列数编号

```sh
grep "搜索的内容" 搜索的路径/文件 参数
```
## locate
> `locate`是一个`简`化版的`find`,可以根据`文件名`搜索本地的`磁盘文`件，本地数据库Linux系统`每天自动`更新一次，为避免的不及时性，在运行之前，先使用`updatedb`命令，`手动更新数据库`

```sh
# 更新本地数据库磁盘文件
updatedb
# 搜索所有目录下以test开头的文件
locate test;
# 搜索指定目录下以某个关键字test开头的文件
locate /home/foryouos/test
# 搜索文件，忽略文件名的大小写使用参数-i
locate TEST -i
# 列出前N个匹配的文件名称或路径名称，使用参数-n
locate test -n 5
# 基于正则表达式
locate -r "\.cpp$"
```
> 正则表达式:
> 1,在正则表达式中`.`可以匹配任何一个非`\n`的单字符
> 2，上边的命令中使用转译字符`\`对特殊字符`.`转译，就得到了普通的字符`.`
> 3,在正则表达式中`$`放到字符尾部，表示字符串必须以这个字符结尾，上边的命令中修饰的是字符p
> 4,正则表达式中的字符c和后边的字符`p`需要进行字节匹配，没有特殊含义







