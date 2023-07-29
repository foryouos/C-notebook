# 下载`ubuntu`系统

> 在[清华大学镜像站](https://mirrors.tuna.tsinghua.edu.cn/)
>
> * 点击获取`下载`链接（`IOS`)
> * 选择你要下载的`Linux系统`

# 设置U盘

> 下载[Rufus](https://rufus.ie/zh/)

* 启动文件之后，选择自己的U盘，并选择自己下载好的`ubuntu`系统

![选择分区与文件](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtYJ63N9TX2XmJg8WuLBdkqFNeS8rE7nC5zjzlydCiad2sz3852emuh8nVEGPqGP7J8b9UJgic5bTdw/640?wx_fmt=png)

* `点击开始`

![点击OK](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtYJ63N9TX2XmJg8WuLBdkqkjNoIfTkdVfmyriccTuQUT6gR6saxdwJ3qbHbZbZWkQHHJe2U6OVH1g/640?wx_fmt=png)

* 选择`推荐`，点击`Ok`
* 将会`清理U盘`数据，点击`确认`
* `等待完成`

# 压缩磁盘文件

> * 点击`win键`
> * 输入`磁盘管理`，`确认`
> * `压缩磁盘`文件，为`Linux`提供`足够的空间`

![磁盘管理](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbvt0EZMfITp1iat7azhaibBxM9pGmgx2hRvzeL34vLMbPoLGL8IUy1HOJ6bpSt6pOAiaPXDtjLuZ2Kow/640?wx_fmt=png)

# 进入BIOS

> 通过`UEFI`固件设置进入`BIOS`

* 点击所有`设置`
* 点击`更新与安全`
* 点击立即`高级`选项
* 点击`立即重新启动`
* 点击`疑难解答`
* 点击`高级选项`
* 选择`UEFI固件`设置
* 点击`重启`

# 设置U盘启动

* 在`Boot页面`下
* 选择`Boot Device Priority ` 启动顺序
* 将`启动为第一位`的设置为自己的`U盘`
* `保存`并重新`启动`

# 启动与安装

> 选择`Ubuntu Install`
>
> 在此即进入`Ubuntu`的安装程序，在安装区间时会检测到本机`已经安装`Windows，选择时务必`选择新`的，开辟`新区`，否则会与将window替代。

# 安装EDK2

> 偷个懒，推荐两篇文章吧！也是最近的，写的挺好的

* [`UEFIEDK2安装`](https://blog.csdn.net/qq_41873192/article/details/125861472)
* [`EDK1环境搭建`](https://blog.csdn.net/weixin_45450696/article/details/131529613)
* [`Ubuntu安装`](https://blog.csdn.net/weixin_43764544/article/details/123987210)

```sh
https://blog.csdn.net/qq_41873192/article/details/125861472
https://blog.csdn.net/weixin_45450696/article/details/131529613
https://blog.csdn.net/weixin_43764544/article/details/123987210
```

# UEFI资料

> * 《`UEFI编程实战`》罗冰老师
> * 《`UEFI原理与编程`》戴正华老师
> * 《`一个UEFI引导程序的实现`》
>
> * 书籍源码在`书籍引言`都有介绍
>
> 微信公众号:`瓶子的跋涉` 回复`UEFI`
>
> 我就直接给出链接啦！觉得公众号有需要就关注，没需要就...嘿嘿！

```sh
# 第一本
链接：https://pan.baidu.com/s/1MgJ2s4VKYCTXex7UFvvrFg?pwd=5213 
提取码：5213 
# 后两本
https://foryouos.lanzoul.com/b0137vsde
密码:gevc
```



# 在win 企业版安装微软应用商城

* 进入`cmd`
* `输入`

```sh
wsreset -i
```

* 在`右`侧`通知栏`目会显示相关的`组件`正在`下载`
