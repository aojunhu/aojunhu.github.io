---
layout:     post
title:      ARM(Cortex-M4)启动过程
subtitle:   ARM(Cortex-M4)启动过程
date:       2018-11-06
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- arm
- boot
---

ARM的启动过程通常包括以下部分：
1. 外部硬件reset
2. 根据BOOT MODE进入不同的启动入口
3. 向量表定义
4. 地址重映射及中断向量表的转移
5. 堆栈初始化
6. 设置系统时钟频率
7. 中断寄存器的初始化
8. 进入C应用程序

# 1. 外部硬件reset
按压reset button后，系统复位。
![](/images/blog/00001.png)
# 2. 根据BOOT MODE进入不同的启动入口
BOOT MODE是由PCB设计决定的，根据BOOT0和BOOT1引脚的电压高低决定进入哪种模式：
![](/images/blog/00002.png)

![](/images/blog/00003.png)

# 3. 向量表定义
Cortex-M4复位后首先从默认向量表处读取SP初始值和PC初始值。为了让Cortex-M4复位后立即有可用的复位向量，必须将向量表存储在ROM中，然后在初始化过程可选地将向量表地址映射到其它区域。
向量表的定义如下：
```
__Vectors       DCD     __initial_sp              ; Top of Stack
                DCD     Reset_Handler             ; Reset Handler
                DCD     NMI_Handler               ; NMI Handler
                DCD     HardFault_Handler         ; Hard Fault Handler
                DCD     MemManage_Handler         ; MPU Fault Handler
                DCD     BusFault_Handler          ; Bus Fault Handler
                DCD     UsageFault_Handler        ; Usage Fault Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     SVC_Handler               ; SVCall Handler
                DCD     DebugMon_Handler          ; Debug Monitor Handler
                DCD     0                         ; Reserved
                DCD     PendSV_Handler            ; PendSV Handler
                DCD     SysTick_Handler           ; SysTick Handler

                ; External Interrupts
                DCD     WDT_IRQHandler            ;  0:  Watchdog Timer
                DCD     RTC_IRQHandler            ;  1:  Real Time Clock
                DCD     TIM0_IRQHandler           ;  2:  Timer0 / Timer1
                DCD     TIM2_IRQHandler           ;  3:  Timer2 / Timer3
                DCD     MCIA_IRQHandler           ;  4:  MCIa
                DCD     MCIB_IRQHandler           ;  5:  MCIb
                DCD     UART0_IRQHandler          ;  6:  UART0 - DUT FPGA
                DCD     UART1_IRQHandler          ;  7:  UART1 - DUT FPGA
                DCD     UART2_IRQHandler          ;  8:  UART2 - DUT FPGA
                DCD     UART4_IRQHandler          ;  9:  UART4 - not connected
                DCD     AACI_IRQHandler           ; 10: AACI / AC97
                DCD     CLCD_IRQHandler           ; 11: CLCD Combined Interrupt
                DCD     ENET_IRQHandler           ; 12: Ethernet
                DCD     USBDC_IRQHandler          ; 13: USB Device
                DCD     USBHC_IRQHandler          ; 14: USB Host Controller
                DCD     CHLCD_IRQHandler          ; 15: Character LCD
                DCD     FLEXRAY_IRQHandler        ; 16: Flexray
                DCD     CAN_IRQHandler            ; 17: CAN
                DCD     LIN_IRQHandler            ; 18: LIN
                DCD     I2C_IRQHandler            ; 19: I2C ADC/DAC
                DCD     0                         ; 20: Reserved
                DCD     0                         ; 21: Reserved
                DCD     0                         ; 22: Reserved
                DCD     0                         ; 23: Reserved
                DCD     0                         ; 24: Reserved
                DCD     0                         ; 25: Reserved
                DCD     0                         ; 26: Reserved
                DCD     0                         ; 27: Reserved
                DCD     CPU_CLCD_IRQHandler       ; 28: Reserved - CPU FPGA CLCD
                DCD     0                         ; 29: Reserved - CPU FPGA
                DCD     UART3_IRQHandler          ; 30: UART3    - CPU FPGA
                DCD     SPI_IRQHandler            ; 31: SPI Touchscreen - CPU FPGA
__Vectors_End
```
![](/images/blog/00004.png)
![](/images/blog/00005.png)
上图中，复位后，从地址0x00000000处取出0x20001BB0放入R13（MSP）中，是栈顶指针。从0x00000004处取出值0x08000569放入R15（PC）中，
0x08000569最低为为1，代表为thumb指令，0x08000568为将要执行指令的地址，该地址就是Reset_Hnadler处理函数的入口地址，可参加下面代码：
```
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  SystemInit
                IMPORT  check_update_requirement
                IMPORT  __main
                LDR     R0, =SystemInit
                BLX     R0
                LDR     R0, =check_update_requirement
                BLX     R0
                cbz     R0, RunUserCode
                LDR     R0, =__main
                BX      R0
                ENDP
```
进入Reset_Handler后，首先进入SystemInit，做系统环境的初始化（clock，ram，Flash等控制器）。
然后进入__main，这是系统函数，直接跳转到 __scatterload，__scatterload 执行代码和数据复制以及 ZI 数据的清零。
根据分散加载文件，拷贝RW数据到RAM,在RAM空间里建立ZI的数据空间，建立运行时的映像存储器映射.然后跳转到 __rt_entry（运行时的入口）则负责初始化 C 库。还设置应用程序的栈和堆，初始化库函数及其静态数据。这时应用程序的堆栈建立了，跳转到main()函数，运行用户代码。
![](/images/blog/00006.png)
![](/images/blog/00007.png)
![](/images/blog/00008.png)
![](/images/blog/00009.png)
![](/images/blog/00010.png)