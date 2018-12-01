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
## 三 更进一步-->返回值为class
 



























































