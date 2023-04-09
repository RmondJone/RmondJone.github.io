---
title: "VPS服务器一键BBR加速教程--网速变10倍"
date: 2023-04-07T10:28:08+08:00
draft: false
categories: ["其他"]
---
### 一、下载BBR多合一脚本并授予执行权限

```
#下载脚本
wget --no-check-certificate -O tcpx.sh https://raw.githubusercontent.com/ylx2016/Linux-NetSpeed/master/tcpx.sh

#授予权限
chmod +x tcpx.sh

#执行脚本
./tcpx.sh
```

### 二、选择安装合适的加速内核并启动加速

![](/images/bbr.webp)


**安装内核和启动加速都会需要重启VPS服务器！！**

### 三、Docker搭建速度测试进行测试对比
这里使用了Docker搭建speedtest,下载对于镜像启动相关服务即可
```
#下载docker镜像
docker pull adolfintel/speedtest
#启动容器
docker run --name=speedtest -p 9001:80 adolfintel/speedtest
```
启动完成即可使用VPS的ip+对应端口访问网速测试网站,点击开始按钮即可进行测速

![](/images/bbr_1.webp)


### 四、BBR加速对比
#### 1、不启动BBR加速，直接裸连

![](/images/bbr_2.webp)


#### 2、锐速

![](/images/bbr_3.webp)


**友情提示：Debian 9和10系统由于内核版本问题都不支持锐速**

#### 3、BBR原版

![](/images/bbr_4.webp)


