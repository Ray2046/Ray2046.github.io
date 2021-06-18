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
Linux系统的用户进程，是所有用户进程的鼻祖，进程号为1，它有许多重要的职责，比如创建Zygote孵化器和属性服务等。并且它是由多个源文件组成的，对应源码目录`system/core/init`中。
### 2、init进程启动核心代码流程分析

```c++
int main(int argc, char** argv) {
    ...
    // 如果是初始化第一阶段，则需要执行下面的步骤1
    if (is_first_stage) {
        ...
        // 清理umask
        umask(0);
        ...
        // 1、创建和挂载启动所需的文件目录
        mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
        mkdir("/dev/pts", 0755);
        mkdir("/dev/socket", 0755);
        mount("devpts", "/dev/pts", "devpts", 0, NULL);
        #define MAKE_STR(x) __STRING(x)
        mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
        ...
        // 初始化Kernel的Log，获取外界的Kernel日志
        InitKernelLogging(argv);
        ...
    }
    // 初识化Kernel的Log，获取外界的Kernel日志
    InitKernelLogging(argv);
    ...
    // 2、初始化属性相关资源
    property_init();
    ...
    // 创建epoll句柄
    epoll_fd = epoll_createl(EPOLL_CLOEXEC);
    ...
    // 3、设置子信号处理函数
    sigchld_handler_init();
    // 导入默认的环境变量
    property_load_boot_defaults();
    // 4、启动属性服务
    start_property_service();
    set_usb_controller();
    ...
    // 加载引导脚本
    LoadBootScripts(am, sm);
    ...   
   while (true) {
       ...
       if (!(waiting_for_prop || Service::is_exec_service_running())) {
           // 内部会偏离执行每个action中携带的command对应的执行函数
           am.ExecuteOneCommand();
       }
       if (!(waiting_for_prop || Service::is_exec_service_running())) {
           if (!shutting_down) {
               // 重启死去的子进程
               auto next_process_restart_time = RestartProcesses();
               ...
           }
           // If there's more work to do, wake up again immediately.
           if (am.HasMoreCommands()) epoll_timeout_ms = 0;
       }
       epoll_event ev;
       int nr = TEMP_FAILURE_RETRY(epoll_wait(epoll_fd, &ev, 1, epoll_timeout_ms));
       if (nr == -1) {
           PLOG(ERROR) << "epoll_wait failed";
       } else if (nr == 1) {
           ((void (*)()) ev.data.ptr)();
       }
   }
   return 0;
}
static void LoadBootScripts(ActionManager&action_manager, ServiceList& service_list) {
    Parser parser = CreateParser(action_manager, service_list);

    std::string bootscript = GetProperty("ro.boot.init_rc", "");
    // bootscript默认是空的
    if (bootscript.empty()) {
        // 5、解析init.rc配置文件
        parser.ParseConfig("/init.rc");
        if (!parser.ParseConfig("/system/etc/init")) {
            late_import_paths.emplace_back("/system/etc/init");
        }
        if (!parser.ParseConfig("/product/etc/init")) {
            late_import_paths.emplace_back("/product/etc/init");
        }
        if (!parser.ParseConfig("/odm/etc/init")) {
            late_import_paths.emplace_back("/odm/etc/init");
        }
        if (!parser.ParseConfig("/vendor/etc/init")) {
            late_import_paths.emplace_back("/vendor/etc/init");
        }
    } else {
        parser.ParseConfig(bootscript);
    }
}
```

**1、创建和挂载启动所需的文件目录**
其中挂载了tmpsf、devpts、proc、sysfs和selinuxfs共5种文件系统（它们均是系统运行时目录）：
**2、对属性服务进行初始化**
```
property_init();
```

### 什么是属性服务？
Windows平台上有一个注册表管理器，注册表的内容采用键值对的形式来记录用户、软件等使用信息。如果系统或软件重启，还是能够根据这份注册表中的记录，进行相应的初识化工作。Android也提供了一个这样类型的机制，即属性服务。

**它具体是如何进行初始化的？**
我们查看system/core/init/property_service.cpp源码中的`property_init()`函数：
```c++
void property_init() {
    mkdir("/dev/__properties__", S_IRWXU | S_IXGRP S_IXOTH);
    CreateSerializedPropertyInfo();
    // 关注点
    if (__system_property_area_init()) {
        LOG(FATAL) << "Failed to initialize property area";
    }
    if (!property_info_area.LoadDefaultPath()) {
        LOG(FATAL) << "Failed to load serialized property info file";
    }
}
```
init进程启动时会启动属性服务，并为其分配内存，用来存储这些属性，如果需要就可以直接读取，具体在代码里就是执行了property_init()函数中的__system_property_area_init()函数去初始化属性内存区域。

**3、设置子进程信号处理函数，如果子进程（zygote进程）异常退出，init进程会调用该函数中设定的信号处理函数来进行处理**
```
sigchld_handler_init();
```
**sigchld_handler_init()的作用：**
防止init进程的子进程成为僵尸进程，为了防止僵尸进程的出现，系统会在子进程暂停和终止的时候发出SIGCJHLD信号，该函数就是用来接收SIGCHLD信号的，注意它仅处理进程终止的SIGCHLD信号。

### 僵尸进程是什么？
在UNIX/Linux中，父进程使用fork创建子进程，子进程终止后，如果父进程不知道子进程已经终止的话，这时子进程虽然已经退出，但是在系统进程表中还为它保留了一些信息（如进程号、运行时间、退出状态等），这个子进程就是所谓的僵尸进程。其中系统进程表是一项有限的资源，如果它被僵尸进程耗尽的话，系统可能会无法创建新的进程。
### 如果是Zygote进程终止了，则会如何？
sigchld_handler_init()函数内部会找到Zygote进程并移除所有的Zygote进程的信息，在重启Zygote服务的启动脚本（如init.zygote64.rc）中带有onrestart选项的服务。

**4、启动属性服务（其中会启动servicemanager(binder服务大管家)、bootanim(开机动画)等重要服务）**
```
start_property_service();
```




博客迭代信息请看[ReleaseNode](https://leopardpan.cn/2020/07/ReleaseNode/)
