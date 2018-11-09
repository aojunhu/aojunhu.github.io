---
layout:     post
title:      KEIL 创建静态链接库+ 调用自己创建的静态链接库
subtitle:   KEIL 创建静态链接库+ 调用自己创建的静态链接库
date:       2018-11-08
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- arm
- boot
---

# 1. 为什么要创建静态链接库？

当公司或者个人需要提供自己的编写的代码供他人调用时，而你或提供方并不想提供源代码（.c源代码），只想提供头文件.h（在头文件中申明函数）供他人调用，他人在调用时只需要知道调用的函数功能是什么，传递的参数是什么等，无需了解函数是怎样具体实现的。 此时就需要静态链接库（当然还有动态链接库，这里先讲解静态链接库的生成和使用）。

# 2. 如何在KEIL 中生成静态链接库.lib 文件
+ 准备生成的静态链接库的源文件：
```
/*
 * file: calc.h
 */
#ifndef __CALC_H__
#define __CALC_H__

void hello(void);
int add(int a, int b);
int sub(int a, int b);
int mul(int a, int b);

#endif

/*
 * file: calc.c
 */
 #include <stdio.h>
#include "calc.h"
#include "utils/debug/art_assert.h"
#include "utils/debug/log.h"

void hello(void)
{
	LOG(LOG_LVL_INFO, "Hello World!\r\n");
}

int add(int a, int b)
{
	return (a + b);
}

int sub(int a, int b)
{
	return (a - b);
}
int mul(int a, int b)
{
	return (a * b);
}
```
+ 打开KEIL ，创建项目，添加源文件 calc.h, calc.c
![](/images/blog/00019.png)

+ 更改KEIL 设置
options for  Target   -->Output  选中 Create Library ，如下图：
![](/images/blog/00020.png)
options for  Target   -->Linker  选择Scatter File，如下图：
![](/images/blog/00021.png)

+ 设置完成后，编译，生成 xxx.lib。
+ 将生成的xxx.lib文件添加到其他的项目中，编译使用。