---
title: 'c++ extern,static,const,volatile关键字'
date: 2017-04-26 11:02:14
categories: 面试基础必备
---
------
## **c++ extern,static,const,volatile关键字**
### **extern**
***作用一(和"C"一起连用)：***
告诉编译器编译函数时按照C的规则去翻译函数名，而C++翻译的因为有函数重载原因，翻译规则和C不一样，这样避免在库中找不到符号。
C语言不支持extern "C"语法，只适用于C++，写法如下:
<!-- more -->
```c++
#ifdef __cplusplus
extern "C" {
#endif

// 正式定义。。。

#ifdef __cplusplus
}
#endif
```

***作用二：***
声明函数或全局变量的作用范围的关键字，其声明的函数和变量可以在本模块活其他模块中使用
默认函数是extern的，所以可以不写。
全局变量就需要写，不然那会被c++看成定义式。
看以下例子:
```c++
#include<stdio.h>

namespace myname {
    int var = 42;
}

extern "C" int _ZN6myname3varE;

int main()
{
    printf("%d\n", _ZN6myname3varE);
    myname::var ++;
    printf("%d\n", _ZN6myname3varE);

    printf("%p\n",&_ZN6myname3varE);
    printf("%p\n", &myname::var);
    return 0;
}
```
输出:
```c++
42
43
0x601040
0x601040
```
在这个例子中，我们根据g++编译器的符号修饰规则，仿造了一个C变量（gcc不进行符号修饰），欺骗了编译器，把myname::var 和 _ZN6myname3varE当成了同一个变量了。
所以声明变量最好加extern

### **static**
**C 语言的 static 两种用途：**

***1. 静态局部变量***
> 用于函数体内部修饰变量，这种变量的生存期长于该函数。

***2. 静态全局变量或函数***
> 静态全局变量：定义在函数体外，用于修饰全局变量，表示该变量只在本文件可见。

**C++ 语言的 static 两种用途：**

***1. 静态数据成员***
> 生存期大于 class 的对象（实体 instance）。
> 静态数据成员是每个 class 有一份

***2. 静态成员函数***
> 静态成员函数不能访问非静态(包括成员函数和数据成员)，但是非静态可以访问静态
> 调用静态成员函数，可以用类名::函数名调用

### **const**

***1. 定义常量***
> const修饰变量，变量的value是不可变的。

***2. 指针使用CONST***
> (1) 指针本身是常量不可变
> ```c++ 
 char* const pContent;
 ```
> (2) 指针所指向的内容是常量不可变
> ```c++ 
 const char* pContent;
 ```
> (3) 两者都不可变
> ```c++ 
 const char* const pContent;
 ```

***3. 类相关CONST***
> (1) const修饰成员变量
> const修饰类的成员函数，表示成员常量，不能被修改，同时它只能在初始化列表中赋值。
> ```c++ 
 class A
 { 
     …
     const int nValue;         //成员常量不能被修改
     …
     A(int x): nValue(x) { } ; //只能在初始化列表中赋值
 }
 ```
> (2) const修饰成员函数
> const修饰类的成员函数，则该成员函数不能修改类中任何非const成员函数。一般写在函数的最后来修饰。
> ```c++ 
 class A
 { 
     …
     void function()const; //常成员函数, 它不改变对象的成员变量.
     //也不能调用类中任何非const成员函数。
 }
 ```
> (3) const修饰类对象/对象指针/对象引用
> const修饰类对象表示该对象为常量对象，其中的任何成员都不能被修改。对于对象指针和对象引用也是一样。
> const修饰的对象，该对象的任何非const成员函数都不能被调用，因为任何非const成员函数会有修改成员变量的企图。
> ```c++ 
 class AAA
 { 
     void func1(); 
     void func2() const; 
 } 
 const AAA aObj; 
 aObj.func1(); //×
 aObj.func2(); //正确
 
 const AAA* aObj = new AAA(); 
 aObj-> func1(); //×
 aObj-> func2(); //正确
 ```

### **volatile**
volatile 影响编译器编译的结果,指出，volatile 变量是随时可能发生变化的，与volatile变量有关的运算，不要进行编译优化，以免出错

volatile变量的几个例子:
> 1) 并行设备的硬件寄存器（如：状态寄存器） 
> 2) 一个中断服务子程序中会访问到的非自动变量(Non-automatic variables) 
> 3) 多线程应用中被几个任务共享的变量 

***问题:***
> 1) 一个参数既可以是const还可以是volatile吗？解释为什么。 
> 2) 一个指针可以是volatile 吗？解释为什么。 
> 3) 下面的函数有什么错误：
> ```c++
 int square(volatile int *ptr) 
 { 
     return *ptr * *ptr; 
 }
 ```

***答案：*** 
> 1) 是的。一个例子是只读的状态寄存器。它是volatile因为它可能被意想不到地改变。它是const因为程序不应该试图去修改它。 
> 
> 2) 是的。尽管这并不很常见。一个例子是当一个中服务子程序修该一个指向一个buffer的指针时。 
> 
> 3) 这段代码有点变态。这段代码的目的是用来返指针*ptr指向值的平方，但是，由于*ptr指向一个volatile型参数，编译器将产生类似下面的代码： 
> ```c
 int square(volatile int *ptr) 
 { 
     int a,b; 
     a = *ptr; 
     b = *ptr; 
     return a * b; 
 }
 ```
> 
> 由于*ptr的值可能被意想不到地该变，因此a和b可能是不同的。结果，这段> 代码可能返不是你所期望的平方值！正确的代码如下： 
> ```c
 long square(volatile int *ptr) 
 { 
     int a; 
     a = *ptr; 
     return a * a; 
 } 
 ```

