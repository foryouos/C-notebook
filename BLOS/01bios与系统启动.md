# 知识储备

> * C语言
> * C++
> * 汇编
> * Linux

# 汇编部分补充

## 数据描述符

* `AX`累加器：用到最多最频繁，AX，AH和AL在乘，除法等操作中有专门的用途。
* `BX`基址寄存器: 用于存放偏移地址
* `CX`为计数寄存器：在循环操作中做计数器用，用于控制循环程序的执行次数
* `DX`数据寄存器：在乘，除法及I/O端口操作时专门用途。

## 指令

### 操作方向标识为DF(`Direction Flas`)

> 使用此指令`控制`方向标志`DF`，决定内存地址增大还是减小。
>
> 在子串操作中使SI或DI的地址指针自动递减，字串处理由后往前。

* `CLD` 使`DF复位`，即让`DF=0` 向`高`地址`增加`
* `STD` 使`DF置位`，即让`DF = 1` 向`低`地址`减少`

# CPU架构

> `CPU架构`是CPU厂商给属于`同一系列`的`CPU`产品定的一个规范，主要目的是为了区分`不同类型CPU`的重要标示。

![CPU架构](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4ByNjnwsNRWwSWz7o6lxGlRJibtbMexd9fVyvTd5RGgPGw26OKQGOQ08g/640?wx_fmt=png)

* 复杂指令集`CISC ` (Complex Instruction Set Computer)： 增强原有指令的功能，设置更为`复杂`的新指令实现`软件功能的硬化`。

* 精简指令集`RISC`  (Reduced Instruction Set Computer)：`较少指令`种类和`简化`指令功能，提高指令的执行效率

![指令集对比](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BvRPydwpIeGqTKjuF64U6eXgaLAISGSf7agzEsNFibbERvPUz2xT4AXg/640?wx_fmt=jpeg)

## 南桥北桥

在早期，芯片组分为南桥芯片组和北桥芯片组两部分，其中北桥负责`CPU`与`内存的数据`交换，`图形处理`，`CPU与PCIE`(高速串行计算机扩展总线标准)数据交换，南桥负责系统的输入输出功能。`北桥芯`片还叫"`图形与内存控制器`"，`南`桥叫"`输入/输出`控制器"。北桥芯片组因与CPU联系密切靠近CPU位置，在现代制造工艺越来越先进，集成度越来越高，`内存控制`器已经被集成到`CPU内部`，显卡收进CPU(核显)，而PCIE控制器收归南桥管理，北桥芯片组功能基本被瓜分。在Intel 芯片组中北桥被取消，而AMD只有早期主板仍保留着北桥和南桥。

`PCIE`:属于`高速串行`点对点双通道`高宽带传输`，所连接的设备分配独享通道带宽，不共享总线带宽，主要支持主动电源管理，错误报告，端对端的可靠性传输，热插拔以及服务质量QOS等功能。

![南桥](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bsiad3zKognzINjEeibkz0PQR2ekCJuad6lKd0jZicsnvJe5tU92BZjibNQ/640?wx_fmt=jpeg)

# 电脑启动过程(早期)

![启动顺序](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BK8ZUDicv3z4CkGQU5POoQFKMYKwV9A6AibeosfQZyOVODxJNAtURHw8A/640?wx_fmt=jpeg)



* 系统`加电BIOS`初始化`硬件`
* BI0S`读`取`引导扇区代码` -- 加载程序
* 加载`内核`并`跳转到内核执行` - 操作`系统内核`

![刚启动时内存布局](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BU2zseB4kZa6REX4UAsrH93ciaLeibOFkK4eh7R0zwxU6OibqNbCUiagfxA/640?wx_fmt=jpeg)

![十六位实模寻址](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bpp9OWibhiapRDXicRSz5KBn2icAD9gIqticmiaLuTP9H1R43D8WRqmDLGgwQ/640?wx_fmt=jpeg)

## 当装入`多个`操作系统

![主引导记录MBR格式](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BfzrQX4xIRkvn2e0TCEib5QMcRFt6Jk70jUBEvP6HuCS8MTwQwjuqhPg/640?wx_fmt=jpeg)

![多个操作系统](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BzQdZGSwq6xfIvm4RxGatO1PFnuTTrKaJR6pSZJ60Lwve6aTazqhU8w/640?wx_fmt=jpeg)



# 工作模式

* `实模式`

> 程序中用到的地址都是`真实`的物理地址。
>
> 在实模式下，内存寻址方式和`8086`（8086,微处理器，`1MB`内存地址,`3微米`晶体管，`IBM`1981年生产的`第一台`电脑就是使用8086简化版，标志着`x86架构`和IBM PC兼容电脑的产生)相同，`机器段`起始地址的`低4位`设置为`0`，由`16位段寄存器`的内容乘以16(`左移4位`)作为`段基址`(Segment Base Address)(能被16整除的主存物理地址)，加上16位段偏移地址形成`20位`物理地址，
>
> ![段基址](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B3XhjCZwyPmsic3DiauW32qmHKlFM1mRAa0554zaIMrCSIKEU9h9sSgug/640?wx_fmt=jpeg)
>
> 最大`寻址空间1MB`，最大`分段64KB`。可以使用`32位指令`，即32位的x86 CPU也可以兼容实模式，此时的实模式相当于高速8086(32位CPU的实模式可以使用32位下的资源)。在`32位CPU`下，系统复位或加电时都是以`实模式启动`，然后再切换为`保护模式`。在实模式下，所有的段都可以`读`，写和可执行的。由于实模式下`没有特权级`，程序可以`随意修改`自己的段基址，加上实模式下对地址的访问就是`实际物理地址`，随意修改给操作系统带来极大安全隐患

* 保护模式

>`标志位`表示`权限`，当用户访问与读取的`段`文件权限进行`对比`，已达到保护的目的。
>
>每一个指令，每一个程序本身就有一个权限，可以用`CPL/RPL`描述，访问的目标字符段也有一个权限为`DPL`。处理器会对特权集进行检查，`判断`当前的`CPL/RPL`是不是`大于等于`DPL。

![实模式与保护模式](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BEDLmz00ZSeC6gRFlxJDKechibd0HIQPSHY05mUKRolWSumXj8JtJIjg/640?wx_fmt=jpeg)

# BIOS概念

> BIOS(`Basic Input Output System`)全称`基本输入/输出系统`，固件，它是存储在`主板ROM`(只读存储器，生成之后`只有一次`写入机会，数据一旦写入则不可更改。按照内容写入方式分为: `可一次变成PROM`，`可擦除`ROM，又分为`EPROM紫外线`擦除电写入和`E2PROM电擦`除电写入等)里的一组程序代码。
>
> 主要包括:
>
> * `加电自检`(Power On Self Test POST)程序，用于开机时对硬件的检测，BIOS包含基本输入输出程序，包括读取键盘，写入屏幕，和执行磁盘I/O等操作过程，去检测开机时系统状况，而`显卡不可检测`
>
> > `Blos`(Blos采用`16位汇编语言`编写)只能运行在`16位`实模式下，实模式下最大寻址范围时`1MB`
> >
> > 系统加电时，当CPU收到复位事件时，当它被上电或重新启动时--指令寄存器就被装入 一个预定义的内存位置，并在那里开始执行。
>
> * 系统初始化代码，包括`硬件设备的初始化`，创建`BIOS中断向量`等
> * 基本的外围`I/O处理`子程序代码
> * `CMOS`设置程序：Complementary Metal-Oxide-Semiconductor: 保存了系统引导的最基本的资料(基本设置，时钟信息)。

# 初始化过程

* 硬件`自检POST`
* 检测系统中`内存`和`显卡`等关键部件的存在和工作状态
* 查找并执行显卡等接口`卡BIOS`，进行`设备初始化`
* 执行`系统BIOS`，进行`系统检测`  -- 检测和配置系统中安装的即插即用设备
* 更新`CMOS`中的扩展`系统配置数据ESCD`
* 按指定`启动顺`序从`软盘`，`硬盘`或`光驱`启动

![bios](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BxA9alCgs4ElF44pRkOrPW70uUlJRhRj1DIoKcERTMn9ofD9ZU98V5A/640?wx_fmt=jpeg)

## 启动加载程序`Bootloader`

![引导启动内核的过程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B2OIUQd5DTicpXEibxicEJqr1iav7QOyqZEwIr71O9RFg7R3P9Ek07TCdEA/640?wx_fmt=jpeg "引导启动内核过程")

![引导启动内核程序](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B6x6tLUVfJR6B1wPtBlGObNGWEfFuPNRicZb4Kic8DcHGHO09Gk4OLib8w/640?wx_fmt=jpeg)

![linux启动代码](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bs1I9XzoFKxkcW7Eib1FXG3Jib2FtgIq56JicnjI8hjulw4iaibYy3cLzuPA/640?wx_fmt=jpeg)

> 到`setup`部分，进入`保护模式`
>
> 运行`system模块`，进入`操作系统`

## bootsect.s

> 工作在`实模式`下，起到`搬运工`的作用

![第一部分](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BJXFziawv1rVvj8HBq46iaHvZtsZx1BEmMucOicjjILtPiadB2U9TV9jX0A/640?wx_fmt=jpeg)

![片段二](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BdyicxGGSmiaofPpXnpvTiaicwcWAmHukPR6kW9VaLnpzdt8mPqGN6LwWmw/640?wx_fmt=jpeg)

![片段三](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B0N6sWgMW2pXic8bibhx5vykoPVRD4wdyH2VxGPjxhFwfwYIj9tqia5rHQ/640?wx_fmt=jpeg)

![片段四](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BWy6JQpMcOGbA1iakjOVywTAWx5Otibypbk6jo2hDicMAUWicIcuNEF0HmQ/640?wx_fmt=jpeg)

![片段六](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B59IeS5UVtXTrZVeCKFcF0q0AroO2tgea1iciby8bvMz5GTxtoedLSwLw/640?wx_fmt=jpeg)

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  ![片段八](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BoM0aqAlVqabwq6t1ZIFxG8GuV9awJU1AiaWnyddt8U8obTibVv9b0YOA/640?wx_fmt=jpeg)

# setup.s

> `setup.s`负责从`BIOS`中`获取系统数据`，并将这`些数据`放到`系统内存`的`适当地方`。此`setup.s`和`system`已经`由bootsect引`导块加载到`内存`中。

![setup](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BgRc8dxJZicruEg0QibzCfPFJ2CJGevt3icYKYeaPuDcafeKZicnZKJjibcw/640?wx_fmt=jpeg)

![片段一](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BYlp41bnLC7O1BlDXY1j3kyKfe4TACrnoiaOeYGibS7ULLrohfUwkAlUA/640?wx_fmt=jpeg)

![片段二](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BWSK6GYVlbT0AhLWhISWiaLqpmc4mvtlT26MjCZKibahiaRP3hBGmlIsyQ/640?wx_fmt=jpeg)

![片段四](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bic87KoggRjlDlIAPKN8nVU4Cjnqu7VdwlMR5PC0tPAZ2A6tapkdfEKQ/640?wx_fmt=jpeg)

![片段五](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BOGQvAOtNiaaCOFAeElGWvkPn5g8C9JtY3KiaiaG8p3e6cGW53zMApyicsQ/640?wx_fmt=jpeg)

![总](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BNydVMOhKibW5eJiahL6pIyPPcfYWZtWzt8UcqKR34pC0yu0cMWHUiajuw/640?wx_fmt=jpeg)

# head.s

> 进一步设置`中断描述符`和`全局描述符`表，设计`页表`--开始对`系统内存`进行`管理`

![head.s](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BhXk6PCSQblUOxcvrNwqeFWg5VxFopVuSXBLklTW31QneHYp4THvzsA/640?wx_fmt=jpeg)



![设置中断描述符](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BcVXZwJwEV8nA2NFuBSaOtjOjQzS8Rtia4Zzx5J3EE3HF1UvyhhVptlA/640?wx_fmt=jpeg)

![mian函数工作流程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BzZhsOHicn9EFib3QPpmticBWqzIGia6uHsK2dvxhoxCuhEMdOuVxgdnxtA/640?wx_fmt=jpeg)

![main函数](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BfsWLLxEujMvZMpqCbuPpic5tRmfYFFTGmhLic72hSianGda1X2IUI9aiaA/640?wx_fmt=jpeg)

# BIOS缺点

* `开发效率`低 ：大部分BIOS使用汇编开发，开发效率低，汇编开发代码与设备的耦合度太高，(软件工程讲究`高内聚低耦合`，目的是使程序模块的可重用性，移值性大大增强)
* `性`能差：BIOS基本输入输出通过中断来完成，开销大，并且BIOS没有提供异步工作模式，大量时间消耗在等待上
* 功能`扩展性`差，升级缓慢：BIOS代码采用静态链接，增加硬件功能时，必须将16位代码放置在0x0C0000 ~ 0x0DFFFF区间，初始化时将其设置为约定的中断处理程序。而且BIOS没有提供动态加载设备驱动方案
* `安全`性：BIOS运行过程中对可执行代码没有安全方面考虑
* 不支持从硬`盘2TB`以上的`地址引导`：受限于BIOS硬盘的寻址方式，BIOS硬盘采用`32位`地址，因而引导扇区的最大逻辑块地址`2^32` (换算成字节地体，即`2^32 X 512` = 2TB)

# UEFI

![UEFI](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B2fNXBA3x10gwrRbOcYtZKEG9zbMfMknmHxibCia4iaWRaPcy6n8QFia4Ug/640?wx_fmt=jpeg)



# 参考资料

* 厦门大学《操作系统原理》
* 《UEFI原理与编程》
  * 获得BIOS中英文对照：公众号回复：`BIOS英文` 即可获得
  * 下载:https://foryouos.lanzoul.com/iApcp138b9be 密码:5213

