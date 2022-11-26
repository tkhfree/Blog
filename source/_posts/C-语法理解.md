---
title: C++语法理解
tags:
  - C++
  - C
comments: true
toc: true
mathjax: true
abbrlink: 1dbcd36e
date: 2020-07-03 14:36:08
urlname:
categories:
thumbnail:
---

### const用法

#### 修饰函数

在普通的非 const成员函数中，this的类型是一个指向类类型的 const指针。可以改变this所指向的值，但不能改变 this所保存的地址。
在 const成员函数中，this的类型是一个指向 const类类型对象的 const指针。既不能改变 this所指向的对象，也不能改变 this所保存的地址。

#### 修饰函数的参数

如果参数作输出用，不论它是什么数据类型，也不论它采用“指针传递”还是“引用传递”，都不能加const修饰，否则该参数将失去输出功能。const只能修饰输入参数。

如果输入参数采用“指针传递”，那么加const修饰可以防止意外地改动该指针，起到保护作用。

```c++
例如StringCopy函数：
void StringCopy (char*strDestination, const char *strSource);
其中strSource是输入参数，strDestination是输出参数。给strSource加上const修饰后，如果函数体内的语句试图改动strSource的内容，编译器将指出错误。
```

如果输入参数采用“值传递”，由于函数将自动产生临时变量用于复制该参数，该输入参数本来就无需保护，所以不要加const修饰。

例如不要将函数void Func1(int x) 写成void Func1(const int x)。同理不要将函数void Func2(A a) 写成void Func2(const A a)。其中A为用户自定义的数据类型。

对于非内部数据类型的参数而言，象void Func(A a) 这样声明的函数注定效率比较底。因为函数体内将产生A类型的临时对象用于复制参数a，而临时对象的构造、复制、析构过程都将消耗时间。
为了提高效率，可以将函数声明改为void Func(A &a)，因为“引用传递”仅借用一下参数的别名而已，不需要产生临时对象。但是函数void Func(A &a) 存在一个缺点：
“引用传递”有可能改变参数a，这是我们不期望的。解决这个问题很容易，加const修饰即可，因此函数最终成为voidFunc(const A &a)。
以此类推，是否应将void Func(int x) 改写为voidFunc(const int&x)，以便提高效率？完全没有必要，因为内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。

##### summary

对于非内部数据类型的输入参数，应该将“值传递”的方式改为“const引用传递”，目的是提高效率。例如将voidFunc(A a) 改为voidFunc(const A &a)。
对于内部数据类型的输入参数，不要将“值传递”的方式改为“const引用传递”。否则既达不到提高效率的目的，又降低了函数的可理解性。例如voidFunc(int x) 不应该改为voidFunc(const int &x)。

#### 修饰函数的返回值

如果给以“指针传递”方式的函数返回值加const修饰，那么函数返回值（即指针）的内容不能被修改，该返回值只能被赋给加const修饰的同类型指针。例如函数

const char * GetString(void);
如下语句将出现编译错误：
char *str = GetString();
正确的用法是
const char  *str =GetString();

如果函数返回值采用“值传递方式”，由于函数会把返回值复制到外部临时的存储单元中，加const修饰没有任何价值。
例如不要把函数int GetInt(void) 写成constint GetInt(void)。
同理不要把函数A GetA(void) 写成constA GetA(void)，其中A为用户自定义的数据类型。

如果返回值不是内部数据类型，将函数AGetA(void) 改写为constA &GetA(void)的确能提高效率。但此时千万千万要小心，一定要搞清楚函数究竟是想返回一个对象的“拷贝”还是仅返回“别名”就可以了，否则程序会出错。
函数返回值采用“引用传递”的场合并不多，这种方式一般只出现在类的赋值函数中，目的是为了实现链式表达。

#### const 成员函数

**任何不会修改数据成员的函数都应该声明为const类型**。如果在编写const成员函数时，不慎修改了数据成员，或者调用了其它非const成员函数，编译器将指出错误，这无疑会提高程序的健壮性。以下程序中，类stack的成员函数GetCount仅用于计数，从逻辑上讲GetCount应当为const函数。编译器将指出GetCount函数中的错误。

```cpp
class Stack
{
 public:
    void Push(int elem);
    int Pop(void);
    int GetCount(void) const; // const 成员函数
 private:
    int m_num;
    int m_data[100];
};
int Stack::GetCount(void) const
{
    ++ m_num; // 编译错误，企图修改数据成员m_num
    Pop();// 编译错误，企图调用非const函数
    return    m_num;
}
```

const 成员函数的声明看起来怪怪的：const关键字只能放在函数声明的尾部，大概是因为其它地方都已经被占用了。
关于Const函数的几点规则：

> - const对象只能访问const成员函数,而非const对象可以访问任意的成员函数,包括const成员函数.
> - const对象的成员是不可修改的,然而const对象通过指针维护的对象却是可以修改的.
> - const成员函数不可以修改对象的数据,不管对象是否具有const性质.它在编译时,以是否修改成员数据为依据,进行检查.
> - 然而加上mutable修饰符的数据成员,对于任何情况下通过任何手段都可修改,自然此时的const成员函数是可以修改它的

const放在后面跟前面有区别么
==>
准确的说const是修饰this指向的对象的
譬如，我们定义了

```cpp
classA
{
public:
    f(int);
};
```

这里f函数其实有两个参数，第一个是A*const this, 另一个才是int类型的参数
如果我们不想f函数改变参数的值，可以把函数原型改为f(const int),但如果我们不允许f改变this指向的对象呢？因为this是隐含参数，const没法直接修饰它，就加在函数的后面了，表示this的类型是const A *const this。
const修饰*this是本质，至于说“表示该成员函数不会修改类的数据。否则会编译报错”之类的说法只是一个现象，根源就是因为*this是const类型的

static const修饰：为了共享又不能修改。

[关于const和static的用法](https://cloud.tencent.com/developer/article/1637765)

### Template

#### 类模板

template< class 形参名，class 形参名，......> class 类名 {...};

**template <class T> class A { public:  T a; T b;  T hy(T c, T &d); };**

```cpp
template<class T>
class A{
    void func();
    T test(T& a, T& b);
}

template<class T>
void A<T>::func(){    
}

template<class T>
T A<T>::test(T a, T b){
}
```

#### 函数模板

**函数模板提供了一种函数行为，该函数行为可以表示成多种类型进行调用，也就是说函数模板表时一个函数家族。**

```cpp
template<T>
inline T const& Max(T const& a, T const& b)
```

##### **这是有人会发出疑问？但我们实例化这个函数模板时，如果与非模板函数完全相同，那会不会产生二义性呢？**

```cpp
int main(){
    int a,b;
    Max<int>(a, b);
}
```

##### 对于非模板函数和同名的模板函数，实例化后的模板函数如果与非模板函数完全相同，那么在调用的时候，编译器会默认为你调用的时非类型的。为什么呢？我是这样理解的：函数模板的实例化是需要花费时间和空间的，编译器肯定会选择时间与空间开支较少的。

### 函数指针-如何理解typedef void (*pfun)(void)

#### 问题:

在刚接触typedef void (*pfun)(void) 这个结构的时候，存在疑惑，为什么typedef后只有一“块”东西，而不是两“块”东西呢？那是谁“替代”了谁啊？我总结了一下，一方面是对typedef的概念不清晰，另一方面受了#define的影响，犯了定向思维的错误。

#### 概念理解：

-typedef 只对已有的类型进行别名定义，不产生新的类型；

-# define只是在预处理过程对代码进行简单的替换。

清晰了解两个概念后，发现它们就是两个不同的概念，并没有太多的联系。

**类比理解：**

```c++
typedef  unsigned int  UINT32;  // UINT32 类型是unsigned int
UINT32 sum;                     // 定义一个变量：int sum;
typedef  int  arr[3];           // arr 类型是 int[3];（存放int型数据的数组）
arr a;                          // 定义一个数组：int a[3];
```

同理：

```c++
typedef  void (*pfun)(void);         // pfun 类型是 void(*)(void)
pfun main;                          // 定义一个函数：void (*main)(void);
```

在博客上看到一个经典的[函数指针](http://www.cnblogs.com/shenlian/archive/2011/05/21/2053149.html)用例：

```c++
#include<stdio.h>
typedef int (*FP_CALC)(int, int);
//注意这里不是函数声明而是函数定义，它是一个地址，你可以直接输出add看看
int add(int a, int b)
{
      return a + b;
}
int sub(int a, int b)
{
      return a - b;
}
int mul(int a, int b)
{
    return a * b;
}
int div(int a, int b)
{
      return b? a/b : -1;
}
//定义一个函数，参数为op，返回一个指针。该指针类型为 拥有两个int参数、
//返回类型为int 的函数指针。它的作用是根据操作符返回相应函数的地址
FP_CALC calc_func(char op)
{
      switch (op)
      {
           case '+' : return add;   // 返回函数的地址
           case '-' : return sub;
           case '*' : return mul;
           case '/' : return div;
           default:
	     return NULL;
      }
      return NULL;
}
//s_calc_func为函数，它的参数是 op，返回值为一个拥有两个int参数、返回类型为int 的函数指针
int (*s_calc_func(char op)) (int, int)
{
      return calc_func(op);
}
//最终用户直接调用的函数，该函数接收两个int整数，和一个算术运算符，返回两数的运算结果
int calc(int a, int b, char op)
{
	FP_CALC fp = calc_func(op);       
    // 根据预算符得到各种运算的函数的地址
     int (*s_fp)(int, int) = s_calc_func(op); 
// 用于测试
// ASSERT(fp == s_fp);                          
// 可以断言这俩是相等的
      if (fp){
          return fp(a, b);  //根据上一步得到的函数的地址调用相应函数，并返回结果
      }
      else{
          return -1;
      }
}

void main()
{
      int a = 100, b = 20;
      printf("calc(%d, %d, %c) = %d\n", a, b, '+', calc(a, b, '+'));
      printf("calc(%d, %d, %c) = %d\n", a, b, '-', calc(a, b, '-'));
      printf("calc(%d, %d, %c) = %d\n", a, b, '*', calc(a, b, '*'));
      printf("calc(%d, %d, %c) = %d\n", a, b, '/', calc(a, b, '/'));
}
```

结合代码理解：

代码作者在注释中表述得很清楚，个人觉得最有意思就是一下这个函数：

FP_CALC calc_func(char op) <--> int (*calc_func(char op)) (int, int)

　　代码作者试图在断言中说明这个关系，相比较，还是FP_CALC calc_func(char op)函数更能表达编码者的意图：calc_func函数返回FP_CALC类型的指针，是一个函数指针，这个函数的形式是int (函数名)(int, int)，代码中int add(int a, int b)、int sub(int a, int b)…正是这样的格式。

[函数指针-如何理解typedef void (*pfun)(void)](https://blog.csdn.net/weixin_30840573/article/details/96461398)

### 引用

```c++
double vals[]={1,2,3,4};
int& setvalue(int i){
    return value[i];
}
setvalue(2) = 30; //函数可以作为左值，因为返回的是一个引用，但是引用不能更改变量范围，例如本文引用的是数组vals的成员，是一个全局变量，如果是在函数内定义的局部变量，则需要用static修饰。
```

判断是引用还是取地址，看是左值还是右值，**左值是引用，右值是取地址**。

### final

在类和函数后面：禁止继承和重载

### ->

指向结构体的指针调用成员，或者指向类对象的指针调用成员函数或者父类函数

```c++
class a {
    public:
    int b();
}
a *ptr;
ptr->b();
//-----------------------------------

struct b{
    int a ;
}
b* ptr;
ptr->a = 0;
```

### extern

extern可以不使用include导入相关的文件，就定义另一个文件的全局变量或者函数

### include

include导入头文件（**.h），相当于在编译的时候把头文件整个拷贝过来，头文件里只需要声明函数，不需要实现函数，函数体的实现在.c文件里，不需要include *.c文件。

### gdb调试

可以直接以调试模式执行程序

```c
gcc -o main main.c -g //-g就是加入gdb模式
```

或者找到正在运行的程序进行调试，前提就是程序编译时加入-g模式

```shell
#找到进程pid
ps -ef | grep xxx
#进入进程内 
gdb attach xxx
(gdb) list #查看源码
(gdb) r #运行run
(gdb) b #加入断点 
b pkt.c:22(在pkt.c文件的22行打断点) 
b eth_rcv （在函数eth_rcv入口打断点） 
info b；显示当前所有断点； 
d num；删除断点num； 
n num；向后执行num步
clear n #清除断点
(gdb) bt #查看函数调用栈的所有信息，当程序执行异常时，可通过此命令查看程序的调用过程；
(gdb) c #继续执行
(gdb) s #进入函数内部
(gdb) n #下一步
until：当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体；
until+行号： 运行至某行，不仅仅用来跳出循环；
finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息；
call 函数(参数)：调用程序中可见的函数，并传递“参数”，如：call gdb_test(55)；
quit：简记为 q ，退出gdb；
```

### 指针+1

指针 + 1 并不是指针代表的地址值 + 1.

指针变量加1，即向后移动1 个位置表示指针变量指向下一个数据元素的首地址。而不是在原地址基础上加1。至于真实的地址加了多少，要看原来指针指向的数据类型是什么。

```c
char a = 'a';
char *p = &a;
cout<<(void*)p<<" "<<(void*)(p+1)<<endl;
//输出：0012FF33  0012FF34
```

p指向的是一个字符，p+1就是移动一个字符大小，一个字符就是一个字节，所以p +1 代表的地址就比 p 代表的地址大1。

```c
int i = 1;
int *p = &i;
cout<<(void*)p<<" "<<(void*)(p+1)<<endl;
//输出：0012FF30  0012FF34
```

p指向的是一个整型，p+1就是移动一个整型大小，即移动4个字节，所以p+1代表的地址比p代表的地址大4.0