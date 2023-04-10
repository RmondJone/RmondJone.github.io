---
title: "Yapi Pro的搭建"
date: 2023-04-07T09:53:15+08:00
draft: false
categories: ["其他"]
---
### 一、Yapi是什么？
YApi 是高效、易用、功能强大的 api 管理平台，旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API，YApi 还为用户提供了优秀的交互体验，开发人员只需利用平台提供的接口数据写入工具以及简单的点击操作就可以实现接口的管理。

### 二、Yapi的快速搭建

#### 1、Docker创建mongoDB的虚拟磁盘
```shell
docker volume create mongo-data
```

#### 2、拉取mongoDB镜像并且创建容器

```shell
docker pull mongo:latest
```
```shell
docker run -d \
  --name mongodb \
  --restart always \
  --net=yapi \
  -p 27017:27017 \
  -v mongo-data:/data/db \
  -e MONGO_INITDB_DATABASE=yapi \
  -e MONGO_INITDB_ROOT_USERNAME=yapipro \
  -e MONGO_INITDB_ROOT_PASSWORD=yapipro1024 \
  mongo
```

#### 3、进入MongoDB容器，初始化表
```shell
docker exec -it mongodb mongosh
```


```shell
use admin;
db.auth("yapipro", "yapipro1024");
# 创建 yapi 数据库
use yapi;
# 创建给 yapi 使用的账号和密码，限制权限
db.createUser({
  user: 'yapi',
  pwd: 'yapi123456',
  roles: [
 { role: "dbAdmin", db: "yapi" },
 { role: "readWrite", db: "yapi" }
  ]
});
# 退出 Mongo Cli
exit
# 退出容器
exit

```

#### 4、宿主机创建Yapi配置文件

```json
 {
   "port": "3000",
   "adminAccount": "xxxxxx@gmail.com",
   "timeout":120000,
   "db": {
     "servername": "mongo",
     "DATABASE": "yapi",
     "port": 27017,
     "user": "yapi",
     "pass": "yapi123456",
     "authSource": ""
   },
   "mail": {
     "enable": true,
     "host": "smtp.gmail.com",
     "port": 465,
     "from": "*",
     "auth": {
       "user": "xxxxxx@gmail.com",
       "pass": "xxx"
     }
   }
 }
```

#### 5、拉取Yapi镜像并创建容器

```shell
docker pull yapipro/yapi:latest
# 初始化数据库表
docker run -d --rm \
  --name yapi-init \
  --link mongodb:mongo \
  --net=yapi \
  -v $PWD/config.json:/yapi/config.json \
   yapipro/yapi \
  server/install.js
# 初始化管理员账号在上面的 config.json 配置中 hexiaohei1024@gmail.com，初始密码是 yapi.pro，可以登录后进入个人中心修改
docker run -d \
   --name yapi \
   --link mongodb:mongo \
   --restart always \
   --net=yapi \
   -p 3000:3000 \
   -v $PWD/config.json:/yapi/config.json \
   yapipro/yapi \
   server/app.js
# 在服务器上验证 yapi 启动是否成功
curl localhost:3000
```

### 三、Yapi的访问地址
直接访问宿主机的3000端口即可，例如 http://localhost:3000

![](/images/yapi_pro.webp)

