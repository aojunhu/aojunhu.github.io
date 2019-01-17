---
layout:     post
title:      使用Python实现自动化测试
subtitle:   使用Python实现自动化测试
date:       2019-01-16
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Python
---

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
File --> Settings --> Project xxx --> Project structure --> New Folder --> src
![](\images\programming\python\auto-test-using-python-002.png)
![](\images\programming\python\auto-test-using-python-003.png)
![](\images\programming\python\auto-test-using-python-004.png)
右键src--> New --> Python Package --> "code_pkg"   
右键code_pkg --> New --> Python File --> "calculator"
![](\images\programming\python\auto-test-using-python-005.png)
![](\images\programming\python\auto-test-using-python-006.png)
calculator.py文件内容如下：
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

```

# 2 使用unittest框架编写测试用例
可以直接使用Python自带的单元测试框架unittest来编写自动化测试用例，利用其组织测试用例，断言预期结果，以及批量执行测试用例等功能。
目录结构如下组织：
![](\images\programming\python\auto-test-using-python-006.png)

# 2.1 创建测试用例
右键src--> New --> Python Package --> "test_case"   
选中 class Calculator， 右键弹出菜单
![](\images\programming\python\auto-test-using-python-007.png)
![](\images\programming\python\auto-test-using-python-008.png)
![](\images\programming\python\auto-test-using-python-009.png)

test_calculator.py 内容如下：
```
import unittest

from src.code_pkg import calculator

class TestCalculator(unittest.TestCase):
    def setUp(self):
        self.calc = calculator.Calculator()
        print "TestCalculator.setUp"

    def test_add(self):
        print "TestCalculator.test_add"
        self.assertEqual(self.calc.add(2, 3), 5)

    def test_subtract(self):
        print "TestCalculator.test_subtract"
        self.assertEqual(self.calc.subtract(4, 3), 1)

    def test_multiply(self):
        print "TestCalculator.test_multiply"
        self.assertEqual(self.calc.multiply(4, 3), 12)

    def test_divide(self):
        print "TestCalculator.test_divide"
        self.assertEqual(self.calc.divide(8, 2), 4)

    def tearDown(self):
        print "TestCalculator.tearDown"

if __name__ == "__main__":
    unittest.main()
```

## 2.2 使用TestSuite来组装多个测试用例
右键src--> New --> Python File --> "runtest"   
runtest.py文件内容如下：
```
# coding=utf-8

import unittest
from src.test_case import test_calculator

#构造测试集
suite = unittest.TestSuite()
suite.addTest(test_calculator.TestCalculator('test_add')) #执行测试方法test_add

if __name__=='__main__':
    #执行测试
    runner = unittest.TextTestRunner()
    runner.run(suite)
```
执行测试，结果如下：
```
TestCalculator.setUp
TestCalculator.test_add
TestCalculator.tearDown
TestCalculator.setUp
TestCalculator.test_divide
TestCalculator.tearDown
TestCalculator.setUp
TestCalculator.test_multiply
TestCalculator.tearDown
TestCalculator.setUp
TestCalculator.test_subtract
TestCalculator.tearDown


Ran 4 tests in 0.007s

OK

Process finished with exit code 0

```