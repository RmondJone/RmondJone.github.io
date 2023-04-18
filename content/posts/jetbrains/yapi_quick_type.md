---
title: "YapiQuickType插件使用说明"
date: 2023-04-07T10:04:45+08:00
tags: ["JetBrains插件"]
categories: [ "JetBrains插件"]
draft: false
---
# YApi QuickType ![Downloads](https://img.shields.io/jetbrains/plugin/d/com.guohanlin.yapiquicktype)

## 一、业务背景
日常开发中，我们会用到YApi这个工具，YApi 是一个可本地部署的、打通前后端及QA的、可视化的接口管理平台，可以进行接口定义以及接口模拟的一些操作。

有关更多的YApi使用教程，可以参考[YApi官网](https://github.com/YMFE/yapi)，这里就不再过多的赘述。

本插件是一个基于YApi开源接口之上的一个编码工具插件，主要用于接口定义实体代码的快速生成，只要在插件设置选项中进行简单的配置，即可一键快速生成多种你想要的语言接口定义实体代码。

目前支持生成的语言有：Kotlin、Java、TypeScript、Dart、Swift、Objective-C、Go、C++

## 二、插件下载以及配置
### 下载

直接插件市场搜索Yapi QuickType进行下载

![](/images/yapi_quicktype_1.webp)

### 插件配置

![](/images/yapi_quicktype_2.webp)

* **YApi根路径填入自建的地址**
* **配置你需要的项目的Id、Token**
  这些都可以在YApi项目配置里找到，找到复制填入即可，项目名称可以随意填写。
* **重启IDE使配置生效**
  
![](/images/yapi_quicktype_3.webp)

![](/images/yapi_quicktype_4.webp)

## 三、插件的使用以及效果

### 插件的使用也非常简单，只需要在你想要生成代码的目录右键即可

![](/images/yapi_quicktype_5.webp)

### JSON代码生成插件：粘贴复制的JSON字符串、输入生成的实体名称、选择想要生成的语言点击OK生成代码
![](/images/yapi_quicktype_6.webp)

### YApi代码生成插件：选择你配置的项目的对应接口和想要生成的语言
![](/images/yapi_quicktype_7.webp)

### 生成代码效果
![](/images/yapi_quicktype_8.webp)

如果接口请求不通，你可以尝试本地搭建QuickTypeNode服务，然后服务地址配置到插件中即可 [QuickTypeNode](https://github.com/RmondJone/QuickTypeNode)

## 三、使用反馈

QQ群：[264587303](https://jq.qq.com/?_wv=1027&k=96R8fd5v)

![](/images/qq_ercode.jpeg)

Telegram: [https://t.me/+9DYAumdLYwxmOTNl](https://t.me/+9DYAumdLYwxmOTNl)

![](/images/tg_ercode.jpeg)

[English document](https://plugins.jetbrains.com/plugin/18847-yapi-quicktype/documentation)