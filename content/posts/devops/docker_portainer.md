---
title: "Docker及图形化管理UI Portainer的搭建"
date: 2023-04-11T14:25:10+08:00
draft: false
categories: ["DevOps"]
tags: ["DevOps","Docker"]
---

## 一、什么是Docker?
Docker 是一个[开源](https://baike.baidu.com/item/%E5%BC%80%E6%BA%90/246339)的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 [Linux](https://baike.baidu.com/item/Linux)或Windows 机器上，也可以实现[虚拟化](https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E5%8C%96/547949)。容器是完全使用[沙箱](https://baike.baidu.com/item/%E6%B2%99%E7%AE%B1/393318)机制，相互之间不会有任何接口。

## 二、搭建Dokcer的方法

参考链接:[https://www.runoob.com/docker/ubuntu-docker-install.html](https://www.runoob.com/docker/ubuntu-docker-install.html)
这里唯一需要注意的就是Docker国内镜像的配置，以MAC为例：

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，我使用的是网易的镜像地址：http://hub-mirror.c.163.com。

在任务栏点击 Docker for mac 应用图标 -> Perferences... -> Daemon -> Registry mirrors。在列表中填写加速器地址即可。修改完成之后，点击 Apply & Restart 按钮，Docker 就会重启并应用配置的镜像地址了。

![](/images/docker_portainer_1.webp)

之后我们可以通过 docker info 来查看是否配置成功。
```java
$ docker info
...
Registry Mirrors:
 http://hub-mirror.c.163.com
Live Restore Enabled: false
```
## 三、什么是Portainer？
Portainer是Docker的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。
## 四、Portainer的搭建
非常简单一条命令即可

```java
docker pull portainer/portainer && docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```

上述命令执行了这几个步骤：
* 拉取 portainer/portainer镜像
* 以 portainer/portainer镜像构造容器，映射容器端口9000到本机9000端口，挂载容器磁盘/var/run/docker.sock到本机/var/run/docker.sock
  执行完命令之后，浏览器访问http://localhost:9000即可打开Portainer
  ![image.png](/images/docker_portainer_2.webp)
## 五、使用Docker搭建nginx,实现域名映射
* 在App Templates里找到Nginx模板，直接运行构建 nginx，或者使用如下命令搭建
```java 
docker pull nginx && docker run -d -p 80:80 -v /Users/jinwenwu/Documents/nginx/nginx.conf:/etc/nginx/nginx.conf  nginx
```
* 修改之前挂载的nginx.conf配置文件
```java
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen 80;
        server_name docker.xyz.cn;
        location / {
            proxy_pass http://192.168.27.180:9000;
        }
    }
}

```
* 在host文件中配置
```java
192.168.27.180 docker.xyz.cn
```
* 在Portainer重启nginx容器，即可通过docker.xyz.cn域名访问Portainer




