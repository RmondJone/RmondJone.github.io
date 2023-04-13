---
title: "1200RMB闲鱼完美黑苹果游戏本 --拯救者14"
date: 2023-04-13T18:56:06+08:00
draft: false
categories: ["黑苹果"]
tags: ["黑苹果"]
---

##  一、硬件配置

配置|名称
--|--
CPU|i7-4720HQ
显卡|HD Graphics 4600
硬盘| 西部数据 1T SSD+希捷1T 机械硬盘
内存|金士顿DDR3 8GB*2
有线网卡| Realtek RTL8111H
无线网卡/蓝牙| BCM943252Z（淘宝买自己动手换）

## 二、黑苹果完美成果以及现状

环境|版本
--|--
OpenCore|0.6.5
MacOS |11.6.1

已驱动：

* 核显完美
* 声卡完美
* 网卡：有线+无线完美
* SIP关闭
* 睡眠正常
* CPU、温度传感器正常
* CPU变频正常
* Siri正常
* 蓝牙正常，可以完美使用AirPods
* 隔空投送正常

已知问题后续会持续修复，敬请期待！

![](/images/lenovo_1.webp)
![](/images/lenovo_2.webp)
![](/images/lenovo_3.webp)
![](/images/lenovo_4.webp)
![](/images/lenovo_5.webp)

## 三、安装教程
最好手头上有一个苹果电脑，如果没有苹果电脑也可以用WinPE，后面会说怎么弄。

### 使用BalenaEtcher烧录镜像

这里没啥好说的，准备一个16G的U盘就行，镜像地址文章最后会贴出来。烧录完，重新插拔一下就可以看到制作好的U盘启动盘了。

![](/images/lenovo_6.webp)

### 使用OpenCore Configurtor挂载U盘启动盘的EFI

![](/images/lenovo_7.webp)

### EFI挂载完成之后，把我提供的EFI直接复制到EFI分区

![](/images/lenovo_8.webp)

Tip: 如果你没有MAC系统环境，也可以用WinPE，然后使用DiskPart命令挂载U盘启动盘的EFI分区也是可以的。具体挂载命令，网上自己搜。

![](/images/lenovo_9.webp)

### BIOS设置

首先开机按F2键，进入BIOS界面进行如下设置：
* 调到Boot选项，调整Boot Mode选项为Legacy Support,
* 调整Boot Priurity为UEFI  Frist
* 调到最后一项选择有个选项调整选择Other OS
* 调到第二个选项卡里有一项是调整为AHCI，还有一个开启CPU虚拟技术。

### U盘插电脑上，设置为U盘启动等就完事了

这里的建议是操作这一步之前，最好把电脑里的重要的文件先做一个云端的备份。然后最好是整盘格式为APFS，尽量不要用混合的分区来装黑苹果。
另外安装的过程中不要断电，会反复安装3次。第三次安装完应该就可以进入系统了。
* 第一次选Install macOS Big Sur这个选项
  
  （1）第一次安装会进入到一个macOS实用工具这个页面，然后选择磁盘工具，把你想要安装的盘抹掉格式化就行，一定要选APFS格式，别选错了。
  
  （2）格式化完回到macOS实用工具，选择安装系统，一直下一步选择你刚刚格式化的那个盘安装系统。
* 第二次选macOS Install这个选项
  等待差不多30分钟这个样子
* 第三次选OS这个选项
  等待差不多10分钟这个样子

### 迁移U盘EFI文件到安装盘EFI分区

这里也没有什么好说的，就是用前面的OpenCore Configurtor工具挂载U盘和安装盘的EFI分区，复制一下就完事了

## 四、资源地址

EFI地址：[https://github.com/RmondJone/Lenovo-Saviour14-i7-4720HQ](https://github.com/RmondJone/Lenovo-Saviour14-i7-4720HQ)

OS镜像：[https://www.aliyundrive.com/s/Bi8yVdtz5Nq](https://www.aliyundrive.com/s/Bi8yVdtz5Nq)

BalenaEtcher:[https://www.balena.io/etcher](https://www.balena.io/etcher/)
