---
title: "使用Docsify生成组件库文档"
date: 2023-04-07T10:02:39+08:00
tags: ["Docker"]
draft: false
categories: ["其他"]
---
# 一、背景
我们在写组件库的时候，都想有一个自动化的工具生成组件库文档，网上也有需要这种类似的库。最终对比下来还是docsify比较简单，而且可定制化、可扩展多一点。

docsify不受项目使用语言限制，只需项目中使用markdown来书写组件库文档即可通过docsify快速的生成组件库文档。

# 二、Docsify的使用

## 全局安装环境
```
npm i docsify-cli -g
```

## 项目中初始化
```
docsify init ./docs
```
执行完毕，生成 docs 目录。里面有3个文件：

* .nojekyll：让gitHub不忽略掉以 _ 打头的文件
* index.html：整个网站的核心文件
* README.md：默认页面
## Docsify的配置
### 主页的设置
如果不想使用README.md作为主页，这可以修改index.html手动指定homepage设置主页
``` js
    window.$docsify = {
      homepage:'home.md',
      ...
    }
```
### 侧边栏的设置
如果需要侧边栏导航，则放开loadSidebar属性，并指定相应的md侧边栏文件
```
    window.$docsify = {
      loadSidebar: 'sidebar.md',
      subMaxLevel: 3, //子菜单最大层级
      ...
    }
```
sidebar.md里的示例
```
* [首页](/)
* APCommonButton按钮组件
    * [APCommonButton](components/button/buttons.md)

* APCommonPopup弹窗组件
    * [APCommonPopup](components/popup/popup.md)
    * [APCommonPopup1](components/popup/popup1.md)
```
![](/images/docsify_1.webp)
![](/images/docsify_2.webp)

### 设置搜索
如果需求全局搜索，只需要配置search属性即可
```
    window.$docsify = {
      search: 'auto',
      auto2top: true,
    }
```
## Docsify文档的生成
设置完成之后，只需输入以下命令即可进行文档预览
```
docsify serve docs
```
## Docsify更多配置
请参考以下链接: [https://docsify.js.org/#/configuration](https://docsify.js.org/#/configuration)

# 三、结合Docker自动化输出文档
## Dockerfile
在项目根目录新建Dockerfile,代码如下：
```

#下载Node环境
FROM node:10.12.0-alpine
#作者信息
MAINTAINER guohanlin
#配置linux服务器需要的环境
RUN apk add --no-cache git bash openssh-client tzdata
#配置环境
ENV TZ Asia/Shanghai
#指定到工作目录
WORKDIR /usr/src/app/
#Docker镜像环境执行npm
RUN npm install -g docsify-cli
#拷贝代码到Docker镜像工作目录
COPY . .
#服务透出端口
EXPOSE 3000
#开始运行服务
CMD docsify serve docs
```

## 服务器生成Docker镜像并运行服务
* 生成Docker镜像
```
docker build . -t xxxxxx(你要生成的镜像名称)
```
* 运行Docker镜像，对外提供文档服务
```
docker run --name commmon_docs -p 9000:3000  xxxxxx(前面生成的镜像名称)
```
