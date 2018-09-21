---
layout:     post
title:      Linux下利用Eclipse C/C++搭建嵌入式ARM开发环境
subtitle:   Linux下利用Eclipse C/C++搭建嵌入式ARM开发环境
date:       2018-09-17
author:     naiquan.hu
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - ARM
    - Linux
    - eclipse
---


# 1. 系统准备
# 1.1 OS
操心系统：ubuntu-16.04.2-desktop-amd64 或ubuntu-14.04.x-desktop-amd64

# 1.2 安装pip
```
$ sudo apt install python-pip
```
把pip更新到最新：
```
$ sudo pip install --upgrade pip
```
# 1.3 安装Python及多个版本的使用
**此步骤为可选**
安装2.7和3.4两个版本：
```
sudo apt-get install python2.7
sudo apt-get install python3.4
```
切换python版本，在终端输入以下命令： 

```
$ ls -larth `which python`*
-rwxr-xr-x 1 root root  306  2月 20  2014 /usr/bin/pythontex3
lrwxrwxrwx 1 root root   58  2月 20  2014 /usr/bin/pythontex -> ../share/texlive/texmf-dist/scripts/pythontex/pythontex.py
lrwxrwxrwx 1 root root    9  5月 10  2017 /usr/bin/python2 -> python2.7
lrwxrwxrwx 1 root root    9  5月 10  2017 /usr/bin/python -> python2.7
lrwxrwxrwx 1 root root    9  5月 10  2017 /usr/bin/python3 -> python3.4
lrwxrwxrwx 1 root root   10  5月 10  2017 /usr/bin/python3m -> python3.4m
-rwxr-xr-x 1 root root 3.2M 11月 24  2017 /usr/bin/python2.7
-rwxr-xr-x 2 root root 3.6M 11月 29  2017 /usr/bin/python3.4m
-rwxr-xr-x 2 root root 3.6M 11月 29  2017 /usr/bin/python3.4

$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.4 2

$ sudo update-alternatives --config pythonThere are 2 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.4   2         auto mode
  1            /usr/bin/python2.7   1         manual mode
  2            /usr/bin/python3.4   2         manual mode

Press enter to keep the current choice[*], or type selection number: 

```
根据出现的序号，选择你需要的python版本，输入序号回车即可。

如果使用python3.4，则还需要安装pip3:
```
$ sudo apt-get install pip3
```
# 2. 安装JDK
[JDK下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
jdk-8u181-linux-x64.tar.gz
```
$ sudo tar zxvf jdk-8u181-linux-x64.tar.gz  -C /opt
$ vi ~/.bashrc

export JAVA_HOME=/opt/jdk1.8.0_181
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH
```

# 3. 安装Toolchain

Download URL:

https://github.com/gnu-mcu-eclipse/arm-none-eabi-gcc/releases/download/v7.3.1-1.1/gnu-mcu-eclipse-arm-none-eabi-gcc-7.3.1-1.1-20180724-0637-centos64.tgz

下载最新版：gnu-mcu-eclipse-arm-none-eabi-gcc-7.3.1-1.1-20180724-0637-centos64.tgz
```
mkdir ~/opt
$ tar -zxvf gnu-mcu-eclipse-arm-none-eabi-gcc-7.3.1-1.1-20180724-0637-centos64.tgz -C ~/opt/
```

注意：**不需要将工具链路径添加到用户或系统路径！**

如果现有一个工具链只有一个单一版本，那么这不会有问题，但如果有相同工具链的多个版本 ，将会出现问题。所以不设置工具链路径，由GNU MCU Eclipse插件管理工具链。

# 4. 安装SEGGER J-Link
J-Link download URL: https://www.segger.com/downloads/jlink/

下载该**特定版本**：JLink_Linux_V612e_x86_64.tgz。
**注意： 不可下载最新版！**

**安装：**
```
$ sudo mkdir /opt/SEGGER
$ sudo tar -zxvf JLink_Linux_V612e_x86_64.tgz -C /opt/SEGGER/
$ sudo cp /opt/SEGGER/JLink_Linux_V612e_x86_64/99-jlink.rules /etc/udev/rules.d/

$ vi ~/.bashrc

export JLINK_HOME=/opt/SEGGER/JLink_Linux_V612e_x86_64
export PATH=${PATH}:${JAVA_HOME}/bin:${JLINK_HOME}
```


**测试：**
1. 通过JLink连接PC到开发板。
2. 执行命令：


```
$ JLinkExe
SEGGER J-Link Commander V6.12e (Compiled Jan  6 2017 17:21:59)
DLL version V6.12e, compiled Jan  6 2017 17:21:51

Connecting to J-Link via USB...O.K.
Firmware: J-Link V9 compiled Apr 20 2018 16:47:26
Hardware version: V9.20
S/N: -1
License(s): RDI, GDB, FlashDL, FlashBP, JFlash, RDDI
VTref = 3.293V


Type "connect" to establish a target connection, '?' for help
J-Link>connect
Please specify device / core. <Default>: UNSPEC
Type '?' for selection dialog
Device>CORTEX-M4
Please specify target interface:
  J) JTAG (Default)
  S) SWD
TIF>S
Specify target interface speed [kHz]. <Default>: 4000 kHz
Speed>
Device "CORTEX-M4" selected.


Found SWD-DP with ID 0x2BA01477
AP-IDR: 0x24770011, Type: AHB-AP
AHB-AP ROM: 0xE00FF000 (Base addr. of first ROM table)
Found Cortex-M4 r0p1, Little endian.
FPUnit: 6 code (BP) slots and 2 literal slots
CoreSight components:
ROMTbl 0 @ E00FF000
ROMTbl 0 [0]: FFF0F000, CID: B105E00D, PID: 000BB00C SCS
ROMTbl 0 [1]: FFF02000, CID: B105E00D, PID: 003BB002 DWT
ROMTbl 0 [2]: FFF03000, CID: B105E00D, PID: 002BB003 FPB
ROMTbl 0 [3]: FFF01000, CID: B105E00D, PID: 003BB001 ITM
ROMTbl 0 [4]: FFF41000, CID: B105900D, PID: 000BB9A1 TPIU
Cortex-M4 identified.

```

连接成功！

# 5. eclipse安装及配置
# 5.1. 安装eclipse
最简单的方法是从GNU MCU Eclipse GitHub Release下载，可以同时获得Eclipse CDT和MCU所需的插件。

[gnumcueclipse-4.4.1 下载](https://github.com/gnu-mcu-eclipse/org.eclipse.epp.packages/releases/download/v4.4.1-20180721-o3a/20180721-1240-gnumcueclipse-4.4.1-oxygen-3a-linux.gtk.x86_64.tar.gz)

下载的压缩包为20180721-1240-gnumcueclipse-4.4.1-oxygen-3a-linux.gtk.x86_64.tar.gz。
```
tar zxvf 20180721-1240-gnumcueclipse-4.4.1-oxygen-3a-linux.gtk.x86_64.tar.gz -C ~/opt/
$ sudo vi /usr/share/applications/eclipse.desktop

[Desktop Entry]
Name=Eclipse
Type=Application
Exec=/home/username/opt/eclipse/eclipse
Terminal=false
Icon=/home/username/opt/eclipse/icon.xpm
Comment=Integrated Development Environment
NoDisplay=false
Categories=Development;IDE;
Name[en]=Eclipse

$ sudo chmod u+x /usr/share/applications/eclipse.desktop
$ ln -sf $JRE_HOME ~/opt/eclipse/jre
```
than, launch eclipse.

# 5.2. 配置Toolchain和J-Link路径
+ **Eclipse menu --> Window  --> Preferences**

+ **MCU --> Global ARM Toolchains Paths**
   Default toolchain: GNU MCU Eclipse ARM Embedded GCC
   Toolchain name:  GNU MCU Eclipse ARM Embedded GCC
   Toolchain folder:  /home/username/opt/gnu-mcu-eclipse/arm-none-eabi-gcc/7.3.1-1.1-20180724-0637/bin

+ **MCU --> Global SEGGER J-Link Path**
	Executable:  JLinkGDBServer
	Folder:    /opt/SEGGER/JLink_Linux_V612e_x86_64
	
+ **MCU --> Workspace ARM Toolchains Paths**
   Default toolchain: GNU MCU Eclipse ARM Embedded GCC
   Toolchain name:  GNU MCU Eclipse ARM Embedded GCC
   Toolchain folder:  /home/username/opt/gnu-mcu-eclipse/arm-none-eabi-gcc/7.3.1-1.1-20180724-0637/bin

+ **MCU --> Workspace SEGGER J-Link Path**
	Executable:  JLinkGDBServer
	Folder:    /opt/SEGGER/JLink_Linux_V612e_x86_64

+ **Apply and Close**



# 6. 安装工作区偏好设置
## 6.1. 自动保存和文本文件编码
如果编辑了代码忘记了保存，编译运行后是没有任何变化的，所以最好设置自动保存。
+ go to the Eclipse **menu → (Windows →) Preferences → General → Workspace**
+ enable **Save automatically before build**

在同个页面，设置文本编码和行分隔符：
+ in the **Text file encoding** section
+ click the **Other** button
+ select **UTF-8** in the combo box
+ check the **New text file line delimiter** to be set to Unix
+ disable the **Build automatically** option
+ last, click the **Apply** button

## 6.2. 显示行号
+ go to the **Eclipse menu → (Window →) Preferences → General → Editors → Text Editors**
+ enable **Show line numbers**
+ enable **Show print margin**
+ set the **Print margin column** at 80
+ click the **Apply** button

## 6.3. 使用活动构建配置进行索引
Eclipse CDT有一个非常强大的索引器，它可以动态解析源代码，并在编辑时提供提示，自动完成帮助，错误报告等，而无需启动构建过程。要使索引器在从一个配置更改为另一个配置时自动同步，请检查索引器是否设置为使用活动配置：
+ go to the **Eclipse menu → (Window →) Preferences → C/C++ → Indexer**
+ enable **Use active build configuration**
+ click the **Apply** button

## 6.4. 编辑器折叠
为了更好地查看源文件的整个内容，建议禁用编辑器折叠：
+ go to the **Eclipse menu → (Window →) Preferences → C/C++ → Editor → Folding**
+ disable **Header Comments** (in fact it is recommended to disable all foldings)
+ click the **Apply** button

## 6.5. 代码格式
Eclipse CDT具有非常有用的功能，允许自动重新格式化一段代码以符合给定的样式。 推荐的样式是GNU：
+ go to the **Eclipse menu → (Window →) Preferences → C/C++ → Code Style → Formatter**
+ in the **Active profile**: field, select **GNU [built-in]**
+ click the **Edit…** button
+ set **Profile name** to GNU with spaces
+ Change the tab policy to **Spaces only**, and the **indentation** and **tab size** to 2

## 6.6. 编译控制台
+ go to the **Eclipse menu → (Window →) Preferences → C/C++ → Build → Console**
+ enable **Wrap lines on the console**
+ enable **Bring console to top when building**
+ increase the **Limit console output:** field, for example from 500 to 5000 lines
+ click the **Apply** button

## 6.7. Doxygen

To enable support for documentation tool comments:
+ go to the **Eclipse menu → (Window →) Preferences → C/C++ → Editor**
+ select **Doxygen** in the Documentation tool comments
这样，你只需输入/\**，剩余的注释会自动补齐。

## 6.8. 调试前面应用
Eclipse会记住以前使用的调试配置，并可将其作为主调试按钮的默认配置。
+ go to the **Eclipse menu → (Window →) Preferences → Run/Debug → Launching**
+ select **Always launch the previously launched application** in the **Launch Operation** section.

## 6.9. Show source roots
对于具有许多源文件夹的大型项目，您可能希望禁用显示源文件夹：
+ go to the **Eclipse menu → (Window →) Preferences → C/C++ → Appearance**
+ disable **Show source roots at top of project**.


# 7. Create a Hello ARM test project

# 7.1 Create and Build

+ go to Eclipse menu, **File** --> **New**, and select the **C Project**
+ In the **C Project** window, set Project name = hello, Project type = Executable --> **Hello World ARM Cortex-M C/C++ Project**
+ Toolchains: Cross ARM GCC
+ click the **Next->** button
![](\images\project-manager\project-manager-0000.png)

In the **Target processor settings**:
![](\images\project-manager\project-manager-0001.png)

In the **Folders**:
![](\images\project-manager\project-manager-0002.png)

In the **Select Configurations**:
![](\images\project-manager\project-manager-0003.png)

In the **GNU ARM Cross Toolchain**:
![](\images\project-manager\project-manager-0004.png)

+ click the **Finish** button.
+ Modify as the following and build the hello project.
![](\images\project-manager\project-manager-0005.png)


# 7.2 Debug

+ go to Eclipse menu, **Run --> Debug Configurations... **
+ Double click **GDB SEGGER J-Link Debugging** and config as following:
![](\images\project-manager\project-manager-0006.png)
![](\images\project-manager\project-manager-0007.png)
![](\images\project-manager\project-manager-0008.png)
+ click the **Debug** button
![](\images\project-manager\project-manager-0009.png)
![](\images\project-manager\project-manager-0010.png)

# 8. 安装OpenOCD

OpenOCD download URL: 

https://github.com/gnu-mcu-eclipse/openocd/releases/download/v0.10.0-8-20180512/gnu-mcu-eclipse-openocd-0.10.0-8-20180512-1921-centos64.tgz

下载文件：gnu-mcu-eclipse-openocd-0.10.0-8-20180512-1921-centos64.tgz。

解压：

```
$ tar -zxvf gnu-mcu-eclipse-openocd-0.10.0-8-20180512-1921-centos64.tgz -C ~/opt/

$ sudo cp ~/opt/gnu-mcu-eclipse/openocd/0.10.0-8-20180512-1921/contrib/60-openocd.rules /etc/udev/rules.d/
$ sudo udevadm control --reload-rules

$ sudo usermod -aG plugdev $USER

```


+ 更新OpenOCD路径

在Eclipse主菜单中： Window --> Preferences --> MCU --> Global OpenOCD Path

Executable: openocd

Folder:   ~/opt/gnu-mcu-eclipse/openocd/0.10.0-8-20180512-1921/bin

# 9. 安装QEMU
QEMU download URL: 

https://github.com/gnu-mcu-eclipse/qemu/releases/download/v2.8.0-3-20180523/gnu-mcu-eclipse-qemu-2.8.0-3-20180523-0703-centos64.tgz

下载文件：gnu-mcu-eclipse-qemu-2.8.0-3-20180523-0703-centos64.tgz


```
$ tar -zxvf gnu-mcu-eclipse-qemu-2.8.0-3-20180523-0703-centos64.tgz -C ~/opt/
$ ~/opt/gnu-mcu-eclipse/qemu/2.8.0-3-20180523-0703/bin/qemu-system-gnuarmeclipse --version
GNU MCU Eclipse 64-bits QEMU emulator version 2.8.0-3 (v2.8.0-3-20180523-dirty)
Copyright (c) 2003-2016 Fabrice Bellard and the QEMU Project developers

```

+ 更新QEMU路径

在Eclipse主菜单中： Window --> Preferences --> MCU --> Global QEMU Path

Executable: qemu-system-gnuarmeclipse

Folder:   ~/opt/gnu-mcu-eclipse/qemu/2.8.0-3-20180523-0703/bin

# 10. 安装Jumper Virtual Lab
Jumper Virtual lab是一个模拟器，它提供了一种快速简便的方法来运行固件文件。 Jumper的eclipse插件与GNU MCU Eclipse的插件配合使用，使您可以轻松地在虚拟设备上运行固件。

+ 创建一个账户

https://vlab.jumper.io/login

+ 安装Jumper Virtual Lab

如果是python2.7，如下命令：

```

$ sudo pip install jumper --upgrade

```

如果是Python3.4，如下命令：

```

$ sudo pip3 install jumper --upgrade

```

