---
layout:     post
title:      递归遍历目录下的所有文件，并替换文件的内容
subtitle:    递归遍历目录下的所有文件，并替换文件的内容
date:       2018-11-09
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Pthon
---

# 递归遍历目录下的所有文件，并替换文件的内容

```
#!/bin/env python                                                       # -*- coding:utf-8 -*-
 
import sys 
import os

def replace(file_path, old_str, new_str):
    try:
        f = open(file_path,'r+')
        all_lines = f.readlines()
        f.seek(0)
        f.truncate()
        for line in all_lines:
            line = line.replace(old_str, new_str)
            f.write(line)
        f.close()
    except Exception,e:
        print e

def list_all_files(rootdir):
    _files = []
    list = os.listdir(rootdir)
    for i in range(0, len(list)):
        path = os.path.join(rootdir, list[i])
        if os.path.isdir(path):
            _files.extend(list_all_files(path))
        if os.path.isfile(path):
            _files.append(path)
    return _files
    
if __name__ == "__main__":
    if len(sys.argv) < 2:
        print "need 1 params"
        sys.exit(1)
    rootdir = sys.argv[1]
    _fs = list_all_files(rootdir)
    for f in _fs:
        replace(f, "gmacro.h", "wifi_config.h") 
```

# Linux下批量把Dos格式的文件转换为UNIX格式
```
#!/bin/sh

showUsage()
{
    echo "Usage: dos2unixdir [destdir]"
    exit 0
}

dir=$PWD

if [ $# -gt 1 ] 
then
    showUsage
fi

if [ $# -gt 0 ] 
then
    dir=$1
fi

echo $dir
for i in `find $dir`
do
    if [ ! -d $i ]
    then
        echo "processing file..." $i
        dos2unix $i $i
    fi  
done           
```
# linux 批量设置文件夹755 文件644权限
```
find -type d|xargs chmod 755
find -type f|xargs chmod 644
```