---
layout:     post
title:      C语言pragma pack()用法详解
subtitle:   C语言pragma pack()用法详解
date:       2019-03-09
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- C
---


# C语言pragma pack()用法详解



# 1. 什么是对齐？为什么要对齐？
计算机中内存空间都是按照byte划分的，从理论上讲似乎对任何类型的变量的访问可以从任何地址开始，但实际情况是在访问特定变量的时候经常在特定的内存地址访问，这就需要各类型数据按照一定的规则在空间上排列，而不是顺序的一个接一个的排放，这就是对齐。

各个硬件平台对存储空间的处理上有很大的不同。一些平台对某些特定类型的数据只能从某些特定地址开始存取。其他平台可能没有这种情况，但是最常见的是如果不按照适合其平台要求对数据存放进行对齐，会在存取效率上带来损失。比如有些平台每次读都是从偶地址开始，如果一个int型（假设为32位系统）如果存放在偶地址开始的地方，那么一个读周期就可以读出，而如果存放在奇地址开始的地方，就可能会需要2个读周期，并对两次读出的结果的高低字节进行拼凑才能得到该int数据。显然在读取效率上下降很多。这也是空间和时间的博弈。

# 2. pragma pack语法
用sizeof运算符求算某结构体所占空间时，并不是简单地将结构体中所有元素各自占的空间相加，这里涉及到内存字节对齐的问题，有时候为了内存对齐需要补齐空字节。通常写程序的时候，不需要考虑对齐问题。编译器会替我们选择适合目标平台的对齐策略。当然，我们也可以通知给编译器传递预编译指令而改变对指定数据的对齐方法。
语法：**#pragma pack( [show] | [push | pop] [, identifier], n )**
作用：指定结构，联合和类的包对齐方式（pack alignment）
show(optical):
显示当前packing aligment的字节数，以warning message的形式显示。
push(optical):
Pushes the current packing alignment value on the internal compiler stack, and sets the current packing alignment value to n. If n is not specified, the current packing alignment value is pushed.
pop(optical):
Removes the record from the top of the internal compiler stack. If n is not specified with pop, then the packing value associated with the resulting record on the top of the stack is the new packing alignment value. If n is specified, for example, #pragma pack(pop, 16), n becomes the new packing alignment value. If you pop with identifier, for example, #pragma pack(pop, r1), then all records on the stack are popped until the record that hasidentifier is found. That record is popped and the packing value associated with the resulting record on the top of is the stack the new packing alignment value. If you pop with an identifier that is not found in any record on the stack, then the pop is ignored.
identifier(optional):
When used with push, assigns a name to the record on the internal compiler stack. When used with pop, pops records off the internal stack until identifieris removed; if identifier is not found on the internal stack, nothing is popped.
n (optional):
Specifies the value, in bytes, to be used for packing. If the compiler option /Zp is not set for the module, the default value for n is 8. Valid values are 1, 2, 4, 8, and 16. The alignment of a member will be on a boundary that is either a multiple of n or a multiple of the size of the member, whichever is smaller.

# 3.结构体对齐规则

结构体中各个成员按照它们被声明的顺序在内存中顺序存储。
1. 将结构体内所有数据成员的长度值相加，记为sum_a； 
2. 将各数据成员内存对齐，按各自对齐模数而填充的字节数累加到和sum_a上，记为sum_b。对齐模数是【该数据成员所占内存】与【#pragma pack指定的数值】中的较小者。
3. 将和sum_b向结构体模数对齐，该模数是【#pragma pack指定的数值】、【未指定#pragma pack时，系统默认的对齐模数8字节】和【结构体内部最大的基本数据类型成员】长度中数值较小者。结构体的长度应该是该模数的整数倍。


## 3.1 基本数据类型所占内存大小



|             | char | bool | short | int  | unsigned int | float | long | double | long long | long double | 指针 |
| ----------- | ---- | ---- | ----- | ---- | ------------ | ----- | ---- | ------ | --------- | ----------- | ---- |
| 32bit编译器 | 1    | 1    | 2     | 4    | 4            | 4     | 4    | 8      | 8         | 8           | 4    |
| 64bit编译器 | 1    | 1    | 2     | 4    | 4            | 4     | 8    | 8      | 8         | 8           | 8    |


以下例子均按32bit编译器处理。


## 3.2 Test

```
#pragma pack(4)  
typedef struct{  
    char c;  
    short sh;  
    int a;  
    float f;  
    int *p;  
    char *s;  
    double d;  
}test_t; 
#pragma pack()
```
总共占28Bytes。 c的偏移量为0，占1个Byte。sh占2个Byte，它的对齐模数是2（2<4，取小者），存放起始地址应该是2的整数倍，因此c后填充1个空字符，sh的起始地址是2。a占4个Byte，对齐模数是4，因此接在sh后存放即可，偏移量为4。f占4个字节，对齐模数是4，存放地址是4的整数倍，起始地址是8。p,s的起始地址分别是12,16。d占8个字节，对齐模数是4（4<8)，d从偏移地址为20处存放。存放后结构体占28个字节，是4的整数倍不用补空字符。


| 0 | 1 | 2 | 3 |
| ---- | ---- | ---- | ---- |
| c | null | sh | sh|
| a | a | a | a |
| f | f | f | f |
| p | p | p | p |
| s | s | s | s |
| d | d | d | d |
| d | d | d | d |


调整成1byte对齐：

```
#pragma pack(1)  
typedef struct{  
    char c;  
    short sh;  
    int a;  
    float f;  
    int *p;  
    char *s;  
    double d;  
}test_t; 
#pragma pack()
```

总共占27Bytes。 c的偏移量为0，占1个Byte。sh占2个Byte，它的对齐模数是1（1<2，取小者），存放起始地址应该是1的整数倍，因此sh的起始地址是1。a占4个Byte，对齐模数是1，因此接在sh后存放即可，偏移量为3。f占4个字节，对齐模数是1，存放地址是1的整数倍，起始地址是7。p,s的起始地址分别是11,15。d占8个字节，对齐模数是1，d从偏移地址为19处存放。存放后结构体占27个字节。


| 0 | 1 | 2 | 3 |
| ---- | ---- | ---- | ---- |
| c | sh | sh | a |
| a | a | a | f |
| f | f | f | p |
| p | p | p | s |
| s | s | s | d |
| d | d | d | d |
| d | d | d | null |


```
#pragma pack(4)  
typedef struct{  
    char ch;  
    short sh;  
    int a;  
    float f;
    int *p;
    char *s;
    char arry[5];
    double d;  
}test_t;
#pragma pack()
```

结果如下：


| 0 | 1 | 2 | 3 |
| ---- | ---- | ---- | ---- |
| c | null | sh | sh |
| a | a | a | a |
| f | f | f | f |
| p | p | p | p |
| s | s | s | s |
| arry[0] | arry[1] | arry[2] | arry[3] |
| arry[4] | null | null | null |
| d | d | d | d |
| d | d | d | d |