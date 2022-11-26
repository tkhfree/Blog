---
title: Python总结一
toc: true
comments: true
tags:
  - Python
abbrlink: d1eb1471
date: 2020-06-29 09:00:45
thumbnail:
urlname:
categories:
---

## 数字类型

### 整形

int long

### 浮点型

float

### 复数

In [11]:

```
a = 1
type(a)
b=1.0
type(b)
c=1111111111111111111111111111111
type(c)
```

Out[11]:

```
int
```

## 表达式

不支持i++，使用i+=1

In [15]:

```
a=6+8
print (a)
14
```

## 字符串和列表

字符串就是特殊形式的列表，但是字符串的值是常量，对于字符串的更改都是更改的其副本，列表是变量，对于列表更改内容，列表地址不变。

字符串养成习惯用双引号来表示

python没有数组的概念，用列表表示，列表中可以存放任意类型的数据，同一列表可以放置多种数据

In [55]:

```
# 取下标操作
str_test="hello"
list_test=["h","e","l","l","o"]
print (str_test[0])
print (list_test[0])
# 取1个到第3个之间的元素
print (str_test[1:4])
print (list_test[0:4])
# 取第1个到5个之间的元素，每隔2个取1个
print (str_test[0:5:2])
print (list_test[0:5:2])
# 取某个元素的下标
print (str_test.index("e"))
print (list_test.index("o"))
# 逆序取得某个元素
print (str_test[-1])
print (list_test[-1])
# 追加元素
print (str_test+",world")
print (list_test+[",","world"])
# 计算长度
print (len(str_test))
print (len(list_test))
# 多次复制
print(str_test*3)
print(list_test*4)
# 复制新变量
str_new_test = str_test[:]
list_new_test = list_test[:]
print(str_new_test)
print(list_new_test)
# python中变量名有点像c中的指针
str_new_test1 = str_new_test
print(id(str_new_test))
print(id(str_new_test1))
print(id(str_test))

# 列表变量可以更改元素
list_test[1]="i"
print(list_test)
# 列表追加
list_test.append("world")
print(list_test)

print(list_test+["python"])
# 列表在第1位插入
list_test[1:1]=[1234567]
print(list_test)

# 字符串的大小写转变
str_test_upper = str_test.upper()
print(str_test_upper)
h
h
ell
['h', 'e', 'l', 'l']
hlo
['h', 'l', 'o']
1
4
o
o
hello,world
['h', 'e', 'l', 'l', 'o', ',', 'world']
5
5
hellohellohello
['h', 'e', 'l', 'l', 'o', 'h', 'e', 'l', 'l', 'o', 'h', 'e', 'l', 'l', 'o', 'h', 'e', 'l', 'l', 'o']
hello
['h', 'e', 'l', 'l', 'o']
140384645081328
140384645081328
140384645081328
['h', 'i', 'l', 'l', 'o']
['h', 'i', 'l', 'l', 'o', 'world']
['h', 'i', 'l', 'l', 'o', 'world', 'python']
['h', 1234567, 'i', 'l', 'l', 'o', 'world']
HELLO
```

## 列表

In [*]:

```
# 几种构建列表方式
list_test1=list("helloworld")
list_test2=["h","e","l","l","o"]
list_test3=[x for x in range(10)]
list_test4=map(lambda x:x ,[1,2,3,4,5])
print(list_test4)
print(list_test1)
```

## **lambda**

构造匿名函数

result = lambda x : x+1

等同于

def result(x): return x+1

## map

map(function,sequenec[,sequence, ...]) -->list

map接受两个参数，一个函数，一个可迭代的对象或者是序列，返回一个列表

## 元组

元组就是常量化的列表

tuple_test=(1,2,3,4,5)

元组表达跟列表基本一致，只是元组里的元素不能修改，列表的操作（除了修改）元组也都可以操作

列表转为元组

tup_test=tuple([1,2,3,4,5])

**什么时候用元组？当希望把一个序列作为参数传递时，不希望被修改**

## 字典

与java中map对应，字典也是一个键值对结构

dict_test={"aa": 1, "bb": 2 , 1:aa , 1.3:bb}

取值

dict_test["aa"] dict_test[2]

dict_test[1.3]

新添加键值对

dict_test["test"]="helloworld"

更新键值对

dict_test["aa"]="python"

## 集合

集合和列表相似，但是集合不能存放相同的元素

set_test=set([[1,2,3,4,5,1,2,3,4,5]])    -->[1,2,3,4,5]

## 异常处理

异常处理基本同java 类似，在try点创建一个标记，当遇到异常时匹配except，抛出异常，如果没有匹配的异常，则向上传递。

```python
try:
  exec1
  try:
    exec2
  fianlly:
    exec3
except ErrorType:
  catch error
```

## 函数

```pyhton
def func(name, age, cls="", *arg, **karg):
```

函数用def定义，func是函数名，name，age是普通参数，必须按位置传入，cls是关键字参数或者默认参数，缺省可选，karg也是关键字参数，和cls关键字参数不一样的是，在函数中存在，但是不知道具体名称的关键字参数，arg是位置参数，同样是在函数中存在，但是不知道具体名称的位置参数。

参数的位置一定是按照（位置参数，关键字参数，*参数，**参数）的顺序来写，其中位置参数如果设定就一定要传入，如果参数定义为：

```python
def func(*arg, **karg):
  state
```

则表示函数可以传入任意类型和任意数量的参数

传参形式是有多种多样的。

```python
def func(name, age=10):
  print ("name is %s, age is %d" %(name,age))
#调用方式
func("hello")
func("hello", 20)
func(name="hello", age=20)
func(age=20, name="hello")

param_list=["hello",20]
func(*param_list)

param_dict={"name":"hello", "age":20}
func(**param_dict)
```

## 迭代器

迭代器iterator需要实现两个方法：

1. \__iter__，返回迭代器自身
2. \__next__，返回迭代器下一个元素

```python
class myiter(object):
    def __init__(self, count):
        self.count = count
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if (0 >= count):
            raise StopIteration()
        else:
            self.count -= 1
            return self.count

myiter it = myiter(4)
while 1:
    e = it.__next__()
    print e
```

## 生成器

迭代器+协程

用open函数打开文件时，最好使用生成器模式。

创建生成器的方式

```python
# 使用yield函数,每次执行都生成一个hello，获取它的值需要配合使用next函数或者迭代获取
def first_generator():
    yield "hello"

result = first_generator()
result.next()
# 使用()获得生成器,当（）配合def使用是方法或者函数，当（）配合“,”使用是构造元组tumple，当和列表推导一起使用是生成器
second_generator = (x for x in range(10))
```

def定义的时一个普通函数，但是里面出现yield这个关键词，这就不是普通函数了，变成生成器了，有next这个成员函数（python3里变成\__next__），**生成器=迭代器+协程**

yield两重魔力，第一重是迭代器有\_\_next\_\_函数，第二重当执行到yield时，相当于return object，但是没有并不完全一样，因为return会退出函数，而yield不会，相当于保存了运行的上下文，把函数挂起来。既然把函数挂起来，相当于该协程让出程序执行权，当\__next\_\_再次被调用，就从当前被挂起的上下文继续执行，这样可以一直循环下去，直到执行不到yield会抛出Raise StopIteration异常。

可以使用生成器实现cat file | grep keyword这样的Linux命令

```python
#!/usr/bin/python

with open("filepath","r+") as file_obj:
  	for file in file_obj:
      	if “keyword” in file.strip():
          	print file
        else:
          	pass
```

[迭代器和生成器的大白话解释](https://segmentfault.com/a/1190000007208388)

## argv和raw_input

argv和raw_input都可以从命令行输入参数，不同点在于argv是从开始运行py文件时，就将输入以参数的形式在程序里使用；raw_input是在程序执行时，根据提示输入数据。

## \_\_name\_\_ ==\_\_main\_\_

\_\_name\_\_是内置变量，表示的是当前模块的名字。当前python文件是程序的主入口时，当前模块的\_\_name\_\_就是\_\_main\_\_，如果当前python文件不是程序的主入口，python是以模块import进来的，则\_\_name\_\_表示当前包的层次，是导入的文件名。

python的\-m参数用于将一个模块或者包作为一个脚本运行，而\_\_main\_\_.py文件相当于是一个包的入口程序。

`python xxx.py` 与 `python -m xxx.py` 的区别，一种是直接运行，一种是当做模块来运行。

```python
#!/user/bin/python
#coding=utf-8
#run.py
import sys
print __name__
print sys.path
```

两种运行方式

```shell
python run.py
__main__    #__name__的值
['/yh1', '/usr/lib/python27.zip',...]


python -m run.py
run   #__name__的值
['', '/usr/lib/python27.zip', ...]
/usr/bin/python: No module named run.py
```

## python的简洁if语句

1. 普通写法

```python
a, b, c = 1, 2, 3
if a > b :
    c =a 
else:
    c = b 
```

2. 一行表达式，为真时放if前，也就是if前的表达式是if为true的值

```python
c = a if a > b else b;
```

3. 二维列表，利用大小判断的0，1当索引

```python
c = [b, a][a>b]
```

