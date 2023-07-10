# `libevent`

> `驱动`，`高性能`
>
> `轻量级`，专注于`网络`，跨平台，支持多种`I/O多路`复用技术，`支持I/O`，定时器和信号等事件

# 安装

##  `openssl`下载编译

>[`Openssl下载`](https://www.openssl.org/source/old/1.1.1/)

```sh
# 解压
tar -xzvf openssl-1.1.1h.tar.gz
# cd 目录
cd openssl-1.1.1h
 # 检查安装环境，并生成makefile
./configure  
 # 编译生成 .o可执行文件
make   
# 安装将必要的资源拷贝置系统指定目录
sudo make install  
# 默认安装位置: ​​/usr/local/bin/openssl
# 查看openssl版本
openssl version
```

##  `libevent`下载编译

* 下载:[地址](https://libevent.org/)
* 在Linux中解压

```sh
tar zxvf 下载的libevent压缩文件
cd libevent-2.1.12-stable
# 将库安装到当前系统中
```

* 源码包安装

```sh
./configure   # 检查安装环境，并生成makefile
make          # 编译生成 .o可执行文件
sudo make install  # 安装将必要的资源拷贝置系统指定目录
```

## 测试

>  在`压缩完目录/sample`目录下
>
> ```sh
> ./hello-world
> # 新建窗口，客户端运行
> netcat 服务器IP 9995
> # 当客户端收到hello-world则libevent在本地运行成功
> # 自己编译.c 需要加上 -levent 库
> ```





# 使用

## `libevent` 调用顺序

* event_base() 初始化event_base
* event_set() 初始化event
* event_base_set() 将event绑定到指定的event_base上
* event_add() 将event添加到事件链表上，注册事件
* event_base_dispatch()循环，检测，分发事件

