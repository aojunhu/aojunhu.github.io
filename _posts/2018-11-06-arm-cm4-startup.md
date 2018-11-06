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


# 4. ARM系统启动过程
# 4.1. 地址划分
boot_memory_base.h
```
#ifndef BOOT_MEMORY_BASE_H
#define BOOT_MEMORY_BASE_H
#define BOOT_ROM_BASE               0x00000000
#define BOOT_ROM_LIMIT              0x1000
#define ROM_CODE_LIMIT              0x10000                 //64K
#define BOOT_ROM_DATABASE           0x20010000          // 8K
#define BOOT_ROM_DATALIMIT      0x2000
#define BOOT_HEADER_SIZE            24                          //refer to boot_type.h
#define BOOT_RAM_CODEBASE           0x20012000          // 4K
#define BOOT_RAM_DATABASE           0x20013000          // 4K
#define UART_DOWNLOAD_CODEBASE  0x20012000
#define UART_DOWNLOAD_DATABASE  0x20013000
#define RAM_BASE                                0x1fff0000
#define RAM_TOP                               0x20032000
#define FLASH_CACHE_BASE                0x10000000
#define FLASH_IMAGE_OFFSET          0x3000
#define RETENTION_MEM_BASE          0x20028000
#define RETENTION_MEM_SIZE          0x2000
#define CACHE_MEM_BASE                  0x2002A000
#define CACHE_MEM_SIZE                  0x8000
#endif
```
![](/images/blog/00011.png)
## 4.2. Level-1 Boot
第一阶段BootLoader在ROM中，ROM起始地址放中断向量表，往后依次放入代码段和数据段，总大小为12KB，其中，数据段的执行域在RAM中。如图橙色部分。
```
typedef struct{
    uint32_t art_flag; 
    uint8_t *target_address;
    uint16_t ram_header_length;
    uint16_t length;
    uint32_t bootram_crc;
    entry_point_t entry_point;
    uint32_t crp_value;
}boot_header_t;
int main()
{
    uint32_t crp_flag;
    flash_init();
    flash_read(CRP_FLASH_OFFSET,sizeof(uint32_t),(uint8_t *)(&crp_flag));       // first 4 bytes is crp flag
    if(crp_flag != CRP_VALID_FLAG)
    {
        // function io set for swd
        sysc_cmp_gpioa_en_2_setf(0);           
        sysc_cmp_gpioa_en_3_setf(0);
        #ifdef BOOT_ROM_DEBUG
        LOG(LOG_LVL_INFO,"crp_invalid SWD out\n");
        #endif
    }
    #ifdef BOOT_ROM_DEBUG
    LOG(LOG_LVL_INFO,"enter main......\n");
    #endif
   
    boot_stat.rx_done= false;
    boot_mode.boot_source = (boot_option_t)sysc_awo_i_boot_mode_getf();
   
    #ifdef FPGA_SET_BOOTMODE
    boot_mode.boot_source = BOOT_FROM_FLASH;
    #endif
   
    boot_mode_set(boot_mode.boot_source);
    while(1)
    {
        boot_header_rx_start();
        while(boot_stat.rx_done==false);
        if(boot_stat.crc_valid == true)
            break;
        else
            boot_stat.rx_done = false;
    }
    boot_header.entry_point();
    return 1;
}
```
![](/images/blog/00012.png)
## 4.3. Level-2 Boot
二级BootLoader从UART或者flash加载到RAM中运行，所以它的加载域和执行域是RAM中的相同位置。因此，MDK project的链接文件如下：
```
#! armcc -E
#include "..\boot_memory_base.h"
LOAD_BOOTRAM BOOT_RAM_CODEBASE - BOOT_HEADER_SIZE  {    ; load region size_region
  BOOT_RAM_EXEC +0  {  ; load address = execution address
        *(boot_header_area,+First)  ;
        *(+RO,+RW,+ZI)
  }
}
```
![](/images/blog/00013.png)
开始的24 bytes存放的是如下内容：
```
const boot_info_t boot_info BOOT_HEADER_ATTRIBUTE = {
    .boot_header = {
        .art_flag = FLASH_VALID_FLAG,
        .target_address = (uint8_t *)BOOT_RAM_CODEBASE,
        .ram_header_length = sizeof(boot_info_t)-sizeof(boot_header_t),
        .length = 0xffff,
        .bootram_crc = TO_BE_FILLED,
        .entry_point = boot_ram_entry,
        .crp_value = crp_flag,
    },
    .sys_nvds_offset = 0x1000,
    .sys_nvds_len = 0x1000,
    .sys_nvds_backup = 1,
    .rfu1 = TO_BE_FILLED,
    .rfu2 = TO_BE_FILLED,
};
```
Image 的Image Entry point : boot_ram_entry
Flash开始的12KB存放二级boot的image文件
![](/images/blog/00014.png)
二级boot生成的image文件格式如下：
![](/images/blog/00015.png)
其中，boot_info_t的长度为0x28。该image文件还需做一下格式转换。
1. 获取img文件总长度lSize。
2. Seek到offset为10处，并写入lSize，其实是设置boot_header_t中的length:
```
    .boot_header = {
        .art_flag = FLASH_VALID_FLAG,
        .target_address = (uint8_t *)BOOT_RAM_CODEBASE,
        .ram_header_length = sizeof(boot_info_t)-sizeof(boot_header_t),
        .length = 0xffff,
        .bootram_crc = TO_BE_FILLED,
        .entry_point = boot_ram_entry,
        .crp_value = crp_flag,
    },
```

3. 分配buffer，大小为image_size – sizeof(boot_info_t).
4. 从OFFSET为0x28处开始读取整个image内容到buffer中。
5. 计算CRC值。
6. 把CRC值写入boot_header_t中：
```
    .boot_header = {
        .art_flag = FLASH_VALID_FLAG,
        .target_address = (uint8_t *)BOOT_RAM_CODEBASE,
        .ram_header_length = sizeof(boot_info_t)-sizeof(boot_header_t),
        .length = 0xffff,
        .bootram_crc = TO_BE_FILLED,
        .entry_point = boot_ram_entry,
        .crp_value = crp_flag,
    },
```
至此，生成了修改以后的boot_ram.bin。该image会在后面生成flash image时，放入flash的0x0-0x1000的位置。
一级boot的最后，会引导进入boot_ram_entry()函数。下面来看看该函数的流程。
![](/images/blog/00016.png)
在调用Reset_Handler函数时，会传递参数进去，所以Reset_Handler函数中要用R4寄存器。
![](/images/blog/00017.png)
```
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  SystemInit
                IMPORT  __main
                LDR     R4, =SystemInit
                BLX     R4
                LDR     R4, =__main
                BX      R4
                ENDP
```
至此，已经引导进入user main函数。