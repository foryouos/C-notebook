---

---

###  个人网站

#### 网站框架
> 使用Github开源框架`Hexo`，拥有蛮多的优秀好看的`主题`,使用起来也十分的方便,主题选好就定下来，因为很多主题涉及的相关配置不同，仍需花费一些时间。

* [`hexo官网`](https://hexo.io/zh-cn/)
* [`hexo开发文档`](https://hexo.io/zh-cn/docs/)
* [`hexo主题`](https://hexo.io/themes/)

#### 网站服务器

> 初步方案使用`Github仓库Page功能存储`页面内容，但由于国内网速过慢，最终被弃用，也不算是完全弃用，使用了`Cloudflare`的Page，后者拥有`全球CDN`，在速度方便高于`Github`。
* [Cloudflare](https://www.cloudflare-cn.com/)
> 通过`Cloudflare`的Page功能可以直接绑定`Github仓库`，实现一建上传，`自动同步`部署Github功能，确实挺方便的。`晚上不要部署，会很慢，直到失败`

#### 资源后台备份
> 使用`Github的私有仓库`，同步所有`Blog笔记`

#### 域名解析
> 根据自身购买的域名，使用Cloudflare Page推荐的解析方式，同步解析就可以，能够实现HTTPS

#### 图片存储
> 单纯使用Github+CloudFlare方案，页面速度也可以，但是大量的图片使得进入页面的速度，以及即使加载到页面，图片未必能加载出来，速度差强人意
> 使用图床将网站所有需要的背景图，logo图片存入图床当中，只要图床的速度够快，那速度就完全没有问题

#### 图床推荐
> 根据国内访问`速度排序`由快倒慢，当然网上还有很多
* [img.tg](https://imgtg.com/)其国内访问速度，基本在1秒以内，国内`CDN`加速
* [路过图床](https://imgse.com/) 全球CDN，国内访问速度一般般
* [SM.SM](https://smms.app/)国际可以，国内一般

#### 首页轮换图
* 网上寻找壁纸`API`,调用`API`，每次返回不同的`img地址`，自己做，时间问题，还没有实现

#### 天气功能
> 使用[和风天气](https://widget.qweather.com/)插件嵌入web端

![image-20230309180534309](E:\学习笔记\Markdown\C-notebook\我的网站\assets\image-20230309180534309.png)


#### 网站
* `foryouos`[博客](https://www.blog.foryouos.cn)
> 个人博客

![image-20230309180243668](E:\学习笔记\Markdown\C-notebook\我的网站\assets\image-20230309180243668.png)


* `foryouos`[主页](www.foryouos.cn)
> 整合网站信息

![image-20230309180201706](E:\学习笔记\Markdown\C-notebook\我的网站\assets\image-20230309180201706.png)

* `foryouos`[导航栏](www.so.foryouos.cn)
> 作为和室友们的自定义导航栏(其实是被迫的，还好确实有好功能)

![image-20230309175951339](E:\学习笔记\Markdown\C-notebook\我的网站\assets\image-20230309175951339.png)

* 详见具体网站[`Github`](https://www.github.com/foryouos)