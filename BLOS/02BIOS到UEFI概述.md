# BIOS阶段划分

* `SEC`: `Security`(安全) ： 处理`平台重`事件，创造一个`临时的内存`区(此时内存还`未初始化`)，在系统中作为一个可信的root，传递信息到`PEI`
* `PEI `: `pre-efi initialization`(预EFI初始化)：`初始化`一些`永久的内存`HOBS(`hand-off BLocks`)中内存，以及在HOBS里面的`FV`(firmware volume)位置，传递`控制权到DXE`阶段
* `DXE `：`driver execution environment`(驱动程序执行环境) ：`重点`关注，服务器上`硬件驱动的执行环境`，与`后期外设`的使用，有极大的关系
* `BDS `: `boot device select` (引导设备选择) 初始化`console设备`；加载设备驱动；尝试`加载和执行启动项`。
* `RT` ：` run time service` (运行时服务) ：此时与`bootloader`关系`紧密`
* `AL` : `after life` (`transition from the os back to the environment`) `of system`

# 与`BMC`通信

## 基础了解

### 平台管理

> * 平台管理(`platform managemet)` : 平台管理表示一系列的`监视`和`控制`功能，`操作的对象`是`系统硬件`，比如监视系统的`温度`，`电压`，`风扇`，`电源`等，并`做响应的`调节工作，以保证系统处于`健康状态`。若系统`不正常`，通过`复位方式`重启
>
> ![平台管理](https://img-blog.csdnimg.cn/20200510114843136.png)
>
> * `BMC` : `Baseboard Management Controller)`： 把上面的功能集成到一个`控制器`上，称为`基板管理控制器`，`BMC`是一个独立的系统，它不依赖系统上的`其它硬件`(如`CPU`,`内存`)，也不依赖于`BIOS`,`OS`等，但`BMC`可以与`BIOS`和`OS`交互。BMC本身也是一个`带外处理器`(一般都是ARM处理器)的小系统，单独用来处理某些工作，其中重点还是平台管理
>
> ![image-20230724141424783](E:\学习笔记\C-notebook\BLOS\assets\image-20230724141424783.png)
>
> * `BMC`通过不同的`接口`与`系统`中其它`组件连接`，`LPC`，`I2C`,`SMBUS`,`Serial`等`基本接口`
> * `IPMI`, 它是与`BMC`匹配的`接口`，所有`BMC`都需要实现这种接口

### `IPMI`

> `IPMI`全称`Intelligent Platform Management Interface` 智能平台`管理接口`，`IPM`是对`平台管理`概念的`具体规范`，该规范定义了`平台管理`的软硬件架构，`交互指令`，`事件格式`，`数据记录`，`能力集`等，而`BMC`是`IPMI`中的一个`核心`部分，属于`IPMI`硬件架构。

![IPMI智能平台管理接口](https://img-blog.csdnimg.cn/20200510131422340.png)

## BIOS通信

> `BIOS`与`BMC`之间的通信，主要使用`IPMI`。共有`两个`阶段`PEI`和`DXE`(包括`DXE`后面的)，用的是不同的`IPMI`函数(重点`注意`)
>
> 虽然后使用`IPMI`，但使用`两个`通道分别是`KCS`，`BT`，一般`使用KCS`通道。
>
> `BMC不会`主动与BIOS通信，BIOS会发送`IPMI命`令给`BMC`,`BMC`如果`成功接收`的话，就会有个`返回信息`给`BIOS`。

## 如果`BMC`与`BIOS`产生了通信故障

* 确认`BIOS`是否发送了`IPMI命令`给`BMC`,可以查看`BMC`返回的`completion code`;
* 确认`BMC`是否收到了`BIOS`发送的`IPMI命令`
* 如果`BIOS`发送了`IPMI命令`，但是`BMC未接收`：可能是`BMC`的`IPMI进程`正处于`忙碌状态`，所以`丢掉`了这条`IPMI命`令(BIOS如果发送失败，可以`尝试多次`发送；另外可以`稍微调高KCS通道`的`延时`)；当然，也有可能是BMC的IPMI进程挂了
* （极少见) `一条IPMI`命令通常设计`2个底层`实现函数，`SendDataToBmcPort()`和`ReceiveBmcDataFormPort()`。接收时，BIOS从`KCS`的`I/O端口`读取`数据`，读完后，会`检测KCS寄存器`中`OBF状态寄存器`，如果`BMC没有写`数据，就会`计数-1`，`延时5ms`，然后重试，当`BMC`一直不写数据时，`计数到0`,就认为`BMC有故障`，返回`Device Error`;

# `ACPI`

> `Advanced Configuration and Power Interface` 简称`ACPI`，`高级配置`与`电源接口`，提供操作系统应用程序管理所有电源管理接口。
>
> 对于BIOS而言，`ACPI`最直观的就是`电源功耗`，从而`影响CPU性能`，

# `OS Loader`

> 操作系统加载器：`引导`加载`程序`
>
> `OS loader`可以通过`BS(Boot Services)`和`RT`使用`UEFI`提供的服务，并且将`计算机的资源`转移到`自己手`中，此过程称为`TSL(Transient System Loader) `。在此阶段结束之后，`OS Loader`会调用`ExitBootServices()函数`，结束`BS`并且`回收BS占用的资`源，然后`BIOS`会进入`RT阶段`，然后`OS loader`会加载`操作系统内核`，逐渐进入OS，此过程称为`TSL`(`Transient System Load`)
>
> 在`BDS`阶段，`BIOS`会选择`可启动项`，按照`设置的顺序`，`逐一`尝试，经过`校验之后`，加载`OS Loader`
>
> ` BIOS`（`BDS`阶段）---->`OS Loader`---->`OS`

# UEFI

> `UEFI`(`Unified Extensible Firmware Interface` ,统一可扩展固件接口)提供给`操作系统`的接口包括`启动服务`(`Boot Service` ，`BS`)和`运行时`服务(`Runtime Service`,`RT`)以及`隐藏`在`BS之后`的丰富的`Protocol`（协议)，`BS`和`RT`以`表`的形式(C语言中的`结构体`)存在。
>
> UEFI的实现分为两部分
>
> * `平台初始化`
> * `固件` - 操作系统`接口`

# 组成

![EFI系统组成](E:\学习笔记\C-notebook\BLOS\assets\image-20230724203111459.png)



> 当`OS Loader`完全`掌握`了计算机资源时，`BS`也就`完成`了它的`使命`。`OS Loader`调用`ExitBootServices()`结束`BS`并`回收BS`占用的资源，之后计算机系统进入`UEFI Runtime`阶段

## BS提供的服务

> `UEFI`提供给操作系统的`接口`，包括`启动服务`(`Boot Services` ,` BS`)

* 事件服务：事件时异步操作的基础，有了事件的支持，才能够在UEFI系统内执行并发操作
* 内存管理：提供内存的分配与释放，管理系统内存映射
* Protocol管理：提供了安装Protocol与设在，注册通知函数的服务
* Protocol使用类服务: Protocol的打开与关闭，查找支持Protocol控制，例打开设备上PciloProtocol,用PciIo->Io.Read()服务读取设备上的寄存器
* 驱动管理：包括用于将驱动安装到控制器的connect服务，以及将驱动从控制器上卸载的disconnect服务。若启动时，需要网络服务，通过loadImage将驱动加载到内存，通过connect服务将驱动安装到设备
* Image管理：加载，卸载，启动和退出UEFI应用程序或驱动
* ExitBootServices：用于结束启动服务

## RT提供的服务

> UEFI提供给操作系统的运行时服务

* 时间服务: 读取/设定系统时间，
* 读取UEFI系统变量：读取/设置系统变量，例如BootOrder用于指定启动项顺序，通过系统变量保存系统配置
* 虚拟内存服务： 将物理地址转换为虚拟地址
* 其它服务：包含重启系统的ResetSystem,获取系统提供的下一个单调单增值。

# UEFI启动过程









# 参考资料

* [什么是BMC](https://blog.csdn.net/qq_32907195/article/details/116450404)
* [gxh1992博客](https://blog.csdn.net/wmx1992?type=blog)
* 《UEFI原理与编程》