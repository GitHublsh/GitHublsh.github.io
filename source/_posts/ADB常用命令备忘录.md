---
title: ADB常用命令备忘录
date: 2017-09-02 23:27:48
tags: [ADB]
---

* adb start-server	启动服务

* adb kill-server	关闭服务
* adb devices	显示当前连接的所有设备（如果服务没有开启会自动开启）
* adb install xxx.apk	将应用安装进设备中
* adb uninstall <包名>	卸载应用
* adb -s <设备名> <命令>	如果有多个设备，指定某一个设备进行操作
* adb pull <手机文件> <电脑文件>	将手机内文件导入到电脑上(文件名均为全称)
* adb push <电脑文件> <手机文件>	将电脑中文件推送到手机上(文件名均为全称)
* adb shell	进入手机命令行终端
* adb logcat  打印 Android 的系统日志
* adb bugreport , 打印dumpsys、dumpstate、logcat的输出，也是用于分析错误
* adb bugreport > d:\bugreport.log  输出比较多，建议重定向到一个文件中
* adb shell pm list package	不带任何选项：列出所有的应用的包名（不知道怎么找应用的包名的同学看这里）
* adb shell pm list package -3	列出第三方应用	-s：列出系统应用	命令最后增加 FILTER：过滤关键字，可以很方便地查找自己想要的应用
