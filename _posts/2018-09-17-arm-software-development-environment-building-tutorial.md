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

# 7.3. 创建一个具体实例
参见7.1，创建工程 CM4Template，目录结构如下：
```
CM4Template
├── ldscripts
│   └── cortex_m4.ld
└── src
    ├── cm4f
    │   ├── include
    │   │   ├── cmsis_compiler.h
    │   │   ├── cmsis_gcc.h
    │   │   ├── cmsis_version.h
    │   │   ├── core_cm4.h
    │   │   ├── Device.h
    │   │   ├── interrupt.h
    │   │   ├── mpu_armv7.h
    │   │   └── system_Device.h
    │   ├── startup_Device.c
    │   ├── system_Device.c
    │   └── vectable.c
    ├── include
    └── main.c

```
开发板上的RAM起始地址为：0x20000000，大小为200KB。Flash起始地址为：0x10000000，大小为1MB。并且从RAM中直接运行程序。
cortex_m4.ld链接脚本内容：
```
ENTRY(Reset_Handler)

MEMORY
{
    FLASH (rx) : ORIGIN = 0x10000000, LENGTH = 1024K
    RAM (rwx) : ORIGIN = 0x20000000, LENGTH = 100K
}

/* The size of the stack used by application */
IRQ_STACK_SIZE	= 0;
FIQ_STACK_SIZE	= 0;
SVC_STACK_SIZE	= 0;
C_STACK_SIZE	= 1024;
 
SECTIONS
{
    .text :
    {
        KEEP(*(.vector_table))
        *(.text)
    } >RAM
    
    .rodata :
    {
        *(.rodata.*)
    } >RAM

    .data :
    {
        *(.data.*)
    } >RAM

    .bss :
    {
        *(.bss.*)
    } >RAM
	
	.stack :
	{
        __stack_start__ = .;
        . += IRQ_STACK_SIZE;
        . = ALIGN(4);
        __irq_stack_top__ = .;
        
        . += FIQ_STACK_SIZE;
        . = ALIGN(4);
        __fiq_stack_top__ = .;
        
        . += SVC_STACK_SIZE;
        . = ALIGN(4);
        __svc_stack_top__ = .;
        
        . += C_STACK_SIZE;
        . = ALIGN(4);
        __c_stack_top__ = .;
        __stack_end__ = .;
	} > RAM
}  
```

Device.h
```
#ifndef DEVICE_H
#define DEVICE_H

#ifdef __cplusplus
extern "C" {
#endif


/* -------------------------  Interrupt Number Definition  ------------------------ */

typedef enum IRQn
{
/* -------------------  Processor Exceptions Numbers  ----------------------------- */
  NonMaskableInt_IRQn           = -14,     /*  2 Non Maskable Interrupt */
  HardFault_IRQn                = -13,     /*  3 HardFault Interrupt */
  MemoryManagement_IRQn         = -12,     /*  4 Memory Management Interrupt */
  BusFault_IRQn                 = -11,     /*  5 Bus Fault Interrupt */
  UsageFault_IRQn               = -10,     /*  6 Usage Fault Interrupt */
  SVCall_IRQn                   =  -5,     /* 11 SV Call Interrupt */
  DebugMonitor_IRQn             =  -4,     /* 12 Debug Monitor Interrupt */
  PendSV_IRQn                   =  -2,     /* 14 Pend SV Interrupt */
  SysTick_IRQn                  =  -1,     /* 15 System Tick Interrupt */

/* -------------------  Processor Interrupt Numbers  ------------------------------ */
  Interrupt0_IRQn               =   0,
  Interrupt1_IRQn               =   1,
  Interrupt2_IRQn               =   2,
  Interrupt3_IRQn               =   3,
  Interrupt4_IRQn               =   4,
  Interrupt5_IRQn               =   5,
  Interrupt6_IRQn               =   6,
  Interrupt7_IRQn               =   7,
  Interrupt8_IRQn               =   8,
  Interrupt9_IRQn               =   9
  /* Interrupts 10 .. 224 are left out */
} IRQn_Type;


/* ================================================================================ */
/* ================      Processor and Core Peripheral Section     ================ */
/* ================================================================================ */

/* -------  Start of section using anonymous unions and disabling warnings  ------- */
#if   defined (__CC_ARM)
  #pragma push
  #pragma anon_unions
#elif defined (__ICCARM__)
  #pragma language=extended
#elif defined(__ARMCC_VERSION) && (__ARMCC_VERSION >= 6010050)
  #pragma clang diagnostic push
  #pragma clang diagnostic ignored "-Wc11-extensions"
  #pragma clang diagnostic ignored "-Wreserved-id-macro"
#elif defined (__GNUC__)
  /* anonymous unions are enabled by default */
#elif defined (__TMS470__)
  /* anonymous unions are enabled by default */
#elif defined (__TASKING__)
  #pragma warning 586
#elif defined (__CSMC__)
  /* anonymous unions are enabled by default */
#else
  #warning Not supported compiler type
#endif


/* --------  Configuration of Core Peripherals  ----------------------------------- */
#define __CM4_REV                 0x0001U   /* Core revision r0p1 */
#define __MPU_PRESENT             1U        /* MPU present */
#define __VTOR_PRESENT            1U        /* VTOR present */
#define __NVIC_PRIO_BITS          3U        /* Number of Bits used for Priority Levels */
#define __Vendor_SysTickConfig    0U        /* Set to 1 if different SysTick Config is used */
#define __FPU_PRESENT             1U        /* FPU present */

#include "../../src/cm4f/include/core_cm4.h"                       /* Processor and core peripherals */
#include "system_Device.h"                  /* System Header */


/* --------  End of section using anonymous unions and disabling warnings  -------- */
#if   defined (__CC_ARM)
  #pragma pop
#elif defined (__ICCARM__)
  /* leave anonymous unions enabled */
#elif (defined(__ARMCC_VERSION) && (__ARMCC_VERSION >= 6010050))
  #pragma clang diagnostic pop
#elif defined (__GNUC__)
  /* anonymous unions are enabled by default */
#elif defined (__TMS470__)
  /* anonymous unions are enabled by default */
#elif defined (__TASKING__)
  #pragma warning restore
#elif defined (__CSMC__)
  /* anonymous unions are enabled by default */
#else
  #warning Not supported compiler type
#endif


#ifdef __cplusplus
}
#endif

#endif  /* DEVICE_H */
```
system_Device.h
```
#ifndef SYSTEM_DEVICE_H
#define SYSTEM_DEVICE_H

#ifdef __cplusplus
extern "C" {
#endif

extern uint32_t SystemCoreClock;     /*!< System Clock Frequency (Core Clock) */


/**
  \brief Setup the microcontroller system.

   Initialize the System and update the SystemCoreClock variable.
 */
extern void SystemInit (void);


/**
  \brief  Update SystemCoreClock variable.

   Updates the SystemCoreClock with current core Clock retrieved from cpu registers.
 */
extern void SystemCoreClockUpdate (void);

#ifdef __cplusplus
}
#endif

#endif /* SYSTEM_DEVICE_H */
```
interrupt.h
```
#ifndef _INTERRUPT_H_
#define _INTERRUPT_H_

#define WEAK __attribute__ ((weak))

typedef void (*intfun_t)();
typedef struct
{
    unsigned long *tos;
    intfun_t      vectors[101];
}__vector_table_t;

void Reset_Handler();

WEAK void NMI_Handler();
WEAK void HardFault_Handler();
WEAK void MemManage_Handler();
WEAK void BusFault_Handler();
WEAK void UsageFault_Handler();
WEAK void SVC_Handler();
WEAK void DebugMon_Handler();
WEAK void PendSV_Handler();
WEAK void SysTick_Handler();
#endif /* _INTERRUPT_H_ */
```
startup_Device.c
```
#include <stdint.h>

extern int main (void);
#if 0
extern const uint32_t _BSS_BEGIN;
extern const uint32_t _BSS_END;
extern const uint32_t _DATA_FLASH_BEGIN;
extern const uint32_t _DATA_FLASH_END;
extern const uint32_t _DATA_RAM_BEGIN;
extern const uint32_t _DATA_RAM_END;

static void initialize_bss_section (void)
{
    uint8_t* begin = (uint8_t*) &_BSS_BEGIN;
    uint8_t* end = (uint8_t*) &_BSS_END;

    for (uint8_t* bss_byte = begin; bss_byte < end; bss_byte++)    {
        *bss_byte = 0;
    }
}

static void initialize_data_section (void)
{
    uint8_t* begin = (uint8_t*) &_DATA_RAM_BEGIN;
    uint8_t* end = (uint8_t*) &_DATA_RAM_END;
    uint8_t* data_flash_byte = (uint8_t*) &_DATA_FLASH_BEGIN;

    for (uint8_t* data_ram_byte = begin; data_ram_byte < end; data_ram_byte++)
    {
        *data_ram_byte = *data_flash_byte;
        data_flash_byte++;
    }
}
#endif

void Reset_Handler()
{
  //initialize_bss_section ();
  //initialize_data_section ();

  SystemInit();

  main();
}

void NMI_Handler (void)
{
    while (1)    {
        // ...
    }
}

void HardFault_Handler (void)
{
    while (1)    {
        // ...
    }
}

void MemManage_Handler (void)
{
    while (1)    {
        // ...
    }
}

void BusFault_Handler (void)
{
    while (1)    {
        // ...
    }
}

void UsageFault_Handler (void)
{
    while (1)    {
        // ...
    }
}

void SVC_Handler (void)
{
    while (1)    {
        // ...
    }
}


void DebugMon_Handler (void)
{
    while (1)    {
        // ...
    }
}

void PendSV_Handler (void)
{
    while (1)    {
        // ...
    }
}

void SysTick_Handler (void)
{
    while (1)    {
        // ...
    }
}
```
system_Device.c
```
#include "interrupt.h"
#include "Device.h"
/*----------------------------------------------------------------------------
  Define clocks
 *----------------------------------------------------------------------------*/
#define  XTAL            (50000000UL)     /* Oscillator frequency */

#define  SYSTEM_CLOCK    (XTAL / 2U)


/*----------------------------------------------------------------------------
  Externals
 *----------------------------------------------------------------------------*/
#if defined (__VTOR_PRESENT) && (__VTOR_PRESENT == 1U)
  extern __vector_table_t vector_table;
#endif

/*----------------------------------------------------------------------------
  System Core Clock Variable
 *----------------------------------------------------------------------------*/
uint32_t SystemCoreClock = SYSTEM_CLOCK;  /* System Core Clock Frequency */


/*----------------------------------------------------------------------------
  System Core Clock update function
 *----------------------------------------------------------------------------*/
void SystemCoreClockUpdate (void)
{
  SystemCoreClock = SYSTEM_CLOCK;
}

/*----------------------------------------------------------------------------
  System initialization function
 *----------------------------------------------------------------------------*/
extern const uint32_t _estack;
void
SystemInit (void)
{

#if defined (__VTOR_PRESENT) && (__VTOR_PRESENT == 1U)
  SCB->VTOR = (uint32_t) &vector_table;
#endif
  //asm(" LDR r0, =_estack");
  //asm(" MSR msp, r0");
#if defined (__FPU_USED) && (__FPU_USED == 1U)
  SCB->CPACR |= ((3U << 10U*2U) | /* enable CP10 Full Access */
      (3U << 11U*2U) ); /* enable CP11 Full Access */
#endif

#ifdef UNALIGNED_SUPPORT_DISABLE
  SCB->CCR |= SCB_CCR_UNALIGN_TRP_Msk;
#endif

  SystemCoreClock = SYSTEM_CLOCK;
}
```
vectable.c
```
#include "interrupt.h"

//------------------------------------------------------------------------------
extern unsigned int __stack_end__;

const __vector_table_t vector_table __attribute__((section(".vector_table")))=

{
    (unsigned long *)&__stack_end__,

    {
    Reset_Handler,

    //--------------------------------------------------------------------------
    //
    // Cortex-M core exceptions
    //
    NMI_Handler,
    HardFault_Handler,
    MemManage_Handler,
    BusFault_Handler,
    UsageFault_Handler,
    0,                          // Reserved
    0,                          // Reserved
    0,                          // Reserved
    0,                          // Reserved
    SVC_Handler,
    DebugMon_Handler,
    0,                          // Reserved
    PendSV_Handler,             // The OS context switch interrupt
    SysTick_Handler,            // The OS timer
    0,
    }
};
```
main.c
```
#include <stdio.h>
#include <stdlib.h>

int
main (void)
{
  return 0;
}
```
之后，编译，debug，主要，在从RAM运行程序时，disable the **Pre-run reset and halt** option in the **Startup** tab
![](\images\project-manager\project-manager-0011.png)

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

