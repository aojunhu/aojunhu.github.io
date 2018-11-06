---
layout:     post
title:      阅读 Android 系统源码
subtitle:   阅读 Android 系统源码
date:       2018-11-06
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
- Android
- linux
---

由于工作需要大量修改framework代码, 在AOSP(Android Open Source Project)源码上花费了不少功夫, Application端和Services端都看和改了不少.
如果只是想看看一些常用类的实现, 在Android包管理器里把源码下载下来, 随便一个IDE配好Source Code的path看就行. 
但如果想深入的了解Android系统, 那么可以看下我的一些简单的总结. 



# 知识

+ Java
Java是AOSP的主要语言之一. 没得说, 必需熟练掌握.
+ 熟练的Android App开发
+ Linux
Android基于Linux的, 并且AOSP的推荐编译环境是Ubuntu 12.04. 所以熟练的使用并了解Linux这个系统是必不可少的. 如果你想了解偏底层的代码, 那么必需了解基本的Linux环境下的程序开发. 如果再深入到驱动层, 那么Kernel相关的知识也要具备.
+ Make
AOSP使用Make系统进行编译. 了解基本的Makefile编写会让你更清晰了解AOSP这个庞大的项目是如何构建起来的.
+ Git
AOSP使用git+repo进行源码管理. 这应该是程序员必备技能吧.
+ C++
Android系统的一些性能敏感模块及第三方库是用C++实现的, 比如: Input系统, Chromium项目(WebView的底层实现).

# 硬件

+ 流畅的国际网络
AOSP代码下载需要你拥有一个流畅的国际网络. 如果在下载代码这一步就失去耐心的话, 那你肯定没有耐心去看那乱糟糟的AOSP代码. 另外, 好程序员应该都会需要一个流畅的Google.
+ 一台运行Ubuntu 12.04的PC.
如果只是阅读源码而不做太多修改的话, 其实不需要太高的配置.
+一台Nexus设备
AOSP项目默认只支持Nexus系列设备. 没有也没关系, 你依然可以读代码. 但如果你想在大牛之路走的更远, 还是改改代码, 然后刷机调试看看吧.
+ 高品质USB线
要刷机时线坏了, 没有更窝心的事儿了.

# 软件
+ Ubuntu 12.04
官方推荐, 没得选.
+ Oracle Java 1.6
注意不要用OpenJDK. 这是个坑, 官方文档虽然有写, 但还是单独提一下.
安装:

```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java6-installer
sudo apt-get install oracle-java6-set-default
```
+ Eclipse
估计会有不少人吐槽, 为什么要用这个老古董. 其实原因很简单, 合适. 刚开始搞AOSP时, 为了找到效率最优的工具, 我尝试过Eclipse, IntelliJ IDEA, Vim+Ctags, Sublime Text+Ctags. 最终结果还是Eclipse. 主要优点有:
1. 有语法分析 (快速准确的类, 方法跳转).
2. 支持C++ (IntelliJ的C++支持做的太慢了).
3. 嵌入了DDMS, View Hierarchy等调试工具.
为了提高效率, 花5分钟背下常用快捷键非常非常值得.
调整好你的classpath, 不要导入无用的代码. 因为AOSP项目代码实在是太多了. 当你还不需要看C++代码时, 不要为项目添加C++支持, 建索引过程会让你崩溃.
+ Intellij IDEA
开发App必备. 当你要调试系统的某个功能是, 常常需要迅速写出一个调试用App, 这个时候老旧的Eclipse就不好用了. Itellij IDEA的xml自动补全非常给力.

# 巨人的肩膀
+ AOSP项目官方: https://source.android.com/source/index.html
这个一定要先读. 项目介绍, 代码下载, 环境搭建, 刷机方法, Eclipse配置都在这里. 这是一切的基础.
+ Android官方Training: https://developer.android.com/training/index.html
这个其实是给App开发者看的. 但是里面也有不少关于系统机制的介绍, 值得细读.
+ 老罗的Android之旅: http://blog.csdn.net/luoshengyang
此老罗非彼老罗. 罗升阳老师的博客非常有营养, 基本可以作为指引你开始阅读AOSP源码的教程. 你可以按照博客的时间顺序一篇篇挑需要的看.但这个系列的博客有些问题:
早期的博客是基于旧版本的Android;
大量的代码流程追踪. 读文章时你一定要清楚你在看的东西在整个系统处于什么样的位置.
+ Innost的专栏: http://blog.csdn.net/innost
邓凡平老师也是为Android大牛, 博客同样很有营养. 但是不像罗升阳老师的那么系统. 更多的是一些技术点的深入探讨.
+ Android Issues: http://code.google.com/p/android/issues/list
Android官方Issue列表. 我在开发过程中发现过一些奇怪的bug, 最后发现这里基本都有记录. 当然你可以提一些新的, 有没有人改就是另外一回事了.
+ Google: https://www.google.com
 一定要能流畅的使用这个工具. 大量的相关知识是没有人系统的总结的, 你需要自己搞定.
# 其它
+ 代码组织
AOSP的编译单元不是和git项目一一对应的, 而是和Android.mk文件一一对应的. 善用mmm命令进行模块编译将节省你大量的时间.
+ Binder
这是Android最基础的进程间通讯. 在Application和System services之间大量使用. 你不仅要知道AIDL如何使用, 也要知道如何手写Binder接口. 这对你理解Android的Application和System services如何交互有非常重要的作用. Binder如何实现的倒不必着急看.
+ HAL
除非你对硬件特别感兴趣或者想去方案公司上班, 否则别花太多时间在这一层.
+ CyanogenMod
这是一个基于AOSP的第三方Rom. 从这个项目的wiki里你能学到很多AOSP官方没有告诉你的东西. 比如如何支持Nexus以外的设备.
+ DIA
这是一个Linux下画UML的工具, 能够帮你梳理看过的代码.
+ XDA
http://www.xda-developers.com/
这里有最新资讯和最有趣的论坛.
+ 想到了再补充.

