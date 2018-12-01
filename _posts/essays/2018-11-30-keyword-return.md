---
layout: post
title: return不同类型变量的区别
category: Cpp
description: 关于c/c++中return的总结（返回值类型），由return *this引起的思考。
---
>近段时间突然想到return *this这个东西，一时间糊涂了，总是在纠结return *this返回的是对象的引用还是对象本身。后来悟过来这应该和返回值类型有关。从这个也查到了c/c++return不同类型是如何实现的，做一个小结以便回顾。  

## 一 结论
  直接抛结论，方便回顾。  
- 主要讨论返回内部变量类型(包含指针)，返回内部变量类型的引用，返回结构体(类)，返回结构体(类)的引用这四种情况。  
- 主要讨论返回局部变量的情况，也就是在栈内存的变量。
1. 返回内部变量类型时，直接返回变量本身。但是中间有一个拷贝的过程，会将局部变量拷贝至一个寄存器，再将寄存器的值返回给赋值的变量。也就是常说的会有一个复制的过程。值得注意的是，返回一个指针变量也是一样的，但是请注意指向栈内存的指针在函数调用结束后是无效的，因为栈空间已经释放，可能被其他操作覆盖。  
2. 返回值类型是局部变量的引用时，返回的是局部变量的地址，根据这个地址读出变量的值。此处有一个问题，局部变量所在的栈内存在调用结束时被销毁，按理说不应该能够根据地址读出正确的变量值。但是实际测试发现，返回局部变量的引用依然可以得到正确的变量值。  
3. 返回一个类对象时和情况1是一样的，返回的是对象的一个拷贝（通常是一个被叫做匿名对象的东西）。这也是为什么不推荐直接返回类对象的原因，因为拷贝一个类需要花费更多的时间，而且还需要再次调用类的拷贝构造函数，因为这真的是一个拷贝过程。  
4. 返回一个类对象的引用和情况2是一样的，直接返回地址，也无需调用拷贝构造函数。此处返回的类对象应该是由参数传入的类对象引用而不应该是返回局部类对象的引用！！


## 二 基础编码分析  
### 1.返回内部变量类型
源码如下  
```
#include <stdio.h>
#include <stdlib.h>
int func()
{
    int rst = 0;
    return rst;
}
void* func2()
{
    void * p = NULL;
    return p;
}
char& fun3()
{
    char ch = 0;
    return ch;
}
int main()
{
    int ret = func();
    func2();
    void* p = func2();
    fun3();
    char ch = fun3();
}
```
调试上述代码，查看相应的汇编码如下
```
int main()
{
00FA1880  push        ebp  
00FA1881  mov         ebp,esp  
00FA1883  sub         esp,0E4h  
00FA1889  push        ebx  
00FA188A  push        esi  
00FA188B  push        edi  
00FA188C  lea         edi,[ebp-0E4h]  
00FA1892  mov         ecx,39h  
00FA1897  mov         eax,0CCCCCCCCh  
00FA189C  rep stos    dword ptr es:[edi]  
00FA189E  mov         ecx,offset _A5C8350F_testreturn@cpp (0FAC003h)  
00FA18A3  call        @__CheckForDebuggerJustMyCode@4 (0FA1212h)  

	int ret = func();
00FA18A8  call        func (0FA138Eh)  
00FA18AD  mov         dword ptr [ret],eax  
	func2();
00FA18B0  call        func2 (0FA1389h)  
	void* p = func2();
00FA18B5  call        func2 (0FA1389h)  
00FA18BA  mov         dword ptr [p],eax  
	fun3();
00FA18BD  call        fun3 (0FA1384h)  
	char ch = fun3();
00FA18C2  call        fun3 (0FA1384h)  
00FA18C7  mov         al,byte ptr [eax]  
00FA18C9  mov         byte ptr [ch],al  
	
}
```
首先看对与局部变量的返回是如何实现的，从上面的代码可以看到调用函数func后又从eax寄存器读出一个值付给了变量ret。也就是说eax保存了对于局部变量的返回。具体的实现可以进入函数func查看相应的汇编码如下：
```
int func()
{
00FA1810  push        ebp  
00FA1811  mov         ebp,esp  
00FA1813  sub         esp,0CCh  
00FA1819  push        ebx  
00FA181A  push        esi  
00FA181B  push        edi  
00FA181C  lea         edi,[ebp-0CCh]  
00FA1822  mov         ecx,33h  
00FA1827  mov         eax,0CCCCCCCCh  
00FA182C  rep stos    dword ptr es:[edi]  
00FA182E  mov         ecx,offset _A5C8350F_testreturn@cpp (0FAC003h)  
00FA1833  call        @__CheckForDebuggerJustMyCode@4 (0FA1212h)  
	int rst = 0;
00FA1838  mov         dword ptr [rst],0  
	return rst;
00FA183F  mov         eax,dword ptr [rst]  
}
```
关键代码在return rst下面：00FA183F  mov  eax,dword ptr [rst]
也就是return时把局部变量存储在了eax寄存器，返回调用函数后从该寄存器读出即为返回值。这也印证了上述的说法。  
### 2.返回指针变量
那对于指针变量的返回和普通变量类型有区别吗？继续查看上述的汇编代码：  
```
void* p = func2();
00FA18B5  call        func2 (0FA1389h)  
00FA18BA  mov         dword ptr [p],eax  
```
这两句与int行变量的返回完全一致，进入func2函数继续查看,关键代码如下：
```
	void * p = NULL;
00FA17D8  mov         dword ptr [p],0  
	return p;
00FA17DF  mov         eax,dword ptr [p]  
```
**从这一点可以看出来返回指针变量与其他变量的方式是完全一样的，这也可以理解，毕竟指针也是内部变量类型之一。**    
### 3.返回局部变量的引用
直接看调用fun3的汇编码：
```
	char ch = fun3();
00FA18C2  call        fun3 (0FA1384h)  
00FA18C7  mov         al,byte ptr [eax]  
00FA18C9  mov         byte ptr [ch],al  
```
直接的区别就是操作步骤增加了一步，之前从eax寄存器读出的就是返回值，而对于引用的返回确时[eax]也就是把eax寄存器保存的是值作为一个地址数据。至于这个地址是什么，需要进入fun3继续查看，关键代码如下：
```
	char ch = 0;
00FA1742  mov         byte ptr [ch],0  
	return ch;
00FA1746  lea         eax,[ch]  
```
最直接的变化就是00FA1746  lea   eax,[ch] **没有将ch变量的值放入eax寄存器而是放入了ch的地址**。这才有了上述函数调用结束后先去eax指向的地址读值再将该值赋予变量ch的两步过程。  
- 总结起来就是直接返回变量时，返回的是变量值的一个拷贝。而返回变量的引用则是返回变量的地址，再根据这个地址读出变量的值。  
## 三 更进一步-->返回值为class/struct

### 1.简单描述返回值为类对象的过程。

```c++
class classname
{... ...}
classname fun()
{
	classname p;
	return p;
}
int main()
{
    classname p;
    p = fun();
}
```

有如上示意代码，其大致过程：先在调用函数处创建匿名对象但**不初始化**2.将该匿名对象的**this指针**作为参数传入被调用函数fun3.在return时通过传入的指针调用匿名对象的**拷贝构造函数**4.结束时返回匿名对象的**地址**5.如果调用函数需要将返回的对象赋值给另一个类对象(示意中为p)，就会根据这个指针再进行一次复制（单纯的复制，不会调用其他函数）。所以上述代码调用了两次构造函数(两次classname p过程)和一次拷贝构造函数(在return p的时候将被调用函数的p作为匿名对象拷贝构造函数的参数)。

### 2.具体源码

```c++
#include <stdio.h>
#include <stdlib.h>
#pragma pack(1) 
struct Person
{
	int age;
	int id;
	int somethingelse;
	char name[12];
	void display()
	{
		printf("age :%X id:%X\n", age, id);
		printf("age:%X \n", &age);
		printf("id:%X \n", &id);
		printf("somethingelse:%X \n", &somethingelse);
		printf("name:%X \n", name);
	}
	Person()
	{
		printf("Call Person()\n");
	}
	Person(const Person&)
	{
		printf("Call Person(const Person&)\n");
	}
};

Person func1()
{
	Person p;
	return p;
}
Person func2()
{
	Person* per = new Person();
	return *per;
}
Person& func3()
{
	Person* p = new Person();
	return *p;
}
int main()
{
	Person p;
	func1().display();
	p = func1();
	p.display();
	func2().display();
	func3().display();
	p = func3();
	p.display();
	return 0;
}
```

上述三个函数func1,func2func3分别分别代表返回局部内对象(栈内存)，返回动态对象(堆内存)，返回动态对象的引用。为了简洁，函数中没有进性delete操作。

### 3.返回局部对象的过程

相应反汇编代码如下

```
	Person p;
01151E12  lea         ecx,[p]  
01151E15  call        Person::Person (0115114Ah)  
	func1().display();
01151E1A  lea         eax,[ebp-100h]  
01151E20  push        eax  
01151E21  call        func1 (0115144Ch)  
01151E26  add         esp,4  
01151E29  mov         ecx,eax  
01151E2B  call        Person::display (01151046h)  
	p = func1();
01151E30  lea         eax,[ebp-120h]  
01151E36  push        eax  
01151E37  call        func1 (0115144Ch)  
01151E3C  add         esp,4  
01151E3F  mov         ecx,dword ptr [eax]  
01151E41  mov         dword ptr [p],ecx  
01151E44  mov         edx,dword ptr [eax+4]  
01151E47  mov         dword ptr [ebp-1Ch],edx  
01151E4A  mov         ecx,dword ptr [eax+8]  
01151E4D  mov         dword ptr [ebp-18h],ecx  
01151E50  mov         edx,dword ptr [eax+0Ch]  
01151E53  mov         dword ptr [ebp-14h],edx  
01151E56  mov         ecx,dword ptr [eax+10h]  
01151E59  mov         dword ptr [ebp-10h],ecx  
01151E5C  mov         edx,dword ptr [eax+14h]  
01151E5F  mov         dword ptr [ebp-0Ch],edx  
	p.display();
01151E62  lea         ecx,[p]  
01151E65  call        Person::display (01151046h)  
```

首先是Person p;直接调用相应的构造函数，过程比较清晰。重点看返回对象的函数func1().display();可以看到在调用函数func1之前有一个入栈的操作，调用函数之前入栈的都是需要传入函数的参数，但是函数func1并没有需要显示传入的参数，那么这个入栈的数据大概率是隐式传输的参数this指针，也就是前面提到的匿名对象的指针，但是该匿名对象的创建过程代码中并没有体现。

进入函数func1()相应的关键汇编码如下：

```
	Person p;
01151A92  lea         ecx,[p]  
01151A95  call        Person::Person (0115114Ah)  
	return p;
01151A9A  lea         eax,[p]  
01151A9D  push        eax  
01151A9E  mov         ecx,dword ptr [ebp+8]  
01151AA1  call        Person::Person (01151375h)  
01151AA6  mov         eax,dword ptr [ebp+8]  
```

其中语句01151A9E  mov    ecx,dword ptr [ebp+8] 将传入的参数放置ecx寄存器之后调用拷贝构造函数(一般来说，函数调用时ebp+4存放返回地址，ebp+8存放第一个传入的参数)。进入 Person::Person (01151375h)关键 源码如下：

```
011518AC  push        ecx  
011518AD  lea         edi,[ebp-0CCh]  
011518B3  mov         ecx,33h  
011518B8  mov         eax,0CCCCCCCCh  
011518BD  rep stos    dword ptr es:[edi]  
011518BF  pop         ecx  
011518C0  mov         dword ptr [this],ecx 
```

上述代码很清除的表明ecx寄存器也就是上一步中传入的参数确实是当前构造函数的this指针，也就是匿名对象的地址。在拷贝构造函数完成之后，还有01151AA6  mov         eax,dword ptr [ebp+8] 操作也就是将this指针放入eax寄存器。之后在函数调用处通过该指针调用返回的对象的成员函数display();

至此返回类对象的过程完毕。

从p = func1()的汇编码中可以看到其差别主要是返回之后的一些复制操作，其余完全相同。

### 4.返回动态动态对象的过程

相应的反汇编码如下：

```
	func2().display();
01151E6A  lea         eax,[ebp-140h]  
01151E70  push        eax  
01151E71  call        func2 (01151456h)  
01151E76  add         esp,4  
01151E79  mov         ecx,eax  
01151E7B  call        Person::display (01151046h)  
```

类比可知，该过程与返回局部类对象完全一致。

### 5.返回动态对象的引用

返回动态对象的引用不同与上述过程，没有调用前函数入栈的操作。其反汇编码如下：

```
	func3().display();
01151E80  call        func3 (01151451h)  
01151E85  mov         ecx,eax  
01151E87  call        Person::display (01151046h)  
	p = func3();
01151E8C  call        func3 (01151451h)  
01151E91  mov         ecx,dword ptr [eax]  
01151E93  mov         dword ptr [p],ecx  
01151E96  mov         edx,dword ptr [eax+4]  
01151E99  mov         dword ptr [ebp-1Ch],edx  
01151E9C  mov         ecx,dword ptr [eax+8]  
01151E9F  mov         dword ptr [ebp-18h],ecx  
01151EA2  mov         edx,dword ptr [eax+0Ch]  
01151EA5  mov         dword ptr [ebp-14h],edx  
01151EA8  mov         ecx,dword ptr [eax+10h]  
01151EAB  mov         dword ptr [ebp-10h],ecx  
01151EAE  mov         edx,dword ptr [eax+14h]  
01151EB1  mov         dword ptr [ebp-0Ch],edx  
	p.display();
01151EB4  lea         ecx,[p]  
01151EB7  call        Person::display (01151046h)
```

进入fun3查看如下：

```
	Person* p = new Person();
01151C27  push        18h  
01151C29  call        operator new (01151352h)  
01151C2E  add         esp,4  
01151C31  mov         dword ptr [ebp-0ECh],eax  
01151C37  mov         dword ptr [ebp-4],0  
01151C3E  cmp         dword ptr [ebp-0ECh],0  
01151C45  je          func3+7Ah (01151C5Ah)  
01151C47  mov         ecx,dword ptr [ebp-0ECh]  
01151C4D  call        Person::Person (0115114Ah)  
01151C52  mov         dword ptr [ebp-0F4h],eax  
01151C58  jmp         func3+84h (01151C64h)  
01151C5A  mov         dword ptr [ebp-0F4h],0  
01151C64  mov         eax,dword ptr [ebp-0F4h]  
01151C6A  mov         dword ptr [ebp-0E0h],eax  
01151C70  mov         dword ptr [ebp-4],0FFFFFFFFh  
01151C77  mov         ecx,dword ptr [ebp-0E0h]  
01151C7D  mov         dword ptr [p],ecx  
	return *p;
01151C80  mov         eax,dword ptr [p]  
```

可以看到return *p时只有一步将对象地址放入寄存器的操作，没有拷贝的过程(没有匿名对象产生，也就没有相应的匿名对象this指针，无从调用)

也就是说返回类对象引用其实是返回其地址，这和返回内部变量类型是一致的，同样的禁止返回局部类对象的引用，编译器不会报错误，但这是一个危险操作。

全文完毕。

后记：感谢 Icoding_F2014博主的详尽分析，本文基本上算是对其分析过程的验证。

* 参考： [Icoding_F2014对于上述过程的论证](https://blog.csdn.net/jmh1996/article/details/78384083)

修订记录：    2018-12-2 01:37       补充返回类对象的有关内容。