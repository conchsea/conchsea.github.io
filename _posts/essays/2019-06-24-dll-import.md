---
layout: post
title: 动态链接库的显式调用和隐式调用
category: Cpp
---
---
> dll的两种调用方式，隐式调用(加载时调用)和显示调用(运行时调用)。

## 一  链接库的使用  

### 1.1   动态/静态链接库

​     win平台的静态链接库以.lib为后缀，动态链接库以.dll为后缀。静态链接库的使用比较容易，导入后使用即可。动态链接库的使用稍有不同，分为显式调用和隐式调用两种。

### 1.2 隐式调用

​    动态链接库的隐式调用依赖.lib和.dll两个文件。此处的lib文件不是静态库，而是与dll相对应的一个lib文件，里面存储着dll中的导出符号和一部分"桩代码"，此处的lib可以被成为导入库。它的作用可以简单理解为通过找到dll中执行函数的地址。

   以vs环境为例， 隐式调用时将lib所在路径设置到附加库目录，同时在附加库中添加库名。之后再工程中包含.h文件即可像正常调用函数一样使用dll。

### 1.3 显示调用

​     如果需要调用的库仅提供了dll，而没有相应的lib符号库，还可以使用显示调用的方式方式来进性函数调用。这种方式的前提是已经知道了dll中所提供的函数名称，及其参数列表，返回值。

​    调用方式如下例所示：

```
typedef uint32_t(*pfun)(uint8* pu8Para，uint32_t u32Length);  
  
int main()  
{  
  
    HINSTANCE hDLL;  
    pfun fun;  
    hDLL=LoadLibrary(TEXT("fundll.dll"));  //加载DLL文件  
    if(hDLL == NULL)
    {
        //error
        return -1;
    }
    fun=(pfun)GetProcAddress(hDLL,"fun");  //取DLL中的函数地址，fun为dll中提供函数的真是名称 
    uint32_t u32Ret = fun(NULL, 0); //函数调用
    FreeLibrary(hDLL);   //卸载动态库
    
    return u32Ret;
}
```



​     **总的来说，隐式调用优点时使用方便，直接调用即可，缺点是程序加载时即导入了dll，导致程序运行时内存占用增加。而显式调用的优缺点刚好相反。一般来说，大型程序开发时优先使用显示调用的方法。**

