---
title: linux入门2
date: 2023-04-24 09:26:43
tags:
- Linux
categories:
    - [计算机科学, Linux]
cover: https://i.imgtg.com/2023/04/24/IeNxM.png
---
# Linux文件操作
## Open

> 通过Open函数打开一个磁盘文件，如果不存在还可以打开新的文件

```cpp
#include <sys/type.h>
#include <sys/stat.h>
#include <fcntl.h>
// open 函数只能在Linux系统中使用g
int open(const char *pathname,int flags);
int open(const char* pathname,int flags,mode_t mode);
```
### 参数

> pathname ：被打开的文件名
> flags:指定使用什么方式打开指定的文件，根据需求指定

* `O_RDONLY`: 以只`读`方式打开文件
* `O_WRONLY`: 以只`写`方式打开文件
* `O_RDWR`: 以`读写`方式打开文件
* `可选属性` , 和上边的属性一起使用
* `O_APPEND`: 新数据`追加到文件尾部`，不会覆盖文件的原来内容
* `O_CREAT`: 如果文件`不`存在，`创建该文件`，如果文件存在什么也不做
* `O_EXCL`: `检测`文件`是否`存在，`必须`要和 `O_CREAT` 一起使用，不能单独使用: O_CREAT | O_EXCL
    * 检测到文件`不`存在，创建新文件
    * 检测到文件已经存在，创建失败，函数直接返回 - 1（如果不添加这个属性，不会返回 - 1）
> `mode`：在创建新文件需要`指定新文件`的权限
	
	* 参数最大值`0777`

	* 返回值
	* 成功:返回内核分配的文件描述符，大于0的整数
	* 失败:-1
## Close函数原型
> `释放`Open函数打开的文件描述符
```cpp
#include <unistd.h>
int close(int fd);
```
* 函数参数`:fd文件描述符`，是open()函数的返回值
* 释放成功返回`0`，调用`失败`返回`-1`

## 测试
### 打开文件
> `授予`文件权限
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>

int main(void)
{
	//打开文件
	int fd = open("a.txt", O_RDWR);//以读写的方式
	//创建新文件
	//int fd = open("./new.txt", O_CREAT|O_RDWR, 0664);
	if (fd == -1)
	{
		printf("file open fail\n");
	}
	else
	{
		printf("create file successful fd:%d\n", fd);
	}
	close(fd); //关闭文件
	return 0;
}
```
### 判断文件状态
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>

int main(void)
{
	//检查文件状态，创建失败返回-1，文件不存在，返回分配的问价描述符
	int fd = open("./new.txt", O_CREAT | O_EXCL | O_RDWR);
	if (fd == -1)
	{
		printf("file open fail\n");
	}
	else
	{
		printf("create file successful fd:%d\n", fd);
	}
	close(fd); //关闭文件
	return 0;
}
```
## read
> 用于读取文件`内部`数据，在Open打开文件的时候`指定读权限`
```cpp
#include <unistd.h>
ssize_t read(int fd,void *buf,size_t count);
```

### 参数
* fd：文件描述符，open()函数的返回值，通过这个参数定位打开的`磁盘文件`
* buf:是一个`传出参数`，指向`一块有效的内存`，用于`存储`从文件中`读出`的数据
	* 传出参数:类似于返回值，将变量地址传递给函数，函数调用完毕，地址中就有数据
* count ：buf指针指向的`内存的大小`，指定可以存储的最大字节数

### 返回值
* `大于0`：从文件中读出的字节数，`读文件成功`
* `等于0`：代表文件读完，`读文件成功`
* `-1` : 读`文件失败`

## write
> 将数据`写入`到文件`内部`，通过Open打开文件的时候需要指定写权限

```cpp
#include <unistd.h>
ssize_t write(int fd,const void *buf,size_t count);
```
## 文件拷贝
> 首先在目录中创建`english.txt文件`
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <unistd.h>
#include <fcntl.h>

int main(void)
{
    // 1. 打开存在的文件english.txt, 读这个文件
    int fd1 = open("./english.txt", O_RDONLY);
    if (fd1 == -1)
    {
        perror("open-readfile");
        return -1;
    }

    // 2. 打开不存在的文件, 将其创建出来, 将从english.txt读出的内容写入这个文件中
    int fd2 = open("copy.txt", O_WRONLY | O_CREAT, 0664);
    if (fd2 == -1)
    {
        perror("open-writefile");
        return -1;
    }

    // 3. 循环读文件, 循环写文件
    char buf[4096];
    int len = -1;
    while ((len = read(fd1, buf, sizeof(buf))) > 0)
    {
        // 将读到的数据写入到另一个文件中
        write(fd2, buf, 
        );
    }
    // 4. 关闭文件
    close(fd1);
    close(fd2);

    return 0;
}
```

## lseek移动文件指针
> `lseek`可以通过此`函数移动`文件`指针`
```cpp
#include <sys/types.h>
#include <unistd.h>

off_t lseek(int fd,off_t offset,int whence);
```
### 参数

* fd：文件描述符，open函数的返回值
* `offset`:`偏移量`
* `whence`:通过这个参数指定函数实现功能
	* `SEEK_SET` ：从文件`头部开始`偏移offset个字节
	* `SEEK_CUR`:从当前文件`指针的位置向后`偏移offset个字节
	* `SEEK_END`:从文件尾部`向后`偏移offset个字节
* 返回值
	* 成功:文件指针`从头部`开始计算总的偏移量
	* 失败:-1
```cpp
lseek(fd,0,SEEK_SET); //文件指针移动到文件头部
lseek(fd,0,SEEK_CUR); //得到当前文件指针的位置
lseek(fd,0,SEEK_END); //得到文件总大小
```

### 扩展实现
```cpp
//拓展文件大小
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
int main()
{
	int fd = open("hello.txt",O_RDWR);
	if(fd == -1)
	{
		perror("open");
		return -1;
	}
	//文件扩展字节
	lseek(fd,1000,SEEK_END);
	write(fd," ",1);
	
	close(fd);
	return 0;
}
```

## trucate/ftruncate
> 截断或扩展文件
```cpp
#include <unistd.h>
#include <sys/types.h>
int truncate(const char*path,off_t length);
int ftruncate(int fd,off_t length);
```
### 参数:
* path：要`扩展/截断`的文件的文件名
* fd:文件描述符,open()得到
* length:文件的`最终`大小
	* `size > length` ，文件被`截断`，尾部多余部分被删除
	* `size < length` 文件被`拓展`，文件最终长度为length
* 返回值: 成功返回0 ，失败返回值 -1


## perror
> 返回值来描述系统`函数的状态`(调用是否成功)成功返回0，失败返回-1.
```cpp
#include <stdio.h>

void perror(const char *s);
//指定字符串值，会输出并且输出对应的错误描述信息
```
## 查看文件属性信息
> `file` `文件名`  `[参数]`

### 参数
* `-b` 只显示文件类型和文件编码，不显示文件名
* `-i` 显示文件的MIME类型 `MIME`(Multipurpose Internet Mail Extensions)`多用途互联网邮件扩展类型`
* `-F` 设置输出字符串的`分隔符`
* `-L` 查看`软链接`文件自身文件属性

## stat 命令
> 显示文件或目录的`详细属性`信息和文件系统状态，输出的更加详细

```cpp
stat [参数] 文件或者目录名
```
### 参数
* `-f` 仅显示文件`所在文件系`统的信息
* `-L` 查看`软链接`文件
* `-c` 查看文件`某个属性`信息
* `-t` 简洁模式，只`显示摘要`信息，不显示属性描述
### 输出的信息属性
*`File` ：文件名
* `Size`: 文件大小，单位字节
* `Blocks`: 文件使用的数据库总数
* `IO Block` ：IO块大小
* `regular file `: 文件的实际类型，文件类型不同，该关键字也会变化
* `Device` : 设备编号
* ` Inode` : Inode号，操作系统用inode编号来识别不同的文件，找到文件数据所在的block读出数据
* `Links`:硬链接计数
*` Access` : 文件所有者+所属组用户+其他人对文件的访问权限
* `Uid`:文件所有者名字和所有者ID
* `Gid` ： 文件所有数组名字已经组ID
* `Access Time` :表示文件的最后访问时间，
* `Modify Time`:表示文件内容的最后修改时间
* `Change Time` :文件的状态休息，被修改则更新，例如文件的硬链接链接数，大小，权限等
* `Birth`：文件生成日期

### stat/lstat函数
> `lstat() `：得到的是`软链接`文件本身的`属性信息`
> `stat()` : 得到的是`软链接`文件`关联`的文件的属性信息

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <unisd.h>

int stat(const char *pathname ,struct stat *buf);
int lstat(const char *pathname,struct stat *buf);
```
#### 参数
* `pathname` ： 文件名，要获取这个文件的`属性信息`
* `buf` ：` 传出参数`，文件的信息被`写入`到这块内存中
* `返回值`:成功返回`0`，失败返回`-1`.
```cpp
struct stat {
    dev_t          st_dev;        	// 文件的设备编号
    ino_t           st_ino;        	// inode节点
    mode_t      st_mode;      		// 文件的类型和存取的权限, 16位整形数  -> 常用
    nlink_t        st_nlink;     	// 连到该文件的硬连接数目，刚建立的文件值为1
    uid_t           st_uid;       	// 用户ID
    gid_t           st_gid;       	// 组ID
    dev_t          st_rdev;      	// (设备类型)若此文件为设备文件，则为其设备编号
    off_t            st_size;      	// 文件字节数(文件大小)   --> 常用
    blksize_t     st_blksize;   	// 块大小(文件系统的I/O 缓冲区大小)
    blkcnt_t      st_blocks;    	// block的块数
    time_t         st_atime;     	// 最后一次访问时间
    time_t         st_mtime;     	// 最后一次修改时间(文件内容)
    time_t         st_ctime;     	// 最后一次改变时间(指属性)
};
```
###  读取文件大小
```cpp
#include <sys/stat.h>
#include <stdio.h>
int main(void)
{
	//定义结构体存储文件休息
	struct stat myst;
	int ret = stat("./english.txt", &myst);
	if (ret == -1)
	{
		perror("stat error"); //失败
	}
	printf("文件大小为:%d\n", (int)myst.st_size);
	return 0;
}
```
## 目录遍历
### `Opendir` 函数打开目录

```cpp
#include <sys/types.h>
#include <dirent.h>
//打开目录
DIR *opendir(const char *name);
```
* 参数： `name`  要打开的目录名字
* 返回值`DIR*`,结构体类型指针，打开失败返回NULL

### readdir

> 目录打开后，通过readdir()函数`遍历目录中的`文件信息。
> `每调用一次`这个函数就可以得到目录中的一个`文件信息`，当目录中的文件信息被`全部遍历`完毕会得到一个`空对象`
```cpp
#include <dirent.h>
struct dirent *readdir(DIR *dirp)
```

### 参数
* `dirp->opendir()`函数的返回值
* 函数调用`成功`，返回读到的文件的信息，目录文件被读完或者函数`调用失败`返回`NULL`

### struct dirent结构体原型
```cpp
struct dirent {
    ino_t          d_ino;       /* 文件对应的inode编号, 定位文件存储在磁盘的那个数据块上 */
    off_t          d_off;       /* 文件在当前目录中的偏移量 */
    unsigned short d_reclen;    /* 文件名字的实际长度 */
    unsigned char  d_type;      /* 文件的类型, linux中有7中文件类型 */
    char           d_name[256]; /* 文件的名字 */
};
```

* 结构体中文件类型`d_type`的宏值
	`DT_BLK`：块设备文件
    `DT_CHR`：字符设备文件
    `DT_DIR`：目录文件
    `DT_FIFO` ：管道文件
    `DT_LNK`：软连接文件
    `DT_REG` ：普通文件
    `DT_SOCK`：本地套接字文件
    `DT_UNKNOWN`：无法识别的文件类型
* 通过readdir()函数`遍历`某个目录中的文件
```cpp
//打开目录
DIR* dir = opendir("/home/foryouos");
struct dirent* ptr = NULL;
//遍历目录
while((ptr = readdir(dir)) != NULL)
{
	
}
```
### closedir

> 目录操作完毕之后，需要通过`closedir()关闭`通过opendir()得到的示例，释放资源
```cpp
//参数是opendir()的返回值
int closedir(DIR *dirp);
//目录关闭成功返回0
//失败返回 -1
```

### 遍历单层目录
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <dirent.h>
//传入目录参数
int main(int argc, char* argv[]) //在./a.out 传入参数目录，绝对路径
{
	DIR* dir = opendir(argv[1]);   //打开文件
	if (dir == NULL) //如果目录 为空
	{
		perror("opendir NULL");
		return -1;
	}
	//遍历当前目录中的文件
	int count = 0;
	while (1) //循环遍历
	{
		struct dirent* ptr = readdir(dir); //读目录的结构体，每次读一层
		if (ptr == NULL)  //如果为空
		{
			printf("目录读完了...\n");
			break;
		}
		//读到一个文件，判断文件类型
		if (ptr->d_type == DT_REG) 
		{
			char* p = strstr(ptr->d_name, ".txt"); //输出名字
			if (p != NULL && *(p + 4) == '\0') //若明细不空
			{
				count++;
				printf("file %d: %s\n", count, ptr->d_name);
			}
		}
	}
	printf("%s目录中txt文件的格式：%d\n", argv[1], count);
	closedir(dir); //关闭目录
	return 0;
}
```


### 遍历多层目录
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <dirent.h>

int GetFile(const char* path)
{
	DIR* dir = opendir(path);
	if (dir == NULL) //如果目录 为空
	{
		perror("opendir NULL");
		return -1;
	}
	//遍历当前目录
	struct dirent* ptr = readdir(dir); //读目录的结构体，每次读一层
	int count = 0;
	while ((ptr = readdir(dir)) != NULL) //只要目录不为空，就一直遍历先去
	{
		//当文件名为.或者为..时，表示当前目录或者父级目录,则通过continue语句跳过，不做处理
		if (strcmp(ptr->d_name, ".") == 0 || strcmp(ptr->d_name, "..") == 0)
		{
			continue;
		}
		else if (ptr->d_type == DT_DIR) //当读取的为目录类型
		{
			//目录
			char newPath[4096];
			sprintf(newPath, "%s/%s", path, ptr->d_name); //输出未见类型，并递归遍历
			count += GetFile(newPath);
		}
		else
		{
			//若为普通文件
			char* p = strstr(ptr->d_name, ".txt");
			if (p != NULL && *(p + 4) == '\0')
			{
				count++;
				printf("%s/%s\n", path, ptr->d_name);
			}
		}
		
	}
	closedir(dir);
	return count;

}
//传入目录参数
int main(int argc, char* argv[]) //在./a.out 传入参数目录，绝对路径
{
	if (argc < 2)
	{
		printf("./a.out path\n");
		return 0;
	}
	int num = GetFile(argv[1]);
	printf("%s .c :%d\n", argv[1], num);
	return 0;
}
```