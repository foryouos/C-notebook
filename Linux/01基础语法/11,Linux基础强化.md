# linux硬件运行状况







# df和du指令

```sh
# 用于显示目录或文件的大小，同时可以后接目录
# du逐级进入指定目录的每一个子目录，逐个计算，并最终显示，若当前用户没有权限，无法进行计算
du   # disk usage 统计目录所占磁盘空间大小
# 显示当前目录下所有文件和文件夹的总大小
du -sh  #也可加目录名

# 接入排序，查看/data目录下的所有文件和文件夹系统，找出搜友GB大小的文件，并从大到小排序
du -sh /data | grep G | sort -nr

# df用于显示目前linux系统上的文件系统磁盘使用情况
# 磁盘，不统计具体文件和文件夹
# df通过文件系统快速获取文件大小信息，速度块，效率高
df  #  disk free 占用多少，还剩多少
# df通过文件系统来获取文件大小，当我们删除一个文件时，这个文件不是马上就消失，而是暂时消失，当所有程序都不同时，才会根据OS的规则释放掉已经删除的文件
df -h  #使用结果以K,M,G为单位，提高信息可读性
```

# 权限问题

* `所有`者
* 所有者`所在组`
* `其它`用户

```sh
# 某文件权限 
 -rwxrwxr--
# 第一位表示文件类型
# 2~4为表示文件所有者对文件的权限
# 5~7表示文件所有者所在组的用户对文件的权限
# 8~10表示其他用户对文件的权限
# r表示可读
# w表示可写
# x表示可执行
# - 表示没有权限
# 二进制表示
# 如果可读，权限二进制位100,十进制是4
# 如果可写，权限二进制位010,十进制是2
# 如果可自行，权限二进制位001，十进制是1

# 组外成员只读，组内成员读与写，所有者全部权限
# rwxrw-r--
# 764
```

# wget下载

> 从网络上自动下载文件，支持http,https,ftp协议，wget可在后台运行，所有它可以批量下载文件

```sh
-P # 指定下载的文件夹
-O # 下载的位置并且可以重命名文件
-N # 如果文件不会新的就不下载了

# 断点续传
wget --continue 链接
wget -c 链接
```





## 举例

```sh
# 下载url.txt文件中所有包含url链接的所有文件
wget -i url.txt
# 下载文件不显示下载信息
wget -q 链接
# 下载文件链接并显示下载信息
wget -d 链接
# 下载一系列文件
wget http://www.lxlinux.net/file_{1..4}.txt
```





# 查看空间

```sh
# 查看当前文件就爱大小
du -sh
# 查看当前目录下及子目录文件大小
du -sh *
# 最大深度为1
du -lh --max-depth=1
# 根据文件排序
du -lh --max-depth=1 | sort -nr
```





# 系统信息查询总

```sh
# 测试使用的WSL2 基于Windows

# 查看内核/操作系统/CPU信息
uname -a

# 查看操作系统版本
head -n -1 /etc/issue

# 查看CPU信息
cat /proc/cpuinfo

# 查看计算机名
hostname

# 列处所有PCI设备
lspci -tv

# 列出所有USB设备
lsusb -tv

# 列出加载的内核模块
env

# 查看内存使用量和交换区使用量
free -m 

# 查看各分区使用情况
df -h

# 查看置顶目录的大小
du -sh <目录名>

# 查看内存总量
grep MemTotal /proc/meminfo

# 查看空闲内存量
grep MemFree /proc/meminfo

# 查看系统运行时间
uptime

# 查看系统负载磁盘和分区
cat /proc/loadavg 

# 查看所有分区
fdisk -l

# 查看所有交换分区
swapon -s

# 查看磁盘参数(仅适用于IDE设备)
hdparm i /dev/hda

# 查看启动时IDE设备检测状态网络
dmesg | grep IDE

# 查看所有网络接口属性
ifconfig

# 查看防火墙设置
iptables

# 查看路由表
route -n

# 查看所有监听㐰
netstat -lntp

# 查看所有已经建立的连接
netstat -antp

# 查看网络统计信息进程
netstat -s

# 终止进程
kill -9 进程号

# 查看所有进程
ps -ef

# 实时显示进程状态用户
top

# 查看活动用户
w

# 查看置顶用于信息
id <用户名>

# 查看用户登录日志
last

# 查看系统所有用户
cut -d: -f1 /etc/passwd

# 查看系统所有组
cut -d: -f1 /etc/grou[

# 查看当前用户的计划任务服务
crontab -l

# 列出所有系统服务
chkconfig -list

# 列出所有启动的系统服务程序
chkconfig -list | grep on

# 查看物理CPU个数，核数，逻辑CPU个数
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU核数 X 超线程数


# 查看CPU的统计信息
lscpu

# 以KB为单位显示磁盘使用量和占有率，-m则是以M为单位显示磁盘使用量和占有率
```

# 具体指令

##  top

![image-20230724111429203](E:\学习笔记\C-notebook\Linux\01基础语法\assets\image-20230724111429203.png)



> * top指令`默认`显示所有`进程`相关信息，默认`显示进程`，线程级可以通过`ps命令`获取
> * `进程实际`使用的内容看`RES`那一列，`VIRT`表示进程使用的是`虚拟内存`数据，`SHR`表示`共享内存`的数据
> * `TIME+`表示进程`使用CPU时间总计`，而非进程的存活时间
> * `%CPU`标识进程所占`CPU的百分比`，通过此可以得出CPU利用率

| 名称      | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| `PID`     | 进程ID                                                       |
| `USER`    | 进程所有者的`ID`                                             |
| `PR`      | 进程优`先级`                                                 |
| `NI`      | `nice值`。负值表示高优先级，`正值`表示`低`优先级             |
| `VIRT`    | 进程使用的`虚拟内存总量`，单位kb。VIRT = SWAP + RES          |
| `SHR`     | `共享`内存大小，单位`kb`                                     |
| `S`       | 进程状态，`D`:`不可中断`的睡眠状态，`R运行`，`S睡眠`，`T跟踪/停止`，`Z僵尸进程` |
| `%CPU`    | 上次`更`新到`现在`的CPU时间占比百分比                        |
| `%MEM`    | 进程使用`物理`内存百分比                                     |
| `TIME+`   | 进程使用的CPU时间总计，单位`1/100秒`                         |
| `COMMAND` | 命令名/命令行                                                |

## route查看路由表

![image-20230724123032967](E:\学习笔记\C-notebook\Linux\01基础语法\assets\image-20230724123032967.png)

* `destination` : 目标网络或目标主机，`Destination`为`default(0.0.0.0),`表示默认网关
* `Gateway` : 网关地址，`0.0.0.0`表示当前记录对应的Destination跟`本机`在同一个`网段`，通信时不需要经过网关(同一个局域网内2台主机通信不需要经过网关)
* `Genmask`: 为`Destination`字段的`网络掩码`，Destination是主机时需要设置`255.255.255.255`是`默认路由`时会设置为`0.0.0.0`
* `Metric `:` 跃点`。指到达指定网络所需的`中转数`，是`大型局域网`和`广域网`设置所必须的
* `Ref` : 路由项`引用次数`
* `Use` : 此路由项被路由查找的次数
* `Iface` : `网卡名字`
* `Flag`含义
  * `U(route is up)` : 该路处于`活跃`
  * `H(target is a host)` : 目标是一部`主机(IP)`而`非网域`(子网掩码是255.255.255.255)
  * `G(use gateway)` : 需要透过外部的主机(gateway) 来传递封包(一般指向默认网关)
  * `R(reinstate route for dynamic routing) `: 使用动态路由时，恢复路由资讯的旗标
  * `D(dynamically installed by daemonn or redirect) `: 已经由服务或`port功能`设定为`动态路由`
  * `M(modified from routing daemon or redirect) `：路由已经`被修改`了
  * `!(reject route) `: 这个路由将`不会被接受`(用来抵挡`不安全的网域`)

# 查看`CPU`信息`lscpu`

```sh
foryouos@bottle:~$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
Address sizes:       36 bits physical, 48 bits virtual
CPU(s):              4
On-line CPU(s) list: 0-3
Thread(s) per core:  2
Core(s) per socket:  2
Socket(s):           1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               158
Model name:          Intel(R) Pentium(R) CPU G4560 @ 3.50GHz
Stepping:            9
CPU MHz:             3504.000
CPU max MHz:         3504.0000
BogoMIPS:            7008.00
Virtualization:      VT-x
Hypervisor vendor:   Windows Subsystem for Linux
Virtualization type: container
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr s
                     se sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 s
                     sse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave osxsave rdr
                     and lahf_lm abm 3dnowprefetch fsgsbase tsc_adjust smep erms invpcid mpx rdseed smap clflushopt inte
                     l_pt ibrs ibpb stibp ssbd
```

> * `Architecture` : `架构`
> * `CPU(s) `: 逻辑CPU个数
> * `THread(s) per core` : 每核`超线程数`
> * `Core(s) per socket` : 每核`CPU`数
> * `Socket(s)` : 物理`CPU数`
> * `NUMA node0 CPU(s)` : 逻辑`CPU`及`numa`映射



## 清屏

```sh
clear # 与ctrl + L类似：刷新屏幕，实质只是让终端显示页后翻一页，向上仍可以看到之前的代码
reset # 完全刷新终端屏幕，之前输入操作信息将被清空，如果你直接使用一个终端，该应用将会立刻关闭

# 补充：在window CMD中清屏
cls
```

# 部分快捷指令

* `Ctrl + C `：中断命令或进程，将`立刻终止`运行的顺序
* `Ctrl + Z` ：该快捷键将`正在运行的顺序`送到后台，
* `Ctrl + D `:  对键盘快捷键将使你`退出当前终端`，如果你`使用SSH连接`，将会关闭
* `Ctrl + A` ：移动光标到`行首`，
* `Ctrl + E `: 移动光标到`行尾`

* `Ctrl + U `: 该快捷键会`擦除当前光标位置`到`行首`的全部内容
* `Ctrl + K` ：查出当前`光标位置`到`行尾`的全部内容
* `Ctrl + w` ：可以擦除`光标位置前`的`单词`，如果光标在一个单词自身上，将擦除从光标位置到行首的`全部单词`。将移动光标到要删除单词后的一个空格上，然后使用此删除
* `Ctrl + y` : `撤回`删除









# 参考资料

* [Linux常用命令](https://blog.csdn.net/xixingzhe2/article/details/82826152)
* [Linux清屏](https://007.gangguana.com/a/4af4da12a84e7c2a3e38e2ae484f0926.shtml)
* 零声教育
* 