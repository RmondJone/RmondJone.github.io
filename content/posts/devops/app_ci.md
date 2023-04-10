---
title: "App自动化打包平台的搭建与维护"
date: 2023-04-07T10:25:32+08:00
tags: ["Devops"]
categories: ["DevOps"]
draft: false
---
### 一、前言
在日常开发中，测试阶段需要开发去手动打包，这样效率很低下，而且如果开发正在其他分支或者正在开发，打包也非常麻烦。所以在这个背景下，自动化打包平台的诞生显得尤为重要。

### 二、基础概念介绍

* Jenkins
  Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件项目可以进行持续集成

* Gradle
  Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建开源工具。它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，目前也增加了基于Kotlin语言的kotlin-based DSL，抛弃了基于XML的各种繁琐配置

* Fastlane
  Fastlane是用Ruby语言编写的一套自动化工具集和框架，每一个工具实际都对应一个Ruby脚本，用来执行某一个特定的任务，而fastlane核心框架则允许使用者通过类似配置文件的形式，将不同的工具有机而灵活的结合在一起，从而形成一个个完整的自动化流程。

### 三、App自动化打包的搭建流程

* 第一步，搭建Jenkins，这个网上都有大量教程，我就不详细赘述了。

* 第二步，搭建Android、IOS开发环境，这里网上也有大量教程，不在赘述

* 第三步，书写Jenkins自动化打包脚本，Jenkins新建对应的任务

总体来说，搭建App自动化打包的步骤主要就这3步。主要介绍的就是第3步，这里用我们工程里的脚本来详细说明，每行命令的作用。

```shell
pipeline {
    agent any
    parameters {
        string(name: 'EMAIL', defaultValue: 'liang@xyz.cn', description: '编译任务结束时通知的邮箱')
        string(name: 'PGYER_DESCRIPTION', defaultValue: '', description: '此次打包备注(可选,默认显示分支和此次任务Build ID)')

        choice(choices: ['ANDROID','IOS'], description: '编译哪个平台?', name: 'BUILD_PLATFORM')
        choice(choices: ['Debug', 'Preview', 'Release'], description: '编译什么版本?(Android P版请选择Preview不要用Release!!!)', name: 'BUILD_TYPE')

        string(name: 'FILE_NAME', defaultValue: 'xyz_app', description: '输出安装包的文件名(尽量英文),不能为空')

        string(name: 'API_HOST', defaultValue: '', description: '接口地址,不改就留空')

        string(name: 'BUILD_VERSION_NAME', defaultValue: '', description: '版本号(格式:x.y.z),不改就留空')

        string(name: 'PGYER_API_KEY', defaultValue: '43aad9a3e16b23382b6350e05546db49', description: '蒲公英分发api key,不懂请保持默认')
        string(name: 'PGYER_USER_KEY', defaultValue: '35f87a2f64bde6cfed825ae188e2c1f0', description: '蒲公英分发user key,不懂请保持默认')

        booleanParam(name:'UPDATE_POD',defaultValue:false,description:'是否更新Pod库?')
        choice(name:'POD_REPO',choices:["xyz-appdev-spec","artsy","master"],description:'具体更新哪一个Pod库')

        booleanParam(name:'GRADLE_OFFLINE',defaultValue:true,description:'是否采用Gradle离线模式?')
    }
    environment {
        LANG = "en_US.UTF-8"
        //sentry cli cdn 链接
        SENTRYCLI_CDNURL = 'http://cdn.npm.taobao.org/dist/sentry-cli'
		    //蒲公英 ipa路径
		    PGYER_IPA = "./build/XYZ_iOS.ipa"
		    //蒲公英 上传描述
		    PGYER_UPDATE_DESCRIPTION = "Jenkins任务号: 【${BUILD_DISPLAY_NAME}】 本次编译的分支:【${BRANCH_NAME}】  编译类型:【${params.BUILD_TYPE}】  额外备注:【${params.PGYER_DESCRIPTION}】"
        //ios 输出目录
        GYM_OUTPUT_DIRECTORY = "./build"
		    //testFlight ipa 路径
		    PILOT_IPA = "${PGYER_IPA}"
		    //testFlight 升级描述
		    PILOT_BETA_APP_DESCRIPTION = "${PGYER_UPDATE_DESCRIPTION}"
		    FL_VERSION_NUMBER_VERSION_NUMBER = "${params.BUILD_VERSION_NAME}"
    }
    tools {
        nodejs "nodejs"
    }
    stages {
        stage('Prepare') {
            steps {
                sh "mkdir build && printenv && printenv > build/env.txt"
                script {
                    if("${params.BUILD_TYPE}" !="Debug"){
                      sh '''
                          sed -i "s#sensorsAnalytics.disablePlugin=true#sensorsAnalytics.disablePlugin=false#g" android/gradle.properties
                      '''
                      sh "cat android/gradle.properties"
                    }
                    sh '''
                        sed -i "s#{...getPersistenceFunctions(pageName)}##g" app/entry/App.js
                    '''
                    sh "cat app/entry/App.js"
                    if("${params.BUILD_VERSION_NAME}" != ""){
                      sh '''
                         sed -i "s/\\(VersionName=\\)\\([0-9]\\{1,\\}\\.[0-9]\\{1,\\}\\.[0-9]\\{1,\\}\\)/\\1${BUILD_VERSION_NAME}/g" android/gradle.properties
                      '''

                      sh "cat android/gradle.properties"
                      sh "cd ios && fastlane build_version"
                    }
                    if("${params.API_HOST}" != ""){
                      sh '''
                         sed -i "s#https://app\\.xyz\\.cn#${API_HOST}#g" android/app/src/main/java/com/focustech/xyz/network/configuration/Host.java
                      '''
                      sh "cat android/app/src/main/java/com/focustech/xyz/network/configuration/Host.java"
                      sh '''
                         sed -i "s#https://app\\.xyz\\.cn#${API_HOST}#g" ios/XYZ_iOS/Resources/Plists/NetEnvironment.json
                      '''
                      sh "cat ios/XYZ_iOS/Resources/Plists/NetEnvironment.json"
                    }
                    if(params.UPDATE_POD && "${params.BUILD_PLATFORM}"=='IOS'){
                       sh "cd ios && pod repo update ${params.POD_REPO}"
                    }
				        }
            }
        }
        stage('Build') {
            when { expression { return !params.UPDATE_POD } }
            steps {
                script {
                    sh 'npm install'
                    if ("${params.BUILD_PLATFORM}"  == 'ANDROID') {
                        if(params.GRADLE_OFFLINE){
                            if ("${params.BUILD_TYPE}"  == "Preview"){
                              sh "cd android && bash gradlew --build-cache --offline assembleRelease"
                            }else if("${params.BUILD_TYPE}"  == "Release"){
                              sh "cd android && bash gradlew --build-cache --offline buildReinforceRelease"
                            }else {
                              sh "cd android && bash gradlew --build-cache --offline assemble${params.BUILD_TYPE}"
                            }
                        }else{
                            if ("${params.BUILD_TYPE}"  == "Preview"){
                              sh "cd android && bash gradlew --build-cache assembleRelease"
                            }else if("${params.BUILD_TYPE}"  == "Release"){
                              sh "cd android && bash gradlew --build-cache buildReinforceRelease"
                            }else {
                              sh "cd android && bash gradlew --build-cache assemble${params.BUILD_TYPE}"
                            }
                        }
                    } else if ("${params.BUILD_PLATFORM}"  == 'IOS'){
                        sh 'cd ios && pod install --verbose'
                        if("${params.BUILD_TYPE}"  == 'Debug'){
                          sh 'cd ios && fastlane build'
                        } else{
                          sh 'cd ios && fastlane build_ts'
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            when { expression { return !params.UPDATE_POD } }
            steps {
                script {
                    archiveArtifacts artifacts: 'build/env.txt'
                    if ("${params.BUILD_PLATFORM}"  == 'ANDROID') {
                        if("${params.BUILD_TYPE}" == 'Release'){
                            sh "mv android/app/build/outputs/apk/release/*.apk build/${FILE_NAME}-${BUILD_TYPE}.apk"
                            sh "zip -r android/app/build/outputs/apk/release/${FILE_NAME}.zip android/app/build/outputs/apk/release/channels/"
                            sh "mv android/app/build/outputs/apk/release/${FILE_NAME}.zip build/"
                            archiveArtifacts artifacts: "build/*.apk"
                            archiveArtifacts artifacts: "build/*.zip"
                        }else if("${params.BUILD_TYPE}" == 'Debug') {
                            sh "mv android/app/build/outputs/apk/debug/*.apk build/${FILE_NAME}-${BUILD_TYPE}.apk"
                            archiveArtifacts artifacts: "build/${FILE_NAME}-${BUILD_TYPE}.apk"
                        }else if("${params.BUILD_TYPE}" == 'Preview') {
                            sh "mv android/app/build/outputs/apk/release/*.apk build/${FILE_NAME}-${BUILD_TYPE}.apk"
                            archiveArtifacts artifacts: "build/${FILE_NAME}-${BUILD_TYPE}.apk"
                        }
                        sh '''
                            curl -F "file=@build/${FILE_NAME}-${BUILD_TYPE}.apk" -F "userKey=${PGYER_USER_KEY}" -F "_api_key=${PGYER_API_KEY}" -F "buildInstallType=2" -F "buildPassword=123456" -F "buildUpdateDescription=${PGYER_UPDATE_DESCRIPTION}" http://www.pgyer.com/apiv2/app/upload
                        '''
                    } else if ("${params.BUILD_PLATFORM}"  == 'IOS'){
                        archiveArtifacts artifacts: "ios/build/*.ipa"
                        archiveArtifacts artifacts: "ios/build/*.app.dSYM.zip"
                        sh "mv ios/build/*.ipa build/${FILE_NAME}-${BUILD_TYPE}.ipa"
                        sh '''
                           curl -F "file=@build/${FILE_NAME}-${BUILD_TYPE}.ipa" -F "userKey=${PGYER_USER_KEY}" -F "_api_key=${PGYER_API_KEY}" -F "buildInstallType=2" -F "buildPassword=123456" -F "buildUpdateDescription=${PGYER_UPDATE_DESCRIPTION}" http://www.pgyer.com/apiv2/app/upload
                       '''
                    }
                }
            }
        }
    }
    post {
        failure {
            emailext attachLog: true, attachmentsPattern: 'build/**', body: "jenkins任务失败!  ${PGYER_UPDATE_DESCRIPTION}  详情请点击:${BUILD_URL} 或直接从附件下载编译log自行查看!", compressLog: true, replyTo: '', subject: 'jenkins任务失败,请排查问题', to: '${EMAIL}'
        }
        aborted {
            emailext attachLog: true, attachmentsPattern: 'build/**', body: "jenkins任务中止!  ${PGYER_UPDATE_DESCRIPTION}  详情请点击:${BUILD_URL}", compressLog: true, replyTo: '', subject: 'jenkins任务已中止', to: '${EMAIL}'
        }
        success {
            script {
              if(!params.UPDATE_POD){
                if ("${params.BUILD_PLATFORM}"  == 'ANDROID') {
                  def packPath = "build/${FILE_NAME}-${BUILD_TYPE}.apk"
                  echo "packPath:${packPath}"
                  def packInfo = """${sh(returnStdout: true, script: "du -s ${packPath}")}"""
                  def packSize = packInfo.substring(0, packInfo.indexOf('	'))
                  echo "packSize:${packSize}"
                  def packSizeMb = packSize.toInteger() / 2 / 1000
                  def packSizeMbStr = "" + packSizeMb

                  if("${params.BUILD_TYPE}" == 'Preview'){
                    if(packSizeMb > 40){
                      emailext body: "任务${BUILD_DISPLAY_NAME}安卓Preview包大小已经超过40Mb，现已达到${packSizeMbStr}Mb！", compressLog: true, replyTo: '', subject: '任务${BUILD_DISPLAY_NAME}安卓Preview包大小已经超过40Mb！', to: 'zhangwenxin@xyz.cn;guohanlin@xyz.cn;jinwenwu@xyz.cn;jinjianxin@xyz.cn'
                    }
                  }
                }else if ("${params.BUILD_PLATFORM}"  == 'IOS'){
                  def packPath = "build/${FILE_NAME}-${BUILD_TYPE}.ipa"
                  echo "packPath:${packPath}"
                  def packInfo = """${sh(returnStdout: true, script: "du -s ${packPath}")}"""
                  def packSize = packInfo.substring(0, packInfo.indexOf('	'))
                  echo "packSize:${packSize}"
                  def packSizeMb = packSize.toInteger() / 2 / 1000
                  def packSizeMbStr = "" + packSizeMb

                  if("${params.BUILD_TYPE}" == 'Preview'){
                    if(packSizeMb > 40){
                      emailext body: "任务${BUILD_DISPLAY_NAME}苹果Preview包大小已经超过40Mb，现已达到${packSizeMbStr}Mb！", compressLog: true, replyTo: '', subject: '任务${BUILD_DISPLAY_NAME}苹果Preview包大小已经超过40Mb！', to: 'zhangwenxin@xyz.cn;guohanlin@xyz.cn;jinwenwu@xyz.cn;jinjianxin@xyz.cn'
                    }
                  }
                }
              }
            }
            script {
               if(!params.UPDATE_POD){
                  if("${params.BUILD_PLATFORM}"=="IOS"){
                    emailext body: "jenkins任务成功! ${PGYER_UPDATE_DESCRIPTION} 详情请点击:${BUILD_URL} 或直接通过蒲公英下载! iOS下载链接 http://www.pgyer.com/xyz_ios", compressLog: true, replyTo: '', subject: 'jenkins任务成功,请下载安装', to: '${EMAIL}'
                  } else{
                    emailext body: "jenkins任务成功! ${PGYER_UPDATE_DESCRIPTION} 详情请点击:${BUILD_URL} 或直接通过蒲公英下载! Android下载链接 http://www.pgyer.com/xyz_android", compressLog: true, replyTo: '', subject: 'jenkins任务成功,请下载安装', to: '${EMAIL}'
                  }
               }
            }
        }
    }
}
```

首先我们来大致说下这个文件的大概意思，网上也有教程不明白的，可以再上网多找找资料。

第一大块**parameters**，这里面主要定义了一些任务的参数，例如我要打哪个平台的包啊、版本号啊、邮件通知啊这些。

第二大块**environment**，这里面主要定义了一些任务执行的环境以及预定义的一些全局的环境变量

第三大块**stages**，这里是比较重要的一块内容，这里主要定义了一些任务的阶段性任务，以及这个阶段性任务执行哪些脚本。

就好像上面的脚本，我把整个大任务分为了3个子任务：
* 第一个子任务**stage('Prepare'）**，主要定义了一些我要打包前需要做的一些事情，例如执行npm、替换工程中部分代码等等。
* 第二个子任务**stage('Build')**，主要就是执行真正的打包流程，例如Android走了Gradle脚本去执行打包，IOS走了FastLane去执行打包。
* 第三个子任务**stage('Deploy')**，主要执行打包完之后的一些操作，例如打出的包的归档操作、包大小检测、上传蒲公英操作、上传蒲公英之后邮件通知操作

### 四、Android、IOS打包脚本详解

我们先来看Android，这里需要你有一定的Gradle基础，可以自己写简单的Gradle任务。从上面的Jenkins脚本，我们可以看到Android在打包的时候，区分了编译版本，debug和preview的直接调用Android自带的打包脚本assembleDebug、assembleRelease，Release版本也就是发布版本，这里我们做了自定义Gradle脚本**buildReinforceRelease**，为什么我们要自定义，因为通常我们上线发布的时候都需要加固APK并且做多渠道操作。如果每次都去手动用工具操作的话，这个是个很蛋疼的活。下面我们来看看这个脚本里有什么。

```groovy
ext {
    //加固插件路径
    reinforce_plugin_path = "${project.rootDir}/reinforce"
	//360加固账号
    reinforce_plugin_name = 'xxxxxxxx'
    reinforce_plugin_passward = 'xxxxxxx'
    //签名信息
    key_store_path = "${project.rootDir}/app/xyz.keystore"
    key_store_passward = 'xxxxxxxxx'
    alias = 'xxxx'
    alias_passward = 'xxxxxxxxxx'
    //Release Apk输出路劲
    release_apk_path = "${project.buildDir}/outputs/apk/release/app-release.apk"
    //渠道配置文件
    chanel_config_path = "${reinforce_plugin_path}/channel.txt"
    //渠道Apk输出路径
    channel_apks_path = "${project.buildDir}/outputs/apk/release/channels/"
}

/**
 * 注释：编译加固渠道包
 * 时间：2019/1/2 0002 14:01
 * 作者：郭翰林
 */
task buildReinforceRelease() {
    group '360reinforce'
    dependsOn('assembleRelease')
    doLast {
        //第一步：清空缓存
        cleanFilesPath(reinforce_plugin_path + "/.cache")
        cleanFilesPath(reinforce_plugin_path + "/output")
        cleanFilesPath(reinforce_plugin_path + "/jiagu.db")
        cleanFilesPath(reinforce_plugin_path + "/tempAndroidManifest.xml")
        cleanFilesPath(channel_apks_path)
        //第二步：开始加固
        reinforceApk()
        //清除无用缓存
        cleanFilesPath(reinforce_plugin_path + "/.cache")
        cleanFilesPath(reinforce_plugin_path + "/output")
        cleanFilesPath(reinforce_plugin_path + "/jiagu.db")
        cleanFilesPath(reinforce_plugin_path + "/tempAndroidManifest.xml")
    }
}

/**
 * 注释：使用360加固加固Release包
 * 时间：2019/1/2 0002 14:32
 * 作者：郭翰林
 */
def reinforceApk() {
    println('开始进行加固操作')
    File releaseApk = new File(release_apk_path)
    if (!releaseApk.exists()) {
        throw new FileNotFoundException("Release包不存在，无法进行加固操作,文件路径：${releaseApk.getAbsolutePath()}")
    }
    //创建渠道文件夹
    File channelApksPath = new File(channel_apks_path)
    if (!channelApksPath.exists()) {
        channelApksPath.mkdir()
    }
    exec {
        commandLine "bash", "-c", "chmod +x ${reinforce_plugin_path}/java/bin/*"
    }
    exec {
        commandLine "bash", "-c", "java -jar ${reinforce_plugin_path}/jiagu.jar -login ${reinforce_plugin_name} ${reinforce_plugin_passward}"
    }
    exec {
        commandLine "bash", "-c", "java -jar ${reinforce_plugin_path}/jiagu.jar -importsign ${key_store_path} ${key_store_passward} ${alias} ${alias_passward}"
    }
    exec {
        commandLine "bash", "-c", "java -jar ${reinforce_plugin_path}/jiagu.jar -importmulpkg ${chanel_config_path}"
    }
    exec {
        commandLine "bash", "-c", "java -jar ${reinforce_plugin_path}/jiagu.jar -jiagu ${release_apk_path} ${channel_apks_path} -autosign -automulpkg"
    }
    println('加固操作结束，加固包路径' + channelApksPath.getAbsolutePath())
}

/**
 * 注释：清空文件夹
 * 时间：2019/1/2 0002 14:15
 * 作者：郭翰林
 */
def cleanFilesPath(String path) {
    File files = new File(path)
    if (!files.exists()) {
        return
    }
    println('开始执行清除:' + files.getAbsolutePath())
    if (files.isDirectory()) {
        String[] content = files.list()//取得当前目录下所有文件和文件夹
        for (String name : content) {
            File temp = new File(path, name)
            if (temp.isDirectory()) {//判断是否是目录
                cleanFilesPath(temp.getAbsolutePath())//递归调用，删除目录里的内容
                temp.delete()
            } else {
                temp.delete()
            }
        }
    }
    files.delete()
}
```
可以看到这里我们使用了360加固来执行加固和打渠道包的操作，那么我们怎么去把360加固引到工程里呢？答案很简单，直接360官网下载，然后直接拉360安装路径的文件夹到工程里。如下图所示：

![](/images/360_jiagu.webp)


Android的部分就介绍到这，下面我们来看IOS打包脚本部分，前面说到IOS是通过FastLane去实现打包的，但众所周知，IOS打包需要证书以及签名，这个我们在FastLane脚本里改怎么去配置？

关于Fastlane怎么集成，怎么部署网上都有教程这里不去赘述，这里主要说的就是Fastlane里的脚本该如何去书写，怎么去配置证书以及签名

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
  desc "build projcet"
  #配置Fastlane任务
  lane :build do
    #配置工程基本信息
    update_app_identifier(
      xcodeproj: "XYZ_iOS.xcodeproj", # Optional path to xcodeproj, will use the first .xcodeproj if not set
      plist_path: "XYZ_iOS/XYZ_iOS-Info.plist", # Path to info plist file, relative to xcodeproj
      app_identifier: "com.focuschina.xyz" # The App Identifier
    )
	#配置签名以及证书
    automatic_code_signing(
      path: "XYZ_iOS.xcodeproj",
	  #启用自动签名
      use_automatic_signing: true,
      code_sign_identity: "iPhone Developer",
	  #根据证书拿到团队ID或者直接去苹果开发者官方查阅
      team_id:"HB2YRTXXXXX",
    )
	#配置生成包的类型以及发布渠道
    gym(scheme: "XYZ_iOS",
      workspace: "XYZ_iOS.xcworkspace",
      configuration: "Debug",
      clean: true,
      output_name:"XYZ_iOS.ipa",
      export_xcargs:"-allowProvisioningUpdates",
      export_method: "development"           # app-store, ad-hoc, package, enterprise, development, developer-id
    )
  end
  lane :build_ts do
    update_app_identifier(
      xcodeproj: "XYZ_iOS.xcodeproj", # Optional path to xcodeproj, will use the first .xcodeproj if not set
      plist_path: "XYZ_iOS/XYZ_iOS-Info.plist", # Path to info plist file, relative to xcodeproj
      app_identifier: "com.focuschina.xyz" # The App Identifier
    )
    automatic_code_signing(
      path: "XYZ_iOS.xcodeproj",
      use_automatic_signing: true,
      code_sign_identity: "iPhone Developer",
      team_id:"HB2YRXXXXXX",
    )
    gym(scheme: "XYZ_iOS",
      workspace: "XYZ_iOS.xcworkspace",
      configuration: "Release",
      clean: true,
      output_name:"XYZ_iOS.ipa",
      export_xcargs:"-allowProvisioningUpdates",
      export_method: "app-store"           # app-store, ad-hoc, package, enterprise, development, developer-id
    )
  end
  lane :build_number do
    increment_build_number(build_number: ENV["BUILD_VERSION_CODE"])
  end
  lane :build_version do
    increment_version_number
  end
  lane :upload do
    pgyer
  end
  lane :upload_ts do
    upload_to_testflight
  end
end

```

从上面可以看到定义了很多**lane: XXX do**,这个就是定义Fastlane的任务的原子操作，翻阅上面Jenkins脚本，最后调用的打包脚本也就是这里的任务。

### 五、Jenkins里的任务配置

前面几个章节已经说了怎么去书写Jenkinsfile脚本以及Android、IOS的打包脚本该怎么去写。下面我们来详细说说，怎么把这些部署到Jenkins，变成可视化的配参任务。

**第一步**，首先新建一个Item任务，然后选择多分支流水

![](/images/ci_1.webp)


**第二步**，配置任务的分支源，以及编译配置

![](/images/ci_2.webp)

![](/images/ci_3.webp)

对，这里我们直接填写Jenkinsfile的名称即可，因为我的Jenkins脚本是直接放在工程根目录下的，如果你不是放在根目录，则需要填写正确的层级。

**第三步** 点击保存，则可以在Jenkins主面板看到新建的多分支流水任务，第一次运行可能看不到可视化参数配置，需要第二次运行分支脚本才可以看到可视化参数配置。

![](/images/ci_4.webp)

![](/images/ci_5.webp)




