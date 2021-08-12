---
layout: post
title: Android系统启动流程及Linux基础知识(一)
date: 2021-06-18
tags: 安卓   
---


本文主要用于个人学习的整理与记录，如有纰漏，望指正。
内容主要来源于JsonChao大佬的文章[Android系统启动流程之init进程启动](https://jsonchao.github.io/2019/02/18/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E4%B9%8Binit%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8/)

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
**属性服务是如何启动的？**
我们查看system/core/init/property_service.cpp源码中的`start_property_service()`函数：
```c++
void start_property_service() {
    selinux_callback cb;
    cb.func_audit = SelinuxAuditCallback;
    selinux_set_callback(SELINUX_CB_AUDIT, cb);

    property_set("ro.property_service.version", "2");

    // 1
    property_set_fd = CreateSocket(PROP_SERVICE_NAME, SOCK_STREAM | SOCK_CLOEXEC | SOCK_NONBLOCK,
                                   false, 0666, 0, 0,  nullptr);
    if (property_set_fd == -1) {
        PLOG(FATAL) << "start_property_service socket creation failed";
    }

    // 2
    listen(property_set_fd, 8);

    // 3、4、5
    register_epoll_handler(property_set_fd, handle_property_set_fd);
}
```
* 1、首先，创建非阻塞式的Socket，并返回`property_set_fd`文件描述符。
### 文件描述符
Linux 系统中，把一切都看做是文件，当进程打开现有文件或创建新文件时，内核向进程返回一个文件描述符，文件描述符就是内核为了高效管理已被打开的文件所创建的索引，用来指向被打开的文件，所有执行I/O操作的系统调用都会通过文件描述符。
详细内容可以查看[文件描述符（File Descriptor）简介](https://segmentfault.com/a/1190000009724931)
* 2、使用listen()函数去监听property_set_fd，此时Socket即成为属性服务端，并且它最多同时可为8个试图设置属性的用户提供服务。
* 3、使用epoll()来监听property_set_fd：当property_set_fd中有数据到来时，init进程将调用handle_property_set_fd()函数进行处理。在Andorid 8.0的源码中则在handle_property_set_fd()函数中添加了handle_property_set函数做进一步封装处理。
* 4、系统属性分为两种属性，即普通属性和控制属性。控制属性用来执行一些命令，比如开机的动画就使用了这种属性。在handle_property_set_fd()函数中会先判断如果属性名是以”ctl.”开头的，就说明是控制属性，如果客户端权限满足，则会调用handle_control_message()函数来修改控制属性。如果是普通属性，则会在客户端全面满足的条件下调用property_set函数来修改普通属性。
* 5、在property_set中会先从属性存储空间中查找该属性，如果有，则更新，否则添加该属性。此外，如果名称是以”ro”开头（表示只读，不能修改），直接返回，如果名称是以”persist.”开头，则写入持久化属性。

### epoll是什么？
在Linux的新内核中，epoll是用来取代select/poll的，它是Linux内核为处理大批量文件描述符的改进版poll，是Linux下多路复用I/O接口select/poll的增强版，它能显著提升程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。
### epoll和select的区别？
epoll内部用于保存事件的数据类型是红黑树，查找速度快，select采用的数组保存信息，查找速度很慢，只有当等待少量文件描述符时，epoll和select的效率才差不多。
**5、解析init.rc配置文件**
```c++
parser.ParseConfig("/init.rc");
```
### init.rc是什么？
它是由Android初始化语言编写的一个非常重要的配置脚本文件。Android初始化语言主要包含5种类型的语句：
* Action（常用）
* Service（常用）
* Command
* Option
* Import
这里了解下Action和Service的格式：
```c++
on <trigger> [&& <trigger>]*     //设置触发器  
    <command>  
    <command>      //动作触发之后要执行的命令
    ...


service <name> <pathname> [ <argument> ]*   //<service的名字><执行程序路径><传递参数>  
    <option>       //option是service的修饰词，影响什么时候、如何启动services  
    <option>  
    ...
```
注意：Android8.0对init.rc文件进行了拆分，每个服务对应一个rc文件。

### init启动Zygote流程？
先看到init.rc的这部分配置代码：
```c++
...
on nonencrypted    
    exec - root -- /system/bin/update_verifier nonencrypted  
    // 1
    class_start main         
    class_start late_start
...
```
1、使用class_start这个COMMAND去启动Zygote。其中class_start对应do_class_start()函数。
```c++
static Result<Success> do_class_start(const BuiltinArguments& args) {
    // Starting a class does not start services which are explicitly disabled.
    // They must be started individually.
    for (const auto& service : ServiceList::GetInstance()) {
        if (service->classnames().count(args[1])) {
            // 2
            if (auto result = service->StartIfNotDisabled(); !result) {
                LOG(ERROR) << "Could not start service'" << service->name()
                           << "' as part of class '" <<  args[1] << "': " <<  result.error();
            }
        }
    }
    return Success();
}
```
2、在system/core/init/builtins.cpp的do_class_start()函数中会遍历前面的Vector类型的Service链表，找到classname为main的Zygote，并调用system/core/init/service.cpp中的startIfNotDisabled()函数。
```c++
bool Service::StartIfNotDisabled() {
    if (!(flags_ & SVC_DISABLED)) {
        return Start();
    } else {
        flags_ |= SVC_DISABLED_START;
    }
    return Success();
}
```
3、如果Service没有再其对应的rc文件中设置disabled选项，则会调用Start()启动该Service。
4、在Start()函数中，如果Service已经运行，则不再启动。如果没有，则使用fork()函数创建子进程，并返回pid值。当pid为0时，则说明当前代码逻辑在子进程中运行，最然后会调用execve()函数去启动子进程，并进入该Service的main函数中，如果该Service是Zygote，则会执行Zygote的main函数。（对应frameworks/base/cmds/app_process/app_main.cpp中的main()函数）
5、最后，调用runtime的start函数启动Zygote。

## 五、总结
经过以上的分析，init进程的启动过程主要分为以下三部：

* 1、创建和挂载启动所需的文件目录。
* 2、初始化和启动属性服务。
* 3、解析init.rc配置文件并启动Zygote进程。
