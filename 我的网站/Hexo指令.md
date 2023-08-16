# install

> 首先检查node_modules目录中是否存在指定模块，如果存在就不在重新安装，即使有新版本

```sh
npm install

# 强制重新安装
npm install -f
```

# update

> 先到远程仓库查询最新版本，然后查询本地版本，更新版本

```sh
# 检查所有可更新的插件
ncu 


# 更新依赖包到最新版本
npm install 依赖包名称@latest -D

# 一键升级所有插件
npm  install -g npm-check-updates
```



[![Netlify Status](https://api.netlify.com/api/v1/badges/8e4c94d5-4596-4170-a3a0-fb348e946b8e/deploy-status)](https://app.netlify.com/sites/foryouos/deploys)





