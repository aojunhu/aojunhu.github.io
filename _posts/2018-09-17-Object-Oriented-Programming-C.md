---
layout:     post
title:      面向对象编程——C语言
subtitle:   面向对象编程——C语言
date:       2018-09-17
author:     naiquan.hu
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
    - C
    - 面向对象
---

# 1. 抽象数据类型
集合中可以存放任意数据类型，提供一些方法来操作它。因此它里面存放的应该是抽象数据类型。

```
set.h

#ifndef SET_H
#define SET_H
extern const void * Set;
void * add (void * set, const void * element);
void * find (const void * set, const void * element);
void * drop (void * set, const void * element);
int contains (const void * set, const void * element);
#endif
```

```
object.h

#ifndef OBJECT_H
#define OBJECT_H
extern const void * Object; /* new(Object); */
int differ (const void * a, const void * b);
#endif
```

```
new.h

#ifndef NEW_H
#define NEW_H
void * new (const void * type, ...);
void delete (void * item);
#endif
```
上面是提供给外界的可见接口，无需改变，当不同的实现只需改变下面的实现部分即可。

```
impl.h

#ifndef IMPL_H
#define IMPL_H
struct Set
{
    unsigned count; //count reconrds the number of elements in a set
};
struct Object
{
    unsigned count; //count records how many times this element has been added to the set
    struct Set * in;
};
#endif
```

方法的具体实现
```
impl.c

#include <assert.h>
#include <stdlib.h>
#include "new.h"
#include "set.h"
#include "object.h"
#include "impl.h"

void * new (const void * type, ...)
{
    const size_t size = * (const size_t *) type;
    void * p = calloc(1, size);
    assert(p);
    return p;
}
void delete (void * _item)
{
    struct Object * item = _item;
    if (item)
    {
        free(item);
    }
}

void * add (void * _set, void * _element)
{
    struct Set * set = _set;
    struct Object * element = (void *) _element;
    assert(set);
    assert(element);

    if (! element->in)
        element->in = set;
    else
        assert(element->in == set);
    ++element->count, ++set->count;
    return (void *) element;
}

void * find (const void * _set, const void * _element)
{
    const struct Set * set = _set;
    const struct Object * element = _element;
    assert(set);
    assert(element);
    return element->in == set ? (void *) element : 0;
}

int contains (const void * _set, const void * _element)
{
    return find(_set, _element) != 0;
}

void * drop (void * _set, const void * _element)
{
    struct Set * set = _set;
    struct Object * element = find(_set, _element);
    if (element)
    {
        if(--element->count == 0)
            element->in = 0;
        --set->count;
    }
    return element;
}
unsigned count (const void * _set)
{
    const struct Set * set = _set;
    assert(set);
    return set->count;
}
int differ (const void * a, const void * b)
{
    return a != b;
}
```

```
main.c

#include <stdio.h>
#include "new.h"
#include "object.h"
#include "set.h"
#include "impl.h"


static size_t _Set = sizeof(struct Set);
static size_t _Object = sizeof(struct Object);
static void * pSet = & _Set;
static void * pObject = & _Object;
int main ()
{
    void * s = new(pSet);
    void * a = add(s, new(pObject));
    void * b = add(s, new(pObject));
    void * c = new(pObject);
    if (contains(s, a) && contains(s, b))
        puts("ok");
    if (contains(s, c))
        puts("contains?");
    if (differ(a, add(s, a)))
        puts("differ?");
    if (contains(s, drop(s, a)))
        puts("drop?");
    delete(drop(s, b));
    delete(drop(s, c));
    return 0;
}
```

# 2. 类的实现
类的创建，销毁，克隆

```
class.h

#ifndef CLASS_H
#define CLASS_H
#include <stddef.h>
typedef struct _Class {
    size_t size;
    void * (* ctor) (void * self, va_list * app);
    void * (* dtor) (void * self);
    void * (* clone) (const void * self);
    int (* differ) (const void * self, const void * b);
}Class;
#endif
```

```
string.h

#ifndef STRING_H
#define STRING_H
#include "class.h"
typedef struct _String {
    const void * Class; /* must be first */
    char * text;
}String;
extern const void * pString;
void * String_ctor (void * _self, va_list * app);
void * String_dtor (void * _self);
void * String_clone (const void * _self);
int String_differ (const void * _self, const void * _b);
#endif
```

string的实现
```
string.c

#include <assert.h>
#include <stdlib.h>
#include <stdarg.h>
#include <string.h>
#include "string.h"
#include "new.h"

void * String_ctor (void * _self, va_list * app)
{
    String * self = _self;
    const char * text = va_arg(* app, const char *);
    self->text = malloc(strlen(text) + 1);
    assert(self->text);
    strcpy(self->text, text);
    return self;
}
void * String_dtor (void * _self)
{
    String * self = _self;
    free(self->text), self->text = 0;
    return self;
}
void * String_clone (const void * _self)
{
    const String * self = _self;
    return new(pString, self->text);
}
int String_differ (const void * _self, const void * _b)
{
    const String * self = _self;
    const String * b = _b;
    if (self == b)
        return 0;
    if (! b || b->Class != pString)
        return 1;
    return strcmp(self->text, b->text);
}
```

```
new.h

#ifndef NEW_H
#define NEW_H
void * new (const void * type, ...);
void delete (void * self);
void * clone (const void * self);
int differ (const void * self, const void * b);
size_t sizeOf (const void * self);
#endif
```

new的实现

```
new.c

#include <assert.h>
#include <stdlib.h>
#include <stdarg.h>
#include "class.h"
#include "new.h"

void * new (const void * _class, ...)
{
    const Class *class = _class;
    void *p = calloc(1, class->size);
    assert(p);
    *(const Class **) p = class;
    if(class->ctor)
    {
        va_list ap;
        va_start(ap, _class);
        p = class->ctor(p, &ap);
        va_end(ap);
    }
    return p;
}
void delete (void * self)
{
    const Class ** cp = self;
    if (self && *cp && (*cp)->dtor)
        self = (*cp)->dtor(self);
    free(self);
}
int differ (const void * self, const void * b)
{
    const Class * const * cp = self;
    assert(self && *cp && (*cp)->differ);
    return (*cp)->differ(self, b);
}
void * clone (const void * self)
{
    const Class * const * cp = self;
    assert(self && *cp && (*cp)->clone);
    return (*cp)->clone(self);
}
size_t sizeOf (const void * self)
{
    const Class * const * cp = self;
    assert(self && * cp);
    return (*cp)->size;
}
```

```
main.c

#include <stdio.h>
#include <stdarg.h>
#include <assert.h>
#include "string.h"
#include "new.h"

static const Class _String = {
    sizeof(String),
    String_ctor, String_dtor,
    String_clone, String_differ
};
const void * pString = &_String;
int main ()
{
    void * a = new(pString, "a"), *aa = clone(a);
    void * b = new(pString, "b");
    if (differ(a, b))
        puts("ok");
    if (differ(a, aa))
        puts("differ?");
    if (a == aa)
        puts("clone?");
    delete(a), delete(aa), delete(b);
    return 0;
}
```
