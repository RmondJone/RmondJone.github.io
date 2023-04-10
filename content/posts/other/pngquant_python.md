---
title: "Python结合Pngquant给工程做图片批量压缩"
date: 2023-04-07T10:23:05+08:00
tags: ["Python"]
draft: false
categories: ["其他"]
---
# 前言
目前Android工程 APK包体积逐渐增大，从压缩图片来说是一个解决方案，但是目前网上都没有什么好用的傻瓜式的批量压缩方案，无意中发现Pngquant可以去做这一件事，但是也只能单个文件夹压缩，无法遍历整个工程文件进行图片压缩处理。

在这个背景下，我觉得开发一个Python脚本结合Pngquant去做这件事情还是有必要的

# 环境搭建
* Pngquant：直接去[官网](https://pngquant.org/)下载，然后加入到环境变量，在命令行可以运行pngquant -h没问题即可
* 下载[可执行文件](https://github.com/RmondJone/PicPngquant/releases/tag/1.0.0)，解压找到对应平台的可执行文件双击运行即可。


# Git源码
[https://github.com/RmondJone/PicPngquant](https://github.com/RmondJone/PicPngquant)

# 脚本介绍
```shell
import os
import platform
import threading


# 压缩线程（同步压缩）
class CompressThread(threading.Thread):
    # 构造方法
    def __init__(self, rootPath, compressFile, compressPath, extensionName) -> None:
        threading.Thread.__init__(self)
        self.root = rootPath
        self.compressFile = compressFile
        self.path = compressPath
        self.extension = extensionName

    # 运行方法
    def run(self) -> None:
        print("\n线程开始运行，压缩图片路径为：" + self.path)
        # 获得锁
        threadLock.acquire()
        cmd = "pngquant 256 --quality=65-80 --skip-if-larger --force --ext .png " + self.path
        os.system(cmd)
        # 重命名后缀
        if self.extension == 'jpg' or self.extension == 'jpeg':
            os.remove(self.path)
            os.rename(os.path.join(self.root, self.compressFile + ".png"),
                      os.path.join(self.root, self.compressFile))
        # 释放锁
        threadLock.release()
        print("\n线程结束运行，压缩图片路径为：" + self.path)


if __name__ == '__main__':
    tag = """
 _____ _             _____                                     _   
|  __ (_)           |  __ \                                   | |  
| |__) |  ___ ______| |__) | __   __ _  __ _ _   _  __ _ _ __ | |_ 
|  ___/ |/ __|______|  ___/ '_ \ / _` |/ _` | | | |/ _` | '_ \| __|
| |   | | (__       | |   | | | | (_| | (_| | |_| | (_| | | | | |_ 
|_|   |_|\___|      |_|   |_| |_|\__, |\__, |\__,_|\__,_|_| |_|\__|
                                  __/ |   | |                      
                                 |___/    |_|     
"""
print(tag)
excludeDir = []
isNeedExclude = input("是否需要配置排除压缩文件夹(Y/N)：")
if isNeedExclude == "Y" or isNeedExclude == "y":
    excludeDirStr = input("请输入需要排除压缩的文件夹(多个以空格分隔)：")
    excludeDir = excludeDirStr.split(" ")
    print("当前配置的排除压缩文件夹为：")
    print(excludeDir)


# 创建压缩线程
def addThread(rootPath, compressFile, compressPath, extensionName):
    compressThread = CompressThread(rootPath, compressFile, compressPath, extensionName)
    compressThread.start()
    threads.append(compressThread)


if os.system("pngquant --version") != 0:
    print("\n未检测到pngquant命令行环境，请参照pngquant官网搭建命令行环境：https://pngquant.org/")
else:
    dirPath = input("请选择需要压缩的文件夹路径：")
    # 去除输入路径首位空格
    dirPath = dirPath.rstrip()
    dirPath = dirPath.lstrip()
    print(dirPath)
    # 初始化线程锁
    threadLock = threading.Lock()
    # 压缩线程数组
    threads = []
    # 开始历遍所有图片
    for root, dirs, files in os.walk(dirPath):
        # 当前路径下所有的图片加入压缩线程
        for childFile in files:
            # 文件名
            childFilePath = os.path.join(root, childFile)
            father_path = os.path.abspath(os.path.dirname(childFilePath) + os.path.sep + ".")
            father_name = os.path.basename(father_path)
            if father_name not in excludeDir:
                # 扩展名
                extension = os.path.splitext(childFilePath)[1][1:]
                if platform.system() != 'Windows':
                    if extension == 'png' or extension == 'jpg' or extension == 'jpeg':
                        addThread(root, childFile, childFilePath, extension)
                else:
                    # Windows版pngquant只支持png压缩
                    if extension == 'png':
                        addThread(root, childFile, childFilePath, extension)
    # 开始遍历执行压缩线程
    for thread in threads:
        thread.join()

```

核心代码，主要就是使用python去遍历配置文件中定义的要压缩的文件夹，然后创建同步线程执行Pngquant压缩处理。

* --quality=65-80：压缩图片质量为65-80
* --skip-if-larger：舍弃意义不大的压缩
* --ext .png：这个是因为默认它会将解压缩后的Png文件重命名加后缀，这个参数即将重命名后加了一个空的字符的后缀，即等于不重命名了
* --force：不重命名后等于要覆盖原来的文件了，这里即强制覆盖原来的文件
