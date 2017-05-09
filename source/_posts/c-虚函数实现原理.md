---
title: c++虚函数实现原理
date: 2017-04-25 14:49:19
categories: 面试基础必备
---
------
## **c++虚函数实现原理**
下图中，我们在子类中覆盖了父类的f()函数:
![virtual_func1](/img/virtual_func1.jpg)
<!-- more -->

下面是对于子类实例中的虚函数表的图:
![vtable](/img/vtable.jpg)

源码参考:
```c++
#include "stdafx.h"
#include <iostream>

using namespace std;

class Base1 
{
public:
	virtual void f() { cout << "Base1::f" << endl; }
	virtual void g() { cout << "Base1::g" << endl; }
	virtual void h() { cout << "Base1::h" << endl; }
	void g2() { cout << "Base1::g2" << endl; }

};

class Base2 
{
public:
	virtual void f() { cout << "Base2::f" << endl; }
	virtual void g() { cout << "Base2::g" << endl; }
	virtual void h() { cout << "Base2::h" << endl; }
	void g2() { cout << "Base2::g2" << endl; }
};

class Base3 
{
public:
	virtual void f() { cout << "Base3::f" << endl; }
	virtual void g() { cout << "Base3::g" << endl; }
	virtual void h() { cout << "Base3::h" << endl; }
	void g2() { cout << "Base3::g2" << endl; }

};

class Derive : public Base1, public Base2, public Base3 
{
public:
	virtual void f() { cout << "Derive::f" << endl; }
	virtual void g1() { cout << "Derive::g1" << endl; }
	void g2() { cout << "Derive::g2" << endl; } // 会保存在代码区

	int a = 1;
};

void test1()
{
	typedef void(*Fun) (void);
	Fun pFun = NULL;

	Derive d;
	cout << "sizeof(Derive)" << sizeof(d) << endl;

	int** pVtab = (int**)&d;

	// Derive::a
	int a = (int)*((int *)&d + 3);
	cout << a << endl;

	//Base1's vtable

	//pFun = (Fun)*((int*)*(int*)((int*)&d+0)+0);

	pFun = (Fun)pVtab[0][0];

	pFun();

	//pFun = (Fun)*((int*)*(int*)((int*)&d+0)+1);

	pFun = (Fun)pVtab[0][1];

	pFun();

	//pFun = (Fun)*((int*)*(int*)((int*)&d+0)+2);

	pFun = (Fun)pVtab[0][2];

	pFun();

	//Derive's vtable

	//pFun = (Fun)*((int*)*(int*)((int*)&d+0)+3);

	pFun = (Fun)pVtab[0][3];

	pFun();

	//The tail of the vtable

	pFun = (Fun)pVtab[0][4];

	cout << pFun << endl;

	//Base2's vtable

	//pFun = (Fun)*((int*)*(int*)((int*)&d+1)+0);

	pFun = (Fun)pVtab[1][0];

	pFun();

	//pFun = (Fun)*((int*)*(int*)((int*)&d+1)+1);

	pFun = (Fun)pVtab[1][1];

	pFun();

	pFun = (Fun)pVtab[1][2];

	pFun();

	//The tail of the vtable

	pFun = (Fun)pVtab[1][3];

	cout << pFun << endl;

	//Base3's vtable

	//pFun = (Fun)*((int*)*(int*)((int*)&d+1)+0);

	pFun = (Fun)pVtab[2][0];

	pFun();

	//pFun = (Fun)*((int*)*(int*)((int*)&d+1)+1);

	pFun = (Fun)pVtab[2][1];

	pFun();

	pFun = (Fun)pVtab[2][2];

	pFun();

	//The tail of the vtable

	pFun = (Fun)pVtab[2][3];

	cout << pFun << endl;
}

int _tmain(int argc, _TCHAR* argv[])
{
	test1();
	return 0;
}
```

输出:
![vfun_output](/img/vfun_output.png)

**安全性**
**一、通过父类型的指针访问子类自己的虚函数**

我们知道，子类没有重载父类的虚函数是一件毫无意义的事情。因为多态也是要基于函数重载的。虽然在上面的图中我们可以看到Base1的虚表中有Derive的虚函数，但我们根本不可能使用下面的语句来调用子类的自有虚函数：

Base1 *b1 = new Derive();

b1->f1(); //编译出错

任何妄图使用父类指针想调用子类中的未覆盖父类的成员函数的行为都会被编译器视为非法，所以，这样的程序根本无法编译通过。但在运行时，我们可以通过指针的方式访问虚函数表来达到违反C++语义的行为。

**二、访问non-public的虚函数**

另外，如果父类的虚函数是private或是protected的，但这些非public的虚函数同样会存在于虚函数表中，所以，我们同样可以使用访问虚函数表的方式来访问这些non-public的虚函数，这是很容易做到的。
