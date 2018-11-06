---
layout:     post
title:      连接器 -- Scatter File & Linker Script File
subtitle:   连接器 -- Scatter File & Linker Script File
date:       2018-11-06
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
- ARM
---
ARM 映像文件是一个层次结构文件， 包含域（region）， 输出段（output section）和输入段（input section）
# 映像文件组成
```
域1 //加载时对应的一块存储区域--rom中
{
     输出段1 //运行时对应的一块存储区域  -- RAM或ROM中
     {
          输入段1 （编译出来各个.o）
          输入段2
          多个输入段（包含代码段和数据段，可能是RO、RW、ZI）
     }
     输出段2
     {
     }
     //最多三个输出段
}
域2
{
     ....
}
```
一个域通常映射到一个物理存储器上，如ROM/RAM等。
文件中各部分在存储系统中地址有两种：
**加载时地址**
映像文件位于存储器中， 运行前的地址。举个例子， 字符串 uint8* hell = "hell world"编译后，运行前保存在ROM中的地址 A， 在运行时被加载到内存中 B， 这里 A 就是加载时地址， 而程序运行时读取的地址时B， 也就是下面的运行时地址。
**运行时地址**
映像文件运行后加载到存储器的地址。
例如，编译好的映像文件存放于ROM中，即RW段和RO段的加载地址都位于ROM中。运行的时候会把RW段从ROM搬到RAM中，同时在RAM中开辟ZI段，即RO段运行时地址在ROM中，RW段、ZI段运行时地址在RAM中。
区域|加载时地址|-->|运行时地址
---|:--:|:--:|---:
RAM|||
RAM|||ZI段
RAM|||RW段
ROM|RW段||
ROM|RO段||RO段
# ARM 映像文件入口点
入口点分为两类：
**初始入口点**
映像文件运行时入口地点， 称为初始入口点 （Intial Entry Point），加载文件后跳转到的入口。
**普通入口点**
汇编程序中 ENTRY 伪操作定义的， 通常用于标志异常处理程序入口，这样连接器删除无用段时，不会把该段代码删除。


初始入口点必须位于映像的运行时域（因为运行时才会跳转到）；

初始入口点所在运行时域的的加载地址和运行时地址相同（固定域， Root Region）

以上映像文件映射信息，在 keil 中通过 scatter 配置文件进行描述。 Scatter File 用于armlink， Linker Script File 用于 GNU LD 它们的功效是一样的，即告诉Linker用一定的memory layout来生成最后的image。
# Scatter File
Scatter file 是一个文本文件，描述连接器（armlink）生成映像文件时需要的信息（加载时域和运行时域 -- 存储时角度和运行时角度看待数据分布 ）（ 连接器会在连接的时候加入加载时候的代码段，完成程序加载工作）,
如下一个文件的组织情况 ：
![](/images/blog/00018.png)
# 例子
一个加载时域
映像文件保存在 0x10000 地址， 运行的时候， 从该地值读取数据加载到指定区域（运行时域没有指定， 同加载时域，并按配置文件分类存放）
```
LR_1 0x010000        ; 加载时域LR_1, 也就是加载的时候，从这个地址读取数据
{
    ER_RO +0 0x400       ; 第一个运行时域， 没有指定其实地址，所以为 0x010000， 大小限制在0x400
    {
        * (+RO)      ; 所有 R0 文件放置在这个区域  (这就是一个输入段)
    }
    ER_RW 0x040000   ; 第二个运行时与，指定地址0x040000， 放置所有RW数据
    {
        * (+RW) 
    }
    ER_ZI +0         ; 最后一个运行时域，放置 ZI
                     ; 地址是上一个地址结束的 0 偏移， 也就是紧跟其后
    {
        * (+ZI)   
    }
}
```
二个加载时域
```
LR_1 0x010000     ; The first load region is at 0x010000.
{    
	ER_RO +0      ; The address is 0x010000.    
	{        
		* (+RO)    
	}
}
LR_2 0x040000     ; The second load region is at 0x040000.
{    
	ER_RW +0      ; The address is 0x040000.    
	{        
		* (+RW)   ; All RW sections are placed consecutively into this region.    
	}    
	ER_ZI +0      ; The address is 0x040000 + size of ER_RW region.    
	{        
		* (+ZI)   ; All ZI sections are placed consecutively into this region    .    
	}
}
```