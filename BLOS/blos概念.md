# 知识储备

> * C语言
> * C++
> * 汇编
> * Linux

# CPU架构

> CPU架构是CPU厂商给属于同一系列的CPU产品定的一个规范，主要目的是为了区分不同类型CPU的重要标示。

![CPU架构](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4ByNjnwsNRWwSWz7o6lxGlRJibtbMexd9fVyvTd5RGgPGw26OKQGOQ08g/640?wx_fmt=png)

* 复杂指令集`CISC ` (Complex Instruction Set Computer)： 增强原有指令的功能，设置更为`复杂`的新指令实现`软件功能的硬化`。

* 精简指令集`RISC`  (Reduced Instruction Set Computer)：`较少指令`种类和`简化`指令功能，提高指令的执行效率

![指令集对比](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BvRPydwpIeGqTKjuF64U6eXgaLAISGSf7agzEsNFibbERvPUz2xT4AXg/640?wx_fmt=jpeg)

## 南桥

![南桥](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bsiad3zKognzINjEeibkz0PQR2ekCJuad6lKd0jZicsnvJe5tU92BZjibNQ/640?wx_fmt=jpeg)

# 电脑启动过程(早期)

![启动顺序](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B7JqfiblrtJGyLq4v4hfzzQUK5Fc8Ribb1ELHNMQPJibXhuBBhHrDHbkvg/640?wx_fmt=jpeg)



* 系统加电BIOS初始化硬件
* BIOS读取引导扇区代码 -- 加载程序
* 加载内核并跳转到内核执行 - 操作系统内核

![刚启动时内存布局](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BU2zseB4kZa6REX4UAsrH93ciaLeibOFkK4eh7R0zwxU6OibqNbCUiagfxA/640?wx_fmt=jpeg)

![十六位实模寻址](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bpp9OWibhiapRDXicRSz5KBn2icAD9gIqticmiaLuTP9H1R43D8WRqmDLGgwQ/640?wx_fmt=jpeg)





# 工作模式

* 实模式

> 程序中用到的地址都是真实的物理地址。
>
> 在实模式下，内存寻址方式和`8086`（8086,微处理器，`1MB`内存地址,`3微米`晶体管，`IBM`1981年生产的`第一台`电脑就是使用8086简化版，标志着`x86架构`和IBM PC兼容电脑的产生)相同，`机器段`起始地址的`低4位`设置为`0`，由`16位段寄存器`的内容乘以16(`左移4位`)作为`段基址`(Segment Base Address)(能被16整除的主存物理地址)，加上16位段偏移地址形成`20位`物理地址，
>
> ![段基址](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B3XhjCZwyPmsic3DiauW32qmHKlFM1mRAa0554zaIMrCSIKEU9h9sSgug/640?wx_fmt=jpeg)
>
> 最大`寻址空间1MB`，最大`分段64KB`。可以使用`32位指令`，即32位的x86 CPU也可以兼容实模式，此时的实模式相当于高速8086(32位CPU的实模式可以使用32位下的资源)。在`32位CPU`下，系统复位或加电时都是以`实模式启动`，然后再切换为`保护模式`。在实模式下，所有的段都可以`读`，写和可执行的。由于实模式下`没有特权级`，程序可以`随意修改`自己的段基址，加上实模式下对地址的访问就是`实际物理地址`，随意修改给操作系统带来极大安全隐患

* 保护模式

>标志位表示权限，当用户访问与读取的段文件权限进行对比，已达到保护的目的。
>
>每一个指令，每一个程序本身就有一个权限，可以用CPL/RPL描述，访问的目标字符段也有一个权限为DPL。处理器会对特权集进行检查，判断当前的CPL/RPL是不是大于等于DPL。

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
> * 系统初始化代码，包括硬件设备的初始化，创建BLOS中断向量等
> * 基本的外围I/O处理的子程序代码
> * CMOS设置程序

# 初始化过程

* 硬件自检POST
* 检测系统中内存和显卡等关键部件的存在和工作状态
* 查找并执行显卡等接口卡BIOS，进行设备初始化
* 执行系统BIOS，进行系统检测  -- 检测和配置系统中安装的即插即用设备
* 更新CMOS中的扩展系统配置数据ESCD
* 按指定启动顺序从软盘，硬盘或光驱启动

![bios](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BxA9alCgs4ElF44pRkOrPW70uUlJRhRj1DIoKcERTMn9ofD9ZU98V5A/640?wx_fmt=jpeg)

## 启动加载程序Bootloader

![引导启动内核的过程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B2OIUQd5DTicpXEibxicEJqr1iav7QOyqZEwIr71O9RFg7R3P9Ek07TCdEA/640?wx_fmt=jpeg "引导启动内核过程")

![引导启动内核程序](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B6x6tLUVfJR6B1wPtBlGObNGWEfFuPNRicZb4Kic8DcHGHO09Gk4OLib8w/640?wx_fmt=jpeg)

![linux启动代码](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bs1I9XzoFKxkcW7Eib1FXG3Jib2FtgIq56JicnjI8hjulw4iaibYy3cLzuPA/640?wx_fmt=jpeg)

> 到setup部分，进入保护模式
>
> 运行system模块，进入操作系统

## bootsect.s

> 工作在实模式下，起到搬运工的作用

![第一部分](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BJXFziawv1rVvj8HBq46iaHvZtsZx1BEmMucOicjjILtPiadB2U9TV9jX0A/640?wx_fmt=jpeg)

![片段二](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BdyicxGGSmiaofPpXnpvTiaicwcWAmHukPR6kW9VaLnpzdt8mPqGN6LwWmw/640?wx_fmt=jpeg)

![片段三](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B0N6sWgMW2pXic8bibhx5vykoPVRD4wdyH2VxGPjxhFwfwYIj9tqia5rHQ/640?wx_fmt=jpeg)

![片段四](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BWy6JQpMcOGbA1iakjOVywTAWx5Otibypbk6jo2hDicMAUWicIcuNEF0HmQ/640?wx_fmt=jpeg)



![片段六](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4B59IeS5UVtXTrZVeCKFcF0q0AroO2tgea1iciby8bvMz5GTxtoedLSwLw/640?wx_fmt=jpeg)

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  ![片段八](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BoM0aqAlVqabwq6t1ZIFxG8GuV9awJU1AiaWnyddt8U8obTibVv9b0YOA/640?wx_fmt=jpeg)

# setup.s

> setup.s负责从BIOS中获取系统数据，并将这些数据放到系统内存的适当地方。此setup.s和system已经由bootsect引导块加载到内存中。

![setup](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BgRc8dxJZicruEg0QibzCfPFJ2CJGevt3icYKYeaPuDcafeKZicnZKJjibcw/640?wx_fmt=jpeg)

![片段一](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BYlp41bnLC7O1BlDXY1j3kyKfe4TACrnoiaOeYGibS7ULLrohfUwkAlUA/640?wx_fmt=jpeg)

![片段二](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BWSK6GYVlbT0AhLWhISWiaLqpmc4mvtlT26MjCZKibahiaRP3hBGmlIsyQ/640?wx_fmt=jpeg)

![片段四](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4Bic87KoggRjlDlIAPKN8nVU4Cjnqu7VdwlMR5PC0tPAZ2A6tapkdfEKQ/640?wx_fmt=jpeg)

![片段五](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BOGQvAOtNiaaCOFAeElGWvkPn5g8C9JtY3KiaiaG8p3e6cGW53zMApyicsQ/640?wx_fmt=jpeg)

![总](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbtKex8JtVx7dyJAicGoPIf4BNydVMOhKibW5eJiahL6pIyPPcfYWZtWzt8UcqKR34pC0yu0cMWHUiajuw/640?wx_fmt=jpeg)

# head.s

> 进一步设置中断描述符和全局描述符表，设计页表--开始对系统内存进行管理

















# 常见语句







# 参考资料

* 厦门大学《操作系统原理》
* 《UEFI原理与编程》

