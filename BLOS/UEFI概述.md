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
> * `BMC`通过不同的`接口`与`系统`中其它`组件连接`，`LPC`，`I2C`,`SMBUS`,`Serial`等`基本接口`
>* `IPMI`, 它是与`BMC`匹配的`接口`，所有`BMC`都需要实现这种接口

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
>
> 引导区位于`FAT32`格式分区。一般FAT32位于磁盘的最开始，大小1MB左右。
>
> * UEFI会把`FAT32的格式`都当做启动磁盘都添加到`启动菜单`中
>
> * 在启动盘`UEFI/Boot/BootX64.efi `中加载
>
> ```sh
> # 硬盘启动盘文件也都位于此
> EFI/Boot/BootX64.efi
> # 32位系统无需 + X64
> ```
>
> 

![系统启动编译结构](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wbJZbSCnZZcz7drtiaLW3VLw6APLdX3VNolxKHjNtWPZkcj5yu733vicDw/640?wx_fmt=png)

![系统内核加载架构](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wbVibicxxJmxzwcF0zuGlx1TutHicMCUugBNIJyia7zFCWb8ArG50v2oYHRg/640?wx_fmt=jpeg)

![USB重装系统过程](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wb53vgx79GQcOuszIicGdYR2Ulh6aqBeRQicM95Vw6VH0s7oDicLsSmn2gg/640?wx_fmt=jpeg)



# UEFI

> `UEFI`(`Unified Extensible Firmware Interface` ,统一可扩展固件接口)提供给`操作系统`的接口包括`启动服务`(`Boot Service` ，`BS`)和`运行时`服务(`Runtime Service`,`RT`)以及`隐藏`在`BS之后`的丰富的`Protocol`（协议)，`BS`和`RT`以`表`的形式(C语言中的`结构体`)存在。
>
> UEFI的实现分为两部分
>
> * `平台初始化`
> * `固件` - 操作系统`接口`

# 组成

> `OS Loader`可以通过`BS`和`RT`使用`UEFI`提供的服务，将计算机资源逐渐转移到自己手中，这个过程为`TSL`(Transient System Load)

![UEFi系统组成](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wbQDjicdnNibvDIr9Mo7bib7863bbV6ANP87yzsSl29G7uTptDs5kjtZoMg/640?wx_fmt=png)



> 当`OS Loader`完全`掌握`了计算机资源时，`BS`也就`完成`了它的`使命`。`OS Loader`调用`ExitBootServices()`结束`BS`并`回收BS`占用的资源，之后计算机系统进入`UEFI Runtime`阶段

## BS提供的服务(TSL阶段)

> `UEFI`提供给操作系统的`接口`，包括`启动服务`(`Boot Services` ,` BS`)

* `事件`服务：事件时`异步`操作的基础，有了事件的支持，才能够在UEFI系统内执行并发操作
* `内存`管理：提供内存的分配与释放，管理系统内存映射
* `Protocol`管理：提供了安装与卸载Protocol的服务，注册通知函数的服务
* `Protocol使用类`服务: `Protocol`的打开与关闭，查找支持Protocol控制，例打开设备上`PciloProtocol`,用`PciIo->Io.Read()`服务读取设备上的`寄存器`
* `驱动管理`：包括用于将驱动安装到控制器的`connect服务`，以及将驱动从控制器上`卸载的disconnect服务`。若启动时，需要网络服务，通过loadImage将驱动加载到内存，通过connect服务将驱动安装到设备
* `Image管理`：加载，卸载，启动和退出UEFI应用程序或驱动
* `ExitBootServices`：用于结束启动服务

## RT提供的服务

> UEFI提供给操作系统的`运行时`服务

* `时间`服务: 读取/设定系统时间，
* 读取`UEFI系统变量`：读取/设置系统变量，例如`BootOrder`用于指定`启动项顺序`，通过系统变量保存系统配置
* `虚拟内存`服务： 将`物理地址`转换为`虚拟地址`
* 其它服务：包含`重启系统的ResetSystem`,获取系统提供的`下一个单调单增值`。

# UEFI启动过程

![image-20230728095402333](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wblJ3rlN5iaZxBFvonYAPvibATfhxJO6FlJs9EkIEG9fq3icq5w0g3PFIQQ/640?wx_fmt=pngg)

![启动过程任务](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wbfhTKsXhA6WDK9qZNuVibGrvbHkyceW78cKCwE36UAxiaEBv5wyasWVpA/640?wx_fmt=jpeg)

## SEC阶段

> `Security Phase`,当CPU收到`ResetVector`信号后，开始执行`第一行`代码， 平台初始化的第一个阶段，安全阶段，`最早`运行的固件代码，此模块`相当部分`使用`汇编`语言开发。
>
> 电脑的开机或者重启，`本质`上是给CPU发送了一个`ResetVector`信号。
>
> 由于`没有初始化内`存，会`临时使用CPU`的缓存，内嵌在CPU中，初始化好的，
>
> SEC作为整个系统执行的起点，可能遇到各种异常，就需要设置`IDT`，有了中断描述符表接受异常，就能让系统遭遇意外情况时，不至于崩溃，为PEI阶段设置临时`内存的基地址和长度`并传给PEI，同时将PEI代码的入口点，将控制权移交过去，并且处理临时内存。
>
> ```sh
> # 位于EDK2如下目录
> UefiCpuPkg/ResetVector/Vtf0
> ```
>
> 

### SEC阶段任务
> * 接受并处理系统`启动`和`重启`信号，以及运行过程中的`严重异常`信号
> * `初始`化`临时存储区域` ，启动系统需要的一些临时`RAM`，空间资源仍然`稀缺`
> * 作为`可信系统`根传递给下一个`阶段PEI`

![image-20230728111306147](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wb1LbZ7X8c7pic9mKNmiaL1tCrcpMQ9pAicD6cGEnxaCBcke9WNzTdFcJBg/640?wx_fmt=png)

### Reset Vector

> * 进入`固件入口`
> * …..

## PEI阶段

> PEI : `Pre-EFI Initialization`

> * `资源`十分`有限`，`PEI后期`进行内存`初始化`
> * 为`DXE`准备执行环境，将需要传递的信息组成`HOB`列表，将控制权移交到`DXE`手中
>

* `PEI Foundation`:负责接受`SEC`发送的交换数据，并扮演`模块分发`的角色
* `PEIMs-EFI Initlization Modules`是模块化的，`PEI`阶段的4档事情就是交给`PEIMs`完成，完成后就来到了`DXE阶段`。

> > 对`系统的初始化`，找出系统中所有的`PEIM`，并根据`PEIM`之间的依赖关系按照顺序执行`PEIM`,

### PEI入口函数

* 系统当前的状态，判断`系统健康状况`
* 可启用`固件的地址`和`大小`
* 临时`RAM区域`的`地址`和`大小`
* `栈`的`地址`和`大小`

### PEI执行流程

![image-20230728112703251](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wbyfRUncfdRmpjMhWN2xoDBRjph0zgcRTwZm2NtlmfiakNWYBjF5YlzibA/640?wx_fmt=png)

> 在`PEI`阶段，`PEIM`，`PPI`，`HOM`组成了PEI阶段，PEI阶段的`module`可理解为`Driver`就是`PEIM`，`PEI阶段`就是由一个一个的PEIM组成的；`PPI`是`PEIM`之间相互调用的接口，由唯一的`GUID`(`全局统一标识符`)引导，内部也包含一些`接口`，HOB相当于信件在PEI阶段创建，会记录当前系统的信息，可以`自定义HOB`，然后在`DXE阶段读出`

![PEI阶段](https://img-blog.csdnimg.cn/670bb844aa004542bbfdc98dbc60ec6a.png)

> `PEIM` :` PEI Module`，会被编译成`efi binary`
>
> 在一套的`BIOS code` 编译完之后，进入到`build目录`就可以找到`PEIM`具体的`efi`
>
> `PPIs` ：`PEIM-to-PEIM Interfaces`,PEIMs被调用通过`PPI`，`Interface`。调用函数必须通过PPI接口
>
> * PPI名称：`GUID` (128-bits)
> * `PPIs`结构体，PPI就是一个结构体，可能包含的功能，数据。
> * PEIM会把PPI注册到`PEI Foundation`。  （`PEI Foundation`管理着庞大的`PPI`数据库)

* `Cire Services`包含后面phase用到的各种Services。在PEI阶段get当前计算机启动的boot Mode有直接定义毫的PEI Service函书，在DXE以及后面的阶段要通过HOB方式通过`get HOB LIS`T然后拆解信息进行`ge`启动`boot Mode`
* `Core Dispatcher`负责派发个`PEIMs`,将`PEIM`按照既定的顺序`Load`并执行，`Dependency`顺序，就是`inf`文件里面的`depx`, 满足条件可执行
* 各`PEIM Entry`可能使用其它`PEIM`和`PPI`
* `PEI Core`最后会找到`DXE`获得之前`phase Data`是从HOB里拿到，PEI Core会创建HOB,PEI和DXE都可以使用`HOB的Data`

> 函数:
>
> * `InstallPPI`安装PPI到`PEI foundation`,`Protocol install `安装完毕后放到`Handle Database`里
> * `LocatePPI()`：根据PPI名称`GUID`从`PEI foundation`找`Interface`
> * `NotifyPPI()` : PPI里的`function` 不会在派发时就执行，通知系统此PPI会在某个PPI被安装时才执行

### Install PPI

```c
 /**

  Install PPI services. It is implementation of EFI_PEI_SERVICE.InstallPpi.
  这是个service，PEI foundation提供的。 通过GUID安装。目的是让别人调用。
  @param PeiServices                An indirect pointer to the EFI_PEI_SERVICES table published by the PEI Foundation.
  标准格式，入口第一人参数是铁定的EFI_PEI_SERVICES指针
  @param PpiList                    Pointer to PPI array that want to be installed.
  第二个参数是PPI List, LIST里包括Flag、GUID和函数 参考.h里的EFI_PEI_PPI_DESCRIPTOR定义

  @retval EFI_SUCCESS               if all PPIs in PpiList are successfully installed.
  @retval EFI_INVALID_PARAMETER     if PpiList is NULL pointer
                                    if any PPI in PpiList is not valid
  @retval EFI_OUT_OF_RESOURCES      if there is no more memory resource to install PPI

**/
EFI_STATUS
EFIAPI
PeiInstallPpi (
  IN CONST EFI_PEI_SERVICES        **PeiServices,
  IN CONST EFI_PEI_PPI_DESCRIPTOR  *PpiList
  );
```

### HOB

> HOB(`Hand-Off Blocks`)：传输信息的载体，PEI和DXE联系薄弱，DXE需要知到PEI初始化的硬件内存等数据，HOB作为桥梁。
>
> `HOB`实际就是一个`链表`，当我们找到hoblist的头，那么整个链表的数据都能得到，`GetHobList()`获取`hoblist`的`指针`，
>
> * 第`一个HOB`总是`PHIT==Phase Handoff GetHobList(),`里面是`boot mode`
> * 其它`HOB`可能出现在`List任意位置`，最重要的是`System Memory HOB & Firmware Volumes`，`HOB`列表总是会以`END_OF_HOB_LIST`结束

![HOB链表](https://img-blog.csdnimg.cn/a4c840de61d243c8ae52613688bda033.png)

#### 添加新的HOB

```c
/**
  Add a new HOB to the HOB List.

  @param PeiServices        An indirect pointer to the EFI_PEI_SERVICES table published by the PEI Foundation.
  @param Type               Type of the new HOB.
  @param Length             Length of the new HOB to allocate.
  @param Hob                Pointer to the new HOB.

  @return  EFI_SUCCESS           Success to create HOB.
  @retval  EFI_INVALID_PARAMETER if Hob is NULL
  @retval  EFI_NOT_AVAILABLE_YET if HobList is still not available.
  @retval  EFI_OUT_OF_RESOURCES  if there is no more memory to grow the Hoblist.

**/
EFI_STATUS
EFIAPI
PeiCreateHob (
  IN CONST EFI_PEI_SERVICES  **PeiServices,
  IN UINT16            Type, //对于自定义的HOB 一般使用EFI_HOB_GUID_TYPE
  IN UINT16            Length,
  IN OUT VOID          **Hob
  );
```



## DXE阶段

> `Driver Execution Environment` 驱动执行环境，主要任务是把基本驱动程序加载起来，建立两者之间的联系。
>
> 执行`大部分系统初始化`的工作，此阶段内存已经`完全`被`初始化`，
>
> * `DXE内核`：复杂DEX基础服务和执行流程
> * `DXE派遣器`: 负责调度执行`EXE驱动`，初始化系统`设备`

![DXE阶段执行流程](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbuvDicPdvq9q8kCkj2w7H9wbS0m2x5v1ZMRKOBXZQfskLSrVBlUPNq1emzZ58ohcW3pnpt4v6goDcg/640?wx_fmt=png)

```c
typedef EFI_STATUS(EFIAPI *EFI_IMAGE_ENTRY_POINT)
(
    IN EFI_HANDLE ImageHandle,
    IN EFI_SYSTEM_TABLE *SystemTable
)
```

> 驱动之间通过`Protocol`进行通信。
>
> * `Protocol`特殊结构体保存着对应的`GUID`，利用系统`BootServices`的`OpenProtocol`，并根据`GUID`打开对应的`Protocol`，进而使用`对应的服务`。当所有的`driver`执行`完毕`，系统`完成初始化`，

## DXE Core

> `DxeMain()`是DXE阶段执行的`主函数`，同时以`参数形式`接受PEI阶段的`HOB表`
>
> * 创建`EFI System Table`在随后的`DXE Drive`r中逐步完善table
> * 生成`Boot Services / Run Time Services / DXE Services`
> * 调用`Dispatcher`，所有的DXE Driver在这个函数中被检测并执行
> * `Driver`执行完毕之后，执行特殊的`DXE Driver`进而进入BDS阶段

## DXE Dispatcher
> * 去`BIOS`芯片中搜寻`DXE Driver`
> *  检测并按照相应的顺序执行所有的`DXE Driver`,在每个`driver`的`inf`文件的`driver`依赖条件都成立时，该driver才被执行

![DXE Dispatcher](https://img-blog.csdnimg.cn/20210520145140677.png)


## BDS阶段

> 全程：`Boot Device Select`
>
> U盘就是寻找具有`FAT32分区`的设备
>
> 执行启动策略BDS三大任务：`console初始化`，`Driver初始化`，`BootDeviceSelect`：用户选择`BDS加载`启动选项里的`OS loader`，最后移交真正的`控制权`给`OS loader`，由OS Loader将
>
> * 初始化控制台设备:查看系统有`多少`加载必要的设备驱动：启动所有检测到的设备，加载`driver`
> * 根据系统设置加载和执行启动项(若加载失败，系统将重新执行`DXE dispatcher`以加载更多的`驱动`，然后重新尝试加载`启动项`)

![DXE执行流程](https://img-blog.csdnimg.cn/48969997637c402fbaa2c50dcf92f9a2.png)

## BDS Steps

* 初始化`语言`和`字符串`数据库
* 获得`当前启动模式`
* 基于`启动模式`建立`设备清单`
* 连接`设备`
* 检测`input output`设备
* 执行`内存测试`
* 进程`引导选项`

![BDS Steps](https://img-blog.csdnimg.cn/86e9844f11dc4525b87d6aeca353d5ae.png)

## TSL阶段

> `Transient System Load`: 操作系统`OS Loader`执行的第`一阶段`，首先作为`UEFI`程序运行，之后TSL退出，系统进入`Run Time`阶段。OS loader的主战场，TSL是正式操作系统加载前的预备阶段，需要Loader找到并加载OS

## RT阶段

> `Run Time` : 系统控制权从`UEFI`内核转交到`OS Loader`手中，UEFI资源回收到OS Loader。在OS Loader中OS获取系统控制权。

## AL阶段

> `After Life` 如果系统/软件遇到灾难性错误，系统固件需要提供错误处理和灾难性恢复机制，此机制运行在`AL`(After Life)阶段。
>
> 由`常驻UEFI`驱动组成，计算机`关机`休眠`睡眠`重启过程中的系统信息都会在这一阶段`保存`。
>
> 

# 源码部分基础

## 源码类型定义

```c
typedef unsigned __int64  UINT64;
typedef __int64           INT64;
typedef unsigned __int32  UINT32;
typedef __int32           INT32;
typedef unsigned short    UINT16;
typedef unsigned short    CHAR16;
typedef short             INT16;
typedef unsigned char     BOOLEAN;
typedef unsigned char     UINT8;
typedef char              CHAR8;
typedef signed char       INT8;
```

# 一些硬件补充

## Hardware Monitor

> 读出所有计算进访问`传感器`的`测试值`
>
> * 不同地点的`temperature`读数`CPU and system temperature`
> * `智能风扇`控制：风扇转速侦测和风扇控制输出
> * `电压监控`

## 分时复用

> 是采用`一物理链接`的不同时段来传输不同`的信号`，能达到`多路复用`的目的。通过事件上`交叉`发送每一路信号的一部分来实现一条电路传送多路信号。
>
> 将整个传输时间分割为`互不重叠`的时间间隔，又称为`时隙`
>
> * 同步分时复用（`STDM`，`Synchronous Time Division Multiplexing`）：采用`固定间隙`分配方式，即将传输信号按特定长度连续地`划分特定`的时间段或者一个`周期`
> * 异步分时复用（[ATDM](https://baike.baidu.com/item/ATDM?fromModule=lemma_inlink)，`Asynchronous Time Division Multiplexing`）：根据用户市级需要动态分配资源的分时复用记数。

## PCI

> * 局部总线：局部总线是在`ISA`和`CPU总线`之间添加`一级总`线或`管理层`。这样可将一些`高速外设如图形卡`，硬件控制器等从ISA总线上卸下而通过局部总线直接挂接在CPU总线上，使之余高速能与CPU总线相匹配。
> * PCI  (Peripheral Component Interconnect) ：` Intel 1991`年推出的用于定义局部总线的标准。`PCI`不同于ISA总线，PCI数据地址总线于数据总线是分时复用。以方便可以节省接插件的管脚数，另一方便便于实现数据传输。

## USB

> `USB总线`提供`中低速率`外围设备的扩充能力，像键盘，鼠标，遥感，喇叭，麦克风等设备，只要是USB接口设计，就可以以`热拔插`（Hot Plug)的方式，直接跟计算机连接或拆除(离线)，计算机与`OS会自动检测`并启用/禁用该设备，达到真正的即插即用。
>
> 新近的`BIOS`直接提供了`USB设备驱动`与`读写`功能，比如开始就可以使用`USB键盘`，`鼠标`以及`USB软盘`，硬盘甚至`USB CD-ROM`来开机。

## ACPI

> 高级配置和电源管理接口：`Advance Configuration and Power Management Interface` .早先ACPI将电源管理几乎全部分配给了`BIOS控制`，限制了`操作系统`在控制电脑。系统可能进入`极地功耗`消耗状态，这些就是可利用多数桌面型电脑上睡眠和休眠设置
>
> 节点方式：
>
> * 显示屏`自动断电`
> * 系统把当前信息存储在`内存`中，只有内存等几个关键部件通电，即挂起到内存
> * 挂起到硬盘，计算机自动关机，关机千将当前数据存储在硬盘上。

## 中断向量表

> `中断向量表`在`内存`中保存，其中放着`256个中断源`所对应的中断处理程序入口

![什么是中断向量表](https://i0.hdslb.com/bfs/article/75ac548ac5423238aed9cc55ba33c4d7dfca13db.png@1256w_1292h_!web-article-pic.avif)

## 英语

* `Keyboard Power On` :键盘开机
* `Wake on LAN`  ： 网卡遥控开机
* 调制解调器/传真机来电开机（`Modem Ring On`)
* CPU过热防护：` CPU Overheat Protection`
* 超频功能：`Overclocking`

# 参考资料

* [什么是BMC](https://blog.csdn.net/qq_32907195/article/details/116450404)
* [gxh1992博客](https://blog.csdn.net/wmx1992?type=blog)
* 《UEFI原理与编程》
* [PEI阶段扩展](https://blog.csdn.net/weixin_45279063/article/details/120194301)
* [DXE](https://blog.csdn.net/weixin_45279063/article/details/115508961)
* [图表化呈现](https://blog.csdn.net/qq_41873192/article/details/125966457)