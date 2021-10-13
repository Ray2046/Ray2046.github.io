---
layout: post
title: AndroidStudio通过WiFi链接手机 & Mac系统制作点击执行脚本
date: 2021-10-13
tags: 安卓
---

## Android Studio 4.1.2 通过wifi连接手机进行adb调试

Android Studio 升级到4.1.2以后也可能是4.1.0以后出现了和Android Wifi ADB插件不兼容的情况。

会提示'Android Wifi ADB' is not compatible

使用插件可以比较方便的实现wifi连接手机进行adb调试，不通过插件也可以实现。

1.手机打开USB调试选项，实现能够通过USB下载调试程序

2.手机通过USB线链接电脑

3.打开命令行窗口，然后切换到adb目录下

`cd C:\Users\XXX…\Android\Sdk\platform-tools`

4.手机和电脑连接同一Wifi，手机能通过手机的设置->关于手机->状态信息->IP地址查看手机的IP地址

5.指定tcp的端口，在 terminal中输入：
`adb tcpip 5555`

6.连接手机，输入指令adb connect IP地址，如：
`adb connect 192.168.31.117`

7.打开Android Studio，查看连接的设备，图中显示表示既可以通过USB又可以通过Wifi调试程序

8.拔掉USB线后剩下一个Wifi连接


## Mac下面通过双击执行shell脚本

1、`touch fileName.sh`

2、`chmod 755 fileName.sh`

3、编辑脚本

4、修改后缀名 `fileName.command`



























-----------
