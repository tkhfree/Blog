---
title: python概述
tags:
  - Python
comments: true
toc: true
mathjax: true
abbrlink: 674cc4e6
date: 2018-11-18 00:21:52
urlname:
categories:
thumbnail:
---

 - python是一个解释型，交互式，无编译的面向对象语言 python缺点：
   
    1. 运行慢，因为python是一个交互式面向对象语言，执行python语句时，每执行一句就要编译一句。c语言是编译好cpu可以执行的文件之后再运行。但是由于网络的延迟，这一缺点影响很小
    2.加密性。解释型语言只能发布源码，而编译型语言可以发布编译后的文件（xxx.exe）。
   
 - python中字符串用单引号'和双引号"都可以，但不可以混用

   


 - python是一个动态语言，变量本身类型不固定

   
   
 - str是不变对象，而list是可变对象。对于可变对象，比如list，对list进行操作，list内部的内容是会变化的。元组(tuple)、数值型（number)、字符串(string)均为不可变对象,而字典型(dictionary)和列表型(list)的对象是可变对象

###list&tuple
list是一个可变数组，tuple是一个不可变数组。（指向永不变），tuple("a","b",["x","y"])，x，y可变，a，b不可变。
### dict&set
python字典是dict函数，其他语言也有称为map，使用键-值（key-value）存储。set跟dict类似，是一组不能改变的key集合，要创建一个set，需要提供一个list作为输入集合
### range
python range()函数生成整数序列，list函数生成list，例如list(range(5))
###函数

 - 定义函数时，需要确定函数名和参数个数；
   
 - 如果有必要，可以先对参数的数据类型做检查；

   


 - 函数体内部可以用return随时返回函数结果；

   


 - 函数执行完毕也没有return语句时，自动return None。

  

 - 函数可以同时返回多个值，但其实就是一个tuple。
 - 函数可以传递默认参数，放在可变参数后面，默认参数必须指向不变对象，不然每次调用一次默认参数的函数就会在指向对象上发生改变
 - 函数可以定义可变参数，定义可变参数传入需是list或者tuple。或者定义函数时可变参数加上*。这样调用时可以输入任意数量参数，或者加上*的list/tuple
 - 关键字参数。可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。

    def person(name, age, **kw):
        print('name:', name, 'age:', age, 'other:', kw)
 函数person除了必选参数name和age外，还接受关键字参数kw。在调用该函数时，可以只传入必选参数或者包含任意个数的关键字参数：

     person('Bob', 35, city='Beijing')
    name: Bob age: 35 other: {'city': 'Beijing'}

 - 返回函数中内部函数可以调用外部函数的参数和局部变量，当调用外层函数输出内部函数的时候，这个参数和局部变量都保存起来了，返回的函数其实并没有被调用，只有调用返回赋值的变量的时候才被调用

在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。

    def f1(a, b, c=0, *args, **kw):
        print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)
    
    def f2(a, b, c=0, *, d, **kw):
        print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
### 装饰器
[详解Python的装饰器](https://www.cnblogs.com/cicaday/p/python-decorator.html)
在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator），本质上，decorator就是一个返回函数的高阶函数