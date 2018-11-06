---
layout:     post
title:      解决Ubuntu下adb不能连接某些厂家的设备问题
subtitle:   解决Ubuntu下adb不能连接某些厂家的设备问题
date:       2018-11-06
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
- adb
- shell
- linux
---

# １．查看vendor ID
```
$ lsusb
Bus 002 Device 030: ID 1f3a:1007
```

# 2．编辑如下文件（如果没有就创建一个）
```
sudo vi /etc/udev/rules.d/51-android.rules

SUBSYSTEM=="usb", ATTRS{idVendor}=="1f3a", ATTRS{idProduct}=="1007",MODE="0666"
```
# ３．然后保存退出，再设置一下权限
```
sudo chmod a+rx /etc/udev/rules.d/51-android.rules
```
# 4. 重启udev和adb server
```
sudo /etc/init.d/udev restart
adb kill-server
adb start-server
```

另外，有些shell脚本执行出错问题解决：
```
sudo dpkg-reconfigure dash
```
选择NO 