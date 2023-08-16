# 迁移shokaX
> 原因 ：shoka好久没有更新啦，bug逐渐呈现出来啦

## 迁移步骤
* 新建目录，并CMD进入目录
* 修改npm的镜像(根据自身安装插件的速度)

```sh
# 修改镜像
npm config set registry http://registry.npm.taobao.org/
# 换回默认镜像
npm config set registry https://registry.npmjs.org/
```



* hexo 初始化

```sh
hexo init
```

![image-20230612160705654](E:\学习笔记\C-notebook\我的网站\assets\image-20230612160705654.png)

* 安装shokax-cli

```sh
npm install shokax-cli --location=global
```

![image-20230612160852937](E:\学习笔记\C-notebook\我的网站\assets\image-20230612160852937.png)

* 安装shokaX主题(建议安装npm主题，因为设计更多的自定义部分)

> 使用SXC主题目录会存到`node_modules`当中，使用`GitHub的安装`，主题文件会存到`themes`当中

```sh
SXC install -r=github shokaX
```

![image-20230612161939850](E:\学习笔记\C-notebook\我的网站\assets\image-20230612161939850.png)

* 主题克隆成功，手动安装所需要的插件

```sh
# 必须的
# npm i hexo-theme-shokax # themes已经有了，无需再次下载(此指令会下载到node_modules目录下，引起重复报错)

# 安装此指令，需要卸载系统默认的
npm i hexo-renderer-multi-next-markdown-it
npm i hexo-autoprefixer
npm i hexo-algoliasearch
npm i hexo-feed
# SEO需要的

```



