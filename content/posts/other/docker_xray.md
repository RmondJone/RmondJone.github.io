---
title: "Docker搭建XRay用户管理以及流量伪装教程"
date: 2023-04-07T10:27:17+08:00
tags: ["Docker"]
draft: false
categories: ["其他"]
---
### 前言
XRay是近几年兴起的科学上网技术，采用新的协议，因功能强大、能有效抵抗墙的干扰而广受好评。

### 一、ACME配置SSL证书
* 安装socat和acme服务
```
# 安装socat
apt install socat

#安装acme
curl https://get.acme.sh | sh
```

* acme 绑定域名邮箱
```
~/.acme.sh/acme.sh --register-account -m my@example.com(随便啥邮箱)
```

* 阿里云创建AccessKey ID及AccessKey Secret，并配置环境变量
```
export Ali_Key="Key_XXX"
export Ali_Secret="Secret_XXX"
```
* 申请证书
```
acme.sh --issue --dns dns_ali -d *.abc.com \
--cert-file /root/ssl/xxxxx.crt \
--key-file  /root/ssl/xxxxx.key \
--fullchain-file /root/ssl/fullchain.crt
```


* CloudFlare创建AccessKey ID及AccessKey Secret，并配置环境变量
```
export CF_Key="Key_XXX"
export CF_Email="Secret_XXX"
```
* 申请证书
```
acme.sh --issue --dns dns_cf -d *.abc.com \
--cert-file /root/ssl/xxxxx.crt \
--key-file  /root/ssl/xxxxx.key \
--fullchain-file /root/ssl/fullchain.crt
```

* 定时任务保证了证书在到期前能自动续期,由于ACME协议和Let’s Encrypt CA都在频繁的更新，因此建议开启acme.sh的自动升级：
```
# 查看定时任务
crontab -l
```
```
~/.acme.sh/acme.sh  --upgrade  --auto-upgrade
```

* 移动SSL到指定目录,例如我这里移动到 /root/ssl文件夹下，没有就去创建一个，后面面板会用到
```
~/.acme.sh/acme.sh --install-cert -d 域名 \
--cert-file /root/ssl/xxxxx.crt \
--key-file  /root/ssl/xxxxx.key \
--fullchain-file /root/ssl/fullchain.crt
```

### 二、X-UI的Docker安装以及用户管理

安装XRay
```
docker run --restart=always --name x-ui -d -v /root/x-ui/db:/etc/x-ui -v /root/ssl:/root/cert  --network host enwaiax/x-ui:latest
```

54321端口是X-UI的管理端口安装完之后，通过你的服务器ip http://ip:54321 端口访问X-UI管理后台。

![](/images/xray.webp)

### 三、X-UI域名访问
搭建nginx
```
docker pull nginx && docker run -d -p 80:80 --network host --name nginx --restart=always nginx
```

进入nginx容器
```
#查看当前nginx的容器id
docker ps -a

#进入nginx容器
docker exec -it xxxx bash
```

更改nginx配置文件
```
#首先更新 apt-get
apt-get update

#安装vim
apt-get install vim

#编辑nginx配置文件
vim /etc/nginx/nginx.conf
```

nginx 通用配置模板
```
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

    server{
        listen 443 ssl;
        server_name  abc.com;
        ssl_certificate /root/ssl/fullchain.crt;
        ssl_certificate_key /root/ssl/xxxxx.key;
        ssl_trusted_certificate /root/ssl/fullchain.crt;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
        
        location / {
            proxy_pass https://abc.com:54321;
        }
    }
    
    server {
        listen	80;
        server_name  abc.com;
        # 核心代码
        rewrite ^(.*)$ https://${server_name}$1 permanent;
    }
}

```
配置完成之后，进入X-UI面板配置证书和端口，重启面板即可
![](/images/xray_1.jpg)


### 四、自建协议转换Clash链接服务

* 安装subconverter转换服务
```
#订阅转换后端
docker run  -d --name=subconverter --restart=always -p 9003:25500 stilleshan/subconverter
#订阅转换前端
docker run -d --name subweb --restart always \
  -p 9005:80 \
  -e API_URL='https://config.xxxx.xxxxx' \  #首先要把订阅转换后端使用nginx反代
  stilleshan/subweb
```

* 配置nginx反向代理

```
server {
    listen 80;
    server_name config.xxxxx.xxxxx;
    location / {
        proxy_pass http://localhost:9003;
    }
}

server {
    listen 80;
    server_name subweb.xxxxx.xxxxx;
    location / {
        proxy_pass http://localhost:9005;
    }
}
```

* 直接通过浏览器访问subweb.xxxxx.xxxxx，点击转换即可获得以下格式生成Clash订阅链接

```
http://config.xxxxx.xxxxx/sub?target=clash&url=%URL%
```
配置参考：[https://github.com/tindy2013/subconverter/blob/master/README-cn.md](https://github.com/tindy2013/subconverter/blob/master/README-cn.md)
