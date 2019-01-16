# 使用Python实现自动化测试


Author: naiquan.hu
Date: 2019-01-16

---

# 1 环境搭建

## 1.1 安装Python及开发工具
官网下载Python和pycharm。

## 1.2 pycharm使用
**创建一个简单的Python工程**
![](\images\programming\python\auto-test-using-python-001.png)

**添加源代码**
File --> Settings --> Project xxx --> Project structure --> New Folder
![](\images\programming\python\auto-test-using-python-002.png)
![](\images\programming\python\auto-test-using-python-003.png)
![](\images\programming\python\auto-test-using-python-004.png)
右键src--> 
![](\images\programming\python\auto-test-using-python-005.png)
![](\images\programming\python\auto-test-using-python-006.png)
```
# encoding: utf-8

class Calculator:
    def add(self, x, y):
        return x + y

    def subtract(self, x, y):
        return x - y

    def multiply(self, x, y):
        return x * y

    def divide(self, x, y):
        return x / y

print Calculator().multiply(3, 6)
```

之后运行。

