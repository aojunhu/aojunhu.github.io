---
layout:     post
title:      设计模式——C语言
subtitle:   设计模式——C语言
date:       2018-11-06
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
- C
---

# 1.单例模式

```
typedef struct _DATA
{
    void* pData;
}DATA;

void* get_data()
{
    static DATA* pData = NULL;

    if(NULL != pData)
        return pData;

    pData = (DATA*)malloc(sizeof(DATA));
    assert(NULL != pData);
    return (void*)pData;
}
```
# 2.原型模式
原型模式本质上说就是对当前数据进行复制。就像变戏法一样，一个鸽子变成了两个鸽子，两个鸽子变成了三个鸽子，就这么一直变下去。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

typedef struct _Person {
    const char *name;
    int age ;
    struct _Person* (*Person_Copy)(struct _Person* p) ;
}Person;

struct _Person* Person_Copy(struct _Person * pData)
{
    struct _Person* p = (struct _Person*)malloc(sizeof(struct _Person));
    assert(NULL != p);
    memmove(p, pData, sizeof(struct _Person));
    return p;
};

struct _Person* mclone(struct _Person* pData){

    return  pData->Person_Copy(pData) ;
}

int main(){
    const char *mname = "zhangsan" ;
    Person p1 = {
                 .name = mname  ,
                 .age = 18,
                 .Person_Copy = Person_Copy,
    } ;
    printf("name of p1 is %s, age is %d\n" , p1.name , p1.age) ;
    Person* p2 = mclone(&p1) ;
    printf("name of p2 is %s, age is %d\n" , p2->name , p2->age) ;
    free(p2) ;
    return 0 ;
}
```
# 2.组合模式
组合模式的定义： 将对象组合成树形结构以表示‘部分-整体’的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。



待添加

# 3.模板模式
template主要是一种流程上的统一，细节实现上的分离。