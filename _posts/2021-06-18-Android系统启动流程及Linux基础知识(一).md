---
layout: post
title: Android系统启动流程及Linux基础知识(一)
date: 2021-06-18
tags: Android   
---


本文主要用于个人学习的整理与记录，如有纰漏，望见谅。

Android系统启动流程共分为四部分：
* init进程启动
* Zygote进程启动
* SystemServer进程启动
* Launcher进程启动

先给出一幅Gityuan博客中的一幅系统启动架构图来对Android系统的启动流程有一个宏观的把控。
![cmd-markdown-logo](http://gityuan.com/images/android-arch/android-boot.jpg)

## 一、启动电源以及系统启动
当电源按下时引导芯片代码会从预定义的地方（固化在ROM）开始执行，加载引导程序BootLoader到RAM，然后执行。

## 二、引导程序BootLoader
它是Android操作系统开始运行前的一个小程序，主要将操作系统OS拉起来并进行。

## 三、Linux内核启动
当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。此外，还启动了Kernel的swapper进程（pid = 0）和kthreadd进程（pid = 2）。下面分别介绍下它们：
### swapper进程
swapper进程：又称为idle进程，系统初始化过程Kernel由无到有开创的第一个进程, 用于初始化进程管理、内存管理，加载Binder Driver、Display、Camera Driver等相关工作。
### kthreadd进程
kthreadd进程：Linux系统的内核进程，是所有内核进程的鼻祖，会创建内核工作线程kworkder，软中断线程ksoftirqd，thermal等内核守护进程。

当内核完成系统设置时，它首先在系统文件中寻找`init.rc文件`，并启动init进程。

## 四、init进程启动
init进程主要用来初始化和启动属性服务，并且启动Zygote进程。
### 1、init进程是什么？



**重要字段说明**





博客迭代信息请看[ReleaseNode](https://leopardpan.cn/2020/07/ReleaseNode/)
