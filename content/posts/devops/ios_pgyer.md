---
title: "IOS 一键打包上传蒲公英脚本"
date: 2023-04-07T10:07:54+08:00
tags: ["Devops","Python"]
categories: ["DevOps"]
draft: false
---
# 一、背景
目前的IOS打包不支持蒲公英上传，并且原有的打包脚本是基于Python直接调用xcodebuild来完成打包操作,有以下几个缺点：

● 脚本书写繁杂难以理解，不方便日后维护，并且脚本中强依赖项目路径，可移植性比较差
● 脚本强依赖Python执行环境
● 只有单纯的打包功能，没有上传蒲公英、钉钉通知等功能
● 打完包之后无法区分是哪一个分支、哪一个时间段打的包

在以上的背景之下，我基于FastLane+Python重新开发了一套打包机制，并有较强的可移植性。

# 二、实现效果
只需工程根目录执行脚本PgyerBuild Debug即可完成**打包+上传蒲公英+钉钉通知**

![](/images/ios_pgyer.webp)


# 三、脚本集成步骤
● 机器安装FastLane，我使用的**版本是2.189.1**
● 工程内集成FastLane，修改FastLane配置文件，**第四章会给出配置模板**
● 拖入Python工程生成的脚本PgyerBuild到**工程根目录**(Python脚本内只需修改工程名即可)
● 集成完毕！执行打包脚本，即可完成**打包+上传蒲公英+钉钉通知**

**脚本使用教程**

```shell
#打Debug包
PgyerBuild Debug
#打Release包
PgyerBuild Release
```

# 四、脚本源码

**Python脚本**

```groovy
# This is a sample Python script.

import json
# Press ⇧F10 to execute it or replace it with your code.
# Press Double ⇧ to search everywhere for classes, files, tool windows, actions, and settings.
import os
import re
import sys
import time

import requests

# 时间格式化字符串
time_format = '%Y%m%d%H%M%S'
# 当前路径
basedir = os.path.dirname(os.path.realpath(sys.argv[0]))
# 蒲公英Apikey
api_key = "cd17b24dxxxxxxxxxxxxxxxxxxxxxxx"
# 蒲公英UserKey
user_key = "7058cxxxxxxxxxxxxxxxxxxxxxxxx"
# 工程名称
projectName = "TooWellMerchant"


# 获取系统当前时间并转换请求数据所需要的格式
def getTime(format_str):
    now = int(time.time())
    timeStruct = time.localtime(now)
    strTime = time.strftime(format_str, timeStruct)
    return strTime


# 执行IOS打包
def fastlane(buildType):
    if buildType == "Debug":
        os.system("fastlane buildDebug")
    else:
        os.system("fastlane buildRelease")


# 获取Git分支号
def getGitBranch():
    with open(f'{basedir}/.git/HEAD') as f:
        return f.read().replace("ref: refs/heads/", "")


# 获取版本号
def getVersionNo():
    with open(f'{basedir}/{projectName}.xcodeproj/project.pbxproj') as f:
        content = f.read()
        return re.findall(r"MARKETING_VERSION = (.+?);", content)[0]


# 上传蒲公英
def uploadPyger(buildType):
    if buildType == 'Debug':
        description = f'[Debug]business-debug-v{getVersionNo()}-{getTime(time_format)}.ipa'
    else:
        description = f'[Release]business-release-v{getVersionNo()}-{getTime(time_format)}.ipa'
    cmd = f'curl -F _api_key={api_key} -F userKey={user_key} -F file=@{basedir}/{projectName}.ipa -F ' \
          f'buildInstallType=2 -F buildPassword=carzone123 -F buildUpdateDescri' \
          f'ption={description} https://www.pgyer.com/apiv2/app/upload '
    print(f'正在执行上传脚本:{cmd}')
    result = os.popen(cmd)
    resultStr = result.read()
    print(f'蒲公英上传结果:{resultStr}')
    return json.loads(resultStr)


# 获取文件大小
def getFileSize(size):
    def strofsize(integer, remainder, level):
        if integer >= 1024:
            remainder = integer % 1024
            integer //= 1024
            level += 1
            return strofsize(integer, remainder, level)
        else:
            return integer, remainder, level

    units = ['B', 'KB', 'MB', 'GB', 'TB', 'PB']
    integer, remainder, level = strofsize(size, 0, 0)
    if level + 1 > len(units):
        level = -1
    return ('{}.{:>03d} {}'.format(integer, remainder, units[level]))


# 发送钉钉通知
def sendDingMessage(message):
    # 构建通知内容
    fileSize = getFileSize(int(message['buildFileSize']))
    buildUpdated = message['buildUpdated']
    buildUpdateDescription = message['buildUpdateDescription']
    buildBuildVersion = message['buildBuildVersion']
    buildVersion = message['buildVersion']
    buildKey = message['buildKey']
    buildQRCodeURL = message['buildQRCodeURL']
    text = []
    text.append('\n\n### IOS构建成功,已上传蒲公英')
    text.append('\n\n**构建应用**:\n\nXXXX')
    text.append(f'\n\n**构建时间**:\n\n{buildUpdated}')
    text.append(f'\n\n**构建描述**:\n\n{buildUpdateDescription}')
    text.append(f'\n\n**蒲公英版本号**:\n\nbuild{buildBuildVersion}')
    text.append(f'\n\n**IOS包大小**:\n\n{fileSize}')
    text.append(f'\n\n**IOS版本号**:\n\n{buildVersion}')
    text.append(f'\n\n**Git分支**:\n\n{getGitBranch()}')
    text.append(f'\n\n**下载地址**:\n\nhttps://www.pgyer.com/{buildKey}')
    text.append(f'\n\n**安装密码**:\n\ncarzone123')
    text.append(f'\n\n![]({buildQRCodeURL})')
    context = ''.join(text)

    # 发送钉钉通知
    webhook = 'https://oapi.dingtalk.com/robot/send?access_token' \
              '=1c7d9f9d880ca4ca9fxxxxxxxxxxxxxxxxxxxxxxxxxxxx '
    headers = {'Content-Type': 'application/json'}
    data = {
        'msgtype': 'markdown',
        'markdown': {
            "title": '蒲公英机器人',
            'text': context
        },
        'at': {
            'atMobiles': [],
            'isAtAll': False
        }
    }
    x = requests.post(url=webhook, data=json.dumps(data), headers=headers)
    if x.json()["errcode"] == 0:
        print("\n*************** 钉钉消息已发送 ***************")


# Press the green button in the gutter to run the script.
if __name__ == '__main__':
    # 构建应用
    fastlane(sys.argv[1])
    # 上传蒲公英
    uploadResult = uploadPyger(sys.argv[1])
    # 发送钉钉通知
    sendDingMessage(uploadResult['data'])
# See PyCharm help at https://www.jetbrains.com/help/pycharm/

```

* 使用**pyinstaller** 打出脚本，重命名PgyerBuild

**FastLane配置模板**

```shell
# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

default_platform(:ios)

platform :ios do
  desc "build project"
  lane :buildDebug do
    #配置工程基本信息
    update_app_identifier(
      xcodeproj: "TooWellMerchant.xcodeproj", # Optional path to xcodeproj, will use the first .xcodeproj if not set
      plist_path: "TooWellMerchant/Info.plist", # Path to info plist file, relative to xcodeproj
      app_identifier: "com.xxxxxx.xxxxxx" # The App Identifier
    )
    #配置签名以及证书
    automatic_code_signing(
      path: "TooWellMerchant.xcodeproj",
      #启用自动签名
      use_automatic_signing: true,
      code_sign_identity: "Apple Development",
      #根据证书拿到团队ID或者直接去苹果开发者官方查阅
      team_id:"K4T4XXXXXX",
    )
    #配置生成包的类型以及发布渠道
    gym(scheme: "TooWellMerchant",
      workspace: "TooWellMerchant.xcworkspace",
      configuration: "Debug",
      clean: true,
      output_name:"TooWellMerchant.ipa",
      #只打Arm64，缩小包体积
      xcargs:"ARCHS='arm64'",
      #IOS 9适配
      export_xcargs:"-allowProvisioningUpdates",
      export_method: "development"           # app-store, ad-hoc, package, enterprise, development, developer-id
    )
  end
  lane :buildRelease do
    update_app_identifier(
      xcodeproj: "TooWellMerchant.xcodeproj", # Optional path to xcodeproj, will use the first .xcodeproj if not set
      plist_path: "TooWellMerchant/Info.plist", # Path to info plist file, relative to xcodeproj
      app_identifier: "com.xxxxxx.xxxxxx" # The App Identifier
    )
    automatic_code_signing(
      path: "TooWellMerchant.xcodeproj",
      use_automatic_signing: true,
      code_sign_identity: "Apple Development",
      team_id:"K4T4XXXXXX",
    )
    gym(scheme: "TooWellMerchant",
      workspace: "TooWellMerchant.xcworkspace",
      configuration: "Release",
      clean: true,
      output_name:"TooWellMerchant.ipa",
      #只打Arm64，缩小包体积
      xcargs:"ARCHS='arm64'",
      #IOS 9适配
      export_xcargs:"-allowProvisioningUpdates",
      export_method: "development"           # app-store, ad-hoc, package, enterprise, development, developer-id
    )
  end
end
```
