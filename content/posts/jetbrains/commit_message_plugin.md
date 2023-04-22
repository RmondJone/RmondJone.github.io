---
title: "Commit Message 插件使用说明"
date: 2023-04-14T15:59:25+08:00
draft: false
categories: ["JetBrains插件"]
tags: ["JetBrains插件"]
---

# Commit-Message-Plugin ![Downloads](https://img.shields.io/jetbrains/plugin/d/com.rmondjone.commit_plugin)
提交信息模板规范化插件

## 一、代码提交规范化的目的

* 为了部门提交代码信息格式规范化
* 为了更好的追溯代码、筛选
* 为了更加快速的定位提交代码所涉及的范围和实现功能
* 为了后续代码的Review、自动生成ChangeLog

## 二、代码提交信息规范模板

本模板修改自《Angular代码提交规范》,分为Header、Body两大块内容，去除Footer，这一部分我们并不常用
```java
[Bug修复](4.27.10):http://jira.xyz.cn/browse/XYZDEV-9043

原因分析:需求内容不完整或错误
```
### Header

**1、提交类型**

* 【新增功能】-【新的功能点、新的需求】
* 【Bug修复】-【修复的Bug:现网发散Bug、测试阶段的Bug、验收阶段的Bug】
* 【文档修改】-【只是修改了文档:注释、README.md等】.
* 【样式修改】-【不影响代码功能的修改:CSS样式、代码格式化等】
* 【代码重构】-【代码更改既不修复错误也不添加功能】
* 【性能优化】-【代码更改可以提高性能】
* 【测试代码】-【添加缺失测试或更正现有测试】
* 【编译代码】-【影响构建系统或外部依赖项的更改:build.gradle、package.json、Podfile等】
* 【持续集成】-【我们的CI配置文件和脚本的更改:Jenkinsfile等】
* 【回退更改】-【代码回退提交更改】
* 【其他提交】-【除以上所有类型之外的提交更改】

**2、涉及范围**

这里我们以版本号划分范围，此次提交代码所涉及到的发布版本。如果涉及多个版本则以4.14.10~4.27.10表示

**3、简要描述**

（1）如果提交类型是【Bug修复】,则简要描述直接填写Bug的JIRA或者Sentry链接

（2）如果提交类型是【新增功能】,则简要描述填写对应需求的JIRA链接或需求的详细描述，如需求过于庞大，则应拆分成小的功能点提交代码，便于Review人员审核,也有利于Bug的回溯。

（3）如果提交类型是其他类型，则简要描述根据你的理解，尽量用简短的文字描述出此次代码提交的目的

### Body

**1、详细描述**

这边的详细描述，力争语句表达清晰此次提交的代码具体涉及的功能点、修改、原因分析等，如功能点很多则应使用序号列出代码所涉及的功能点。如下所示

```java
1、安心管选择对象页修改
2、家庭成员、车辆、房屋的列表、编辑、新增页修改
3、安心管关系对象重定义
```

## 三、IDEA 插件集成以及使用

**插件下载**

为了规范成员的提交代码信息，我新写了一个IDEA插件来帮助快速生成代码提交信息模板。插件市场直接搜索Commit-Message-Plugin 

![](/images/commit_plugin_1.png)

**插件使用** 

在提交弹窗中点击小飞机图标唤起提交信息填写弹窗,填写相应的信息提交即可生成提交信息

![](/images/commit_plugin_2.png)
![](/images/commit_plugin_3.png)

**自定义提交模版和类型**

本插件支持修改自己的提交模版，但是固有的几个变量`@Type`、`@Scope`、`@ShortDescription`、`@LongDescription`是不可以更改的，输入@会自动补全提示。

![](/images/commit_plugin_4.png)

## 四、使用反馈

QQ群：[264587303](https://jq.qq.com/?_wv=1027&k=96R8fd5v)

![](/images/qq_ercode.jpeg)

Telegram: [https://t.me/+9DYAumdLYwxmOTNl](https://t.me/+9DYAumdLYwxmOTNl)

![](/images/tg_ercode.jpeg)

## 五、捐赠

您的支持是我维护更新插件的动力，捐赠地址：[捐赠](https://rmondjone.github.io/%E5%85%B3%E4%BA%8E%E6%88%91/)


[English document](https://plugins.jetbrains.com/plugin/12256-commit-message-create/documentation)
