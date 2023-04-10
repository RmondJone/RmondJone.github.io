---
title: "Gradle结合蒲公英做一键上传打包至蒲公英"
date: 2023-04-07T10:11:30+08:00
tags: ["Devops"]
categories: ["DevOps"]
draft: false
---
# 一、背景
在日常工作中，我们会使用蒲公英上传我们的APK，用来测试。每次打包，然后再去找到包，再上传到蒲公英，流程很繁琐，于是我想着把这些操作写成gralde脚本来简化这些操作。

# 二、脚本的实现
利用蒲公英开发出的Api [https://www.pgyer.com/doc/view/api#uploadApp](https://www.pgyer.com/doc/view/api#uploadApp)，调用curl命令直接上传Apk包到蒲公英上

```java
import groovy.json.JsonBuilder
import groovy.json.JsonSlurper
import org.apache.tools.ant.taskdefs.condition.Os

//https://www.pgyer.com/doc/view/api#uploadApp
ext {
    api_key = "xxxxxxxxxxxxxxxxxx"
    user_key = "xxxxxxxxxxxxxxxxxx"
    // Apk输出路劲
    apk_path = "${project.rootDir}/apks"
}

/**
 * 注释：打包Debug包并上传蒲公英
 * 时间：2021/7/5 0005 15:06
 * 作者：郭翰林
 */
task pgyerBuildDebug() {
    group 'pgyer'
    dependsOn('assembleDebug')
    doLast {
        def apkFileTree = fileTree(dir: apk_path, includes: ["**/*.apk"])
        apkFileTree.each {
            File file ->
                def newFile = renameFileName(file)
                uploadApp(newFile, "[Debug]${newFile.name}")
        }
    }
}

/**
 * 注释：打包Release并上传蒲公英
 * 时间：2021/7/5 0005 15:07
 * 作者：郭翰林
 */
task pgyerBuildRelease() {
    group 'pgyer'
    dependsOn('assembleRelease')
    doLast {
        def apkFileTree = fileTree(dir: apk_path, includes: ["**/*.apk"])
        apkFileTree.each {
            File file ->
                def newFile = renameFileName(file)
                uploadApp(newFile, "[Release]${newFile.name}")
        }
    }
}

/**
 * 注释：获取Git分支
 * 时间：2021/9/8 0008 10:59
 * 作者：郭翰林
 * @return
 */
def loadGitBranch() {
    def config = file("${project.rootDir}/.git/HEAD")
    String content = ""
    config.eachLine { line ->
        content = line
    }
    return content.replaceAll("ref: refs/heads/", "")
}

/**
 * 注释：重命名APK，解决执行删除之后打包名称没有变化的Bug
 * 时间：2021/7/29 0029 23:19
 * 作者：郭翰林
 * @param file
 * @return
 */
def renameFileName(File file) {
    def names = file.name.split("-")
    def appName = names[0]
    def buildType = names[1]
    def versionNo = names[2]
    def buildTime = project.hasProperty('BUILD_TIME') ? BUILD_TIME : new Date().format("yyyyMMddHHmm", TimeZone.getTimeZone("GMT+08:00"))
    File newFile = new File(apk_path, "${appName}-${buildType}-${versionNo}-${buildTime}.apk")
    file.renameTo(newFile)
    return newFile
}

/**
 * 注释：删除apk输出文件夹
 * 时间：2021/7/28 0028 15:35
 * 作者：郭翰林
 */
task deleteApkPath() {
    doLast {
        delete apk_path
        println("*******************APK输出目录已删除***********************")
    }
}

/**
 * 注释：打包Task任务依赖注入deleteApkPath
 * 时间：2021/7/28 0028 15:38
 * 作者：郭翰林
 */
project.tasks.whenTaskAdded { Task currentTask ->
    if (currentTask.name == "assembleDebug" || currentTask.name == "assembleRelease") {
        currentTask.dependsOn('deleteApkPath')
        currentTask.mustRunAfter('deleteApkPath')
    }
}


/**
 * 注释：上传App至蒲公英
 * 时间：2021/7/5 0005 15:13
 * 作者：郭翰林
 */
def uploadApp(File file, String description) {
    def out = new ByteArrayOutputStream()
    exec {
        if (Os.isFamily(Os.FAMILY_WINDOWS)) {
            ExecSpec execSpec = commandLine "cmd", "/c", "curl -F _api_key=${api_key} -F userKey=${user_key} -F file=@${file.absolutePath} -F buildInstallType=2 -F buildPassword=xxxxxx -F buildUpdateDescription=${description} https://www.pgyer.com/apiv2/app/upload"
            execSpec.standardOutput = out
        } else {
            ExecSpec execSpec = commandLine "bash", "-c", "curl -F _api_key=${api_key} -F userKey=${user_key} -F file=@${file.absolutePath} -F buildInstallType=2 -F buildPassword=xxxxxx -F buildUpdateDescription=${description} https://www.pgyer.com/apiv2/app/upload"
            execSpec.standardOutput = out
        }
    }
    println("蒲公英上传结果：\n${out.toString()}")
    def uploadResult = new JsonSlurper().parseText(out.toString())
    if (uploadResult.code == 0) {
        sendMsgToDing(getFileSize(file), uploadResult.data)
    } else {
        println("蒲公英上传失败原因:${uploadResult.message}")
    }
}

/**
 * 注释：获取文件大小
 * 时间：2021/7/30 0030 8:56
 * 作者：郭翰林
 * @param file
 */
def getFileSize(File file) {
    long size = file.size()
    if (size < 1024) {
        return "${new BigDecimal(size).setScale(1, BigDecimal.ROUND_HALF_UP).doubleValue()}B"
    } else if (size < 1024 * 1024) {
        return "${new BigDecimal(size / 1024).setScale(1, BigDecimal.ROUND_HALF_UP).doubleValue()}KB"
    } else if (size < 1024 * 1024 * 1024) {
        return "${new BigDecimal(size / 1024 / 1024).setScale(1, BigDecimal.ROUND_HALF_UP).doubleValue()}MB"
    }
}

/**
 * 注释：发送钉钉通知
 * 时间：2021/7/29 0029 20:01
 * 作者：郭翰林
 * @param data
 * @return
 */
def sendMsgToDing(def fileSize, def data) {
    def conn = new URL("https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx").openConnection()
    conn.setRequestMethod('POST')
    conn.setRequestProperty("Connection", "Keep-Alive")
    conn.setRequestProperty("Content-type", "application/json;charset=UTF-8")
    conn.setConnectTimeout(30000)
    conn.setReadTimeout(30000)
    conn.setDoInput(true)
    conn.setDoOutput(true)
    def dos = new DataOutputStream(conn.getOutputStream())
    def qrCodeUrl = "![](" + data.buildQRCodeURL + ")"
    def _title = "蒲公英机器人"
    def _content = new StringBuffer()
    _content.append("\n\n### Android构建成功,已上传蒲公英")
    _content.append("\n\n**构建应用**:\n\nxxxApp")
    _content.append("\n\n**构建时间**:\n\n${data.buildUpdated}")
    _content.append("\n\n**构建描述**:\n\n${data.buildUpdateDescription}")
    _content.append("\n\n**蒲公英版本号**:\n\nbuild${data.buildBuildVersion}")
    _content.append("\n\n**Android包大小**:\n\n${fileSize}")
    _content.append("\n\n**Android版本号**:\n\n${data.buildVersion}")
    _content.append("\n\n**Git分支**:\n\n${loadGitBranch()}")
    _content.append("\n\n**下载地址**:\n\nhttps://www.pgyer.com/${data.buildKey}")
    _content.append("\n\n**安装密码**:\n\ncarzone123")
    _content.append("\n\n" + qrCodeUrl)
    def json = new JsonBuilder()
    json {
        msgtype "markdown"
        markdown {
            title _title
            text _content.toString()
        }
        at {
            atMobiles([])
            isAtAll false
        }
    }
    dos.writeBytes(json.toString())
    def input = new BufferedReader(new InputStreamReader(conn.getInputStream()))
    String line = ""
    String result = ""
    while ((line = input.readLine()) != null) {
        result += line
    }
    dos.flush()
    dos.close()
    input.close()
    conn.connect()
    println("钉钉发送通知结果:\n${result}")

    println("*************** 钉钉消息已发送 ***************")
}

```
![](/images/gradle_pgyer.webp)
