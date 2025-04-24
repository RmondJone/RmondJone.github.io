---
title: "Android开机流程详解"
date: 2025-04-24T17:14:44+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

## Android系统启动流程详解

Android系统的启动流程是一个复杂的过程，涉及从硬件初始化到应用程序加载的多个阶段。下面我将详细描述Android系统的完整启动流程。

![](/images/android_launcher.png)

### 1. Boot ROM阶段（上电启动）

当用户按下电源键后，Android设备启动的第一阶段是Boot ROM代码执行：

1. **CPU复位**：电源键按下后，CPU是最后被复位的器件（防止CPU先于I/O设备或内存初始化导致失败）
2. **执行ROM代码**：CPU复位后程序计数器(PC)指向ROM的零地址，开始执行固化的启动代码
3. **Bootloader加载**：ROM代码初始化部分硬件并加载Bootloader到RAM中

Boot ROM根据存储介质不同有两种启动方式：
- **NOR Flash启动**：支持字节寻址，可直接执行ROM中的Bootloader
- **NAND Flash启动**：不支持字节寻址，需先将Bootloader复制到RAM再执行

### 2. Bootloader阶段

Bootloader是Android系统启动前运行的小程序，主要功能包括：

1. **硬件初始化**：初始化显示器、键盘、控制台等硬件设备
2. **内存设置**：建立内存映射，为内核启动准备环境
3. **内核加载**：找到并加载压缩的Linux内核镜像(zImage)到内存
4. **安全验证**：检查内核的数字签名（厂商通常会锁定Bootloader）

Bootloader通常分为两个阶段：
- **IPL(Initial Program Load)**：检测外部RAM并加载SPL
- **SPL(Second Program Load)**：初始化更多硬件，加载内核并转移控制权

### 3. Linux内核启动阶段

内核启动后主要完成以下工作：

1. **内核初始化**：
    - 设置缓存、受保护内存、调度列表
    - 创建异常向量表和初始化中断处理函数
    - 初始化内存管理和进程通信机制

2. **驱动加载**：
    - 静态加载：编译进内核的驱动自动加载
    - 动态加载：通过LKM(Loadable Kernel Module)机制加载，如Binder驱动

3. **Android特有定制**：
    - Binder：Android特有的IPC机制
    - Ashmem：Android共享内存
    - Logger：支持logcat日志
    - Wake locks：电源管理机制

4. **启动init进程**：内核最后会在系统文件中查找并启动init进程(pid=1)

### 4. Init进程阶段

Init是Android系统的第一个用户空间进程，主要职责包括：

1. **挂载文件系统**：如/sys、/dev、/proc等目录
2. **初始化属性服务**：在共享内存中初始化和存储系统属性
3. **解析init.rc脚本**：执行Android初始化语言(AIL)定义的命令

init.rc文件主要包含四种类型声明：
- **Actions**：以触发器决定是否执行的动作序列
- **Services**：init进程启动的程序，退出时可自动重启
- **Commands**：执行的命令
- **Options**：服务的描述选项

4. **启动关键服务**：
    - 启动ServiceManager(binder服务管家)
    - 启动bootanim(开机动画服务)
    - 启动ueventd、logd等守护进程

5. **孵化Zygote进程**：通过解析init.zygoteXX.rc文件启动Zygote

### 5. Zygote进程阶段

Zygote(孵化器)是Android系统的第一个Java进程，主要功能：

1. **创建虚拟机**：初始化Dalvik/ART虚拟机
2. **预加载资源**：
    - 预加载Java类(/frameworks/base/preloaded-classes)
    - 预加载资源(主题、布局等)
3. **建立Socket服务**：注册服务器套接字等待AMS请求
4. **启动SystemServer**：通过fork方式创建系统服务进程

Zygote的设计解决了两个关键问题：
- **快速启动**：通过预加载共享库减少应用启动时间
- **内存优化**：通过COW(Copy-On-Write)机制共享只读内存

### 6. SystemServer进程阶段

SystemServer是Zygote孵化的第一个进程，负责启动和管理所有Java框架层服务：

1. **初始化主线程**：创建MainLooper和SystemServiceManager
2. **启动引导服务**：
    - ActivityManagerService(AMS)：管理四大组件
    - PackageManagerService(PMS)：管理APK安装和权限
    - PowerManagerService：电源管理

3. **启动核心服务**：
    - BatteryService：电池状态监控
    - WebViewUpdateService：WebView更新

4. **启动其他服务**：
    - WindowManagerService(WMS)：窗口管理
    - BluetoothService：蓝牙服务
    - CameraService：相机服务

5. **系统就绪**：所有服务启动后调用systemReady()通知各服务

### 7. Launcher启动阶段

系统服务的最后阶段是启动Launcher：

1. **启动SystemUI**：包括状态栏、导航栏等系统UI
2. **关闭开机动画**：WMS通知SurfaceFlinger关闭动画
3. **启动Launcher**：
    - PMS返回已安装应用信息
    - AMS启动HOME类型的Activity
    - 显示应用图标和桌面

4. **发送启动完成广播**：ACTION_BOOT_COMPLETED通知系统启动完成

### 8. 整体流程总结

Android系统启动的整体流程可以简化为以下步骤：

1. **Boot ROM** → **Bootloader** → **Linux Kernel** → **Init进程**
2. **Init** → **Zygote** → **SystemServer**
3. **SystemServer** → **核心服务** → **Launcher**

每个阶段都依赖于前一阶段的成功执行，共同构成了Android系统从硬件上电到用户界面显示的完整启动链条。理解这个流程对于Android系统开发、性能优化和问题排查都有重要意义。
