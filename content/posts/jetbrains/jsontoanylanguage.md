---
title: "JsonToAnyLanguage插件使用说明"
date: 2023-04-14T15:16:53+08:00
draft: false
categories: ["JetBrains插件"]
tags: ["JetBrains插件"]
---
# JsonToAnyLanguage ![Downloads](https://img.shields.io/jetbrains/plugin/d/com.guohanlin.JsonToAnyLanguage)

## 一、业务背景

日常开发中，我们经常要使用插件来实现JSON转业务实体，但是目前插件市场没有一款插件可以同时支持生成多种语言，导致需要下载很多个插件，很多强迫症患者是无法接受的。

基于这个背景，我开发了这个插件，统一把这些功能集成到一个插件中去

## 二、插件下载以及配置

### 下载

直接插件市场搜索JsonToAnyLanguage进行下载

![](https://plugins.jetbrains.com/files/19297/screenshot_27a37d61-0a38-4141-b521-22ffb1d55288)

### 插件配置

默认可以不用配置插件，只有当请求QuickTypeNode服务失败时，你可以本地自建QuickTypeNode服务，然后把访问链接配置到设置中

![](https://plugins.jetbrains.com/files/19297/screenshot_547d2d5a-c569-4f3e-960f-ae33c89c2bec)

## 三、插件的使用以及效果

插件使用上也非常简单，只需在你想要生成代码的目录上右键即可调用起插件菜单，然后在插件弹窗里粘贴Json字符串、选择你想要生成的语言即可

![](https://plugins.jetbrains.com/files/19297/screenshot_b38f20c2-a6e7-4d79-9a94-a3c5022be46c)
![](https://plugins.jetbrains.com/files/19297/screenshot_db4dfa14-8bd3-4c2d-b43f-aa403691ffbf)
![](https://plugins.jetbrains.com/files/19297/screenshot_98f07641-329e-46e4-84dc-d7941d9f0046)

## 四、注意事项

如果Node服务访问不通，你可以尝试本地搭建QuickTypeNode服务，然后服务地址配置到插件中即可 

[QuickTypeNode](https://github.com/RmondJone/QuickTypeNode)

## 五、使用反馈

如果你使用了YApi，这里推荐另一个更好用的JetBrains插件：[YApi QuickType](https://plugins.jetbrains.com/plugin/18847-yapi-quicktype)

使用反馈：[https://rmondjone.github.io/posts/jetbrains/jsontoanylanguage/](https://rmondjone.github.io/posts/jetbrains/jsontoanylanguage/)


QQ群：[264587303](https://jq.qq.com/?_wv=1027&k=96R8fd5v)

![](/images/qq_ercode.jpeg)

Telegram: [https://t.me/+9DYAumdLYwxmOTNl](https://t.me/+9DYAumdLYwxmOTNl)

![](/images/tg_ercode.jpeg)

## 六、捐赠

您的支持是我维护更新插件的动力，捐赠地址：[捐赠](https://rmondjone.github.io/%E5%85%B3%E4%BA%8E%E6%88%91/)

[English document](https://plugins.jetbrains.com/plugin/19297-jsontoanylanguage/documentation)
