---
layout: post
title: 回调函数及其应用场景
category: Cpp
description: 对于回调函数的理解总是云里雾里，这次做一个系统的总结。
---
>日常开发中遇到过使用回调函数的场景，但是对其具体定义和实现方式一直没有清晰的认识，导致平时能不用就不用，这次做一个总结克服它。

## 一  回调函数的定义
### 1.1 百度百科定义

如果你把函数的指针（地址）**作为参数**传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。回调函数**不是由该函数的实现方直接调用**，而是在特定的事件或条件发生时由**另外的一方调用的**，用于对该事件或条件进行响应。

### 1.2其他定义

1. 回调函数是指**使用者自己定义一个函数**，实现这个函数的程序内容，然后把这个函数（入口地址）作为参数传入别人（或系统）的函数中，由别人（或系统）的函数在运行时来调用的函数。**函数是你实现的**，但由别人（或系统）的函数在运行时通过参数传递的方式调用，这就是所谓的回调函数。简单来说，就是**由别人的函数运行期间通过函数指针来回调你实现的函数**。 

2. 所谓回调，就是模块A要通过模块B的某个函数b()完成一定的功能，但是函数b()自己无法实现全部功能，需要反过头来调用模块A中的某个函数a()来完成，这个a()就是回调函数。 

   

   总结来说，**回调函数这一设计允许了底层代码调用在高层定义的子程序。**

## 二  回调函数简单实例
### 1.1无参数的回调实例

```
//定义带参回调函数
void PrintfText(char* s) 
{
    printf(s);
}
//定义实现带参回调函数的"调用函数"
void CallPrintfText(void (*callfuct)(char*),char* s)
{
    callfuct(s);
}
//在main函数中实现带参的函数回调
int main(int argc,char* argv[])
{
    CallPrintfText(PrintfText,"Hello World!\n");
    return 0;
}
```

### 1.2将上述代码修改为带参函数

```
//定义带参回调函数
void PrintfText(char* s) 
{
    printf(s);
}
//定义实现带参回调函数的"调用函数"
void CallPrintfText(void (*callfuct)(char*),char* s)
{
    callfuct(s);
}
//在main函数中实现带参的函数回调
int main(int argc,char* argv[])
{
    CallPrintfText(PrintfText,"Hello World!\n");
    return 0;
}
```

以上两个实例对回调函数的定义有了清晰的说明，但是没有指明回调函数的真正用途：**回调函数这一设计允许了底层代码调用在高层定义的子程序。**这也是前文提到的，回调函数主要是为了让其他人已经实现的函数调用，而不是让自己实现的函数调用。第三章将给出回调函数的实用用途。

## 三  Qt5中引入的消息重定向机制使用的回调函数

### 1.1使用场景

一般在开发Qt程序时，主要由两种调试方式：一种是在debug模式下调试，另一种是在程序中使用qDebug()等函数打印信息帮助调试。一旦程序release投入实际使用，这些打印信息将无法被开发人员看到，qt5使用消息重定向机制使得打印信息可以重定向输出至log文件。这样即使软件在使用过程中发生问题也可以凭借日志文件追溯。

如下是Qt帮助文档对于注册回调函数的描述：

```
Installs a Qt message handler which has been defined previously. Returns a pointer to the previous message handler.
The message handler is a function that prints out debug messages, warnings, critical and fatal error messages. The Qt library (debug mode) contains hundreds of warning messages that are printed when internal errors (usually invalid function arguments) occur. Qt built in release mode also contains such warnings unless QT_NO_WARNING_OUTPUT and/or QT_NO_DEBUG_OUTPUT have been set during compilation. If you implement your own message handler, you get total control of these messages.
The default message handler prints the message to the standard output under X11 or to the debugger under Windows. If it is a fatal message, the application aborts immediately.
Only one message handler can be defined, since this is usually done on an application-wide basis to control debug output.
To restore the message handler, call qInstallMessageHandler(0).
```



### 1.2Qt回调代码实例

如下代码是qt提供的例子：

```
#include <qapplication.h>
#include <stdio.h>
#include <stdlib.h>
void myMessageOutput(QtMsgType type, const QMessageLogContext &context, const QString &msg)
{
    QByteArray localMsg = msg.toLocal8Bit();
    switch (type)
    {
        case QtDebugMsg:
        fprintf(stderr, "Debug: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
        case QtInfoMsg:
        fprintf(stderr, "Info: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
        case QtWarningMsg:
        fprintf(stderr, "Warning: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
        case QtCriticalMsg:
        fprintf(stderr, "Critical: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
        case QtFatalMsg:
        fprintf(stderr, "Fatal: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        abort();
    }
}

int main(int argc, char **argv)
{
    qInstallMessageHandler(myMessageOutput);
    QApplication app(argc, argv);
    ...
    return app.exec();
}

```

如上代码，函数void myMessageOutput(QtMsgType type, const QMessageLogContext &context, const QString &msg)可以看做是我们自己实现的回调函数，qInstallMessageHandler(myMessageOutput)是Qt提供的注册回调函数。之后，一旦Qt提供的几种消息类型发生就会调用我们定义的回调函数。

以上代码稍做修改即可实现重定向消息输出至log文件，代码如下：

```
void myMessageOutput(QtMsgType type, const QMessageLogContext &context, const QString &msg)
{
    QByteArray localMsg = msg.toLocal8Bit();
    switch (type) {
    case QtDebugMsg:
        fprintf(stderr, "Debug: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
    case QtInfoMsg:
        fprintf(stderr, "Info: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
    case QtWarningMsg:
        fprintf(stderr, "Warning: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
    case QtCriticalMsg:
        fprintf(stderr, "Critical: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
    case QtFatalMsg:
        fprintf(stderr, "Fatal: %s (%s:%u, %s)\n", localMsg.constData(), context.file, context.line, context.function);
        break;
    }
}
int main(int argc, char *argv[])
{
    qInstallMessageHandler( myMessageOutput );
    QCoreApplication a(argc, argv);

    ...
}
```

经过修改后的回调函数可以实现将打印信息及其所在的，文件，函数，代码行等输出至log文件。

**这个例子可以帮助我们清晰的认识回调函数的具体应用场景。**

## 四 标准回调函数定义方式

> 回调函数的定义及其实现方式并不复杂，如果不清楚其应用场景只是一味的看概念确实容易对其理解有偏差。本小结将给出关于回调函数及注册回调函数一些标准的定义。  

### 1.回调函数定义

定义时最好符合如下函数原型：typedef void (*SCT_XXX)(LPVOID lp, const CBParamStruct& cbNode);   

其中SCT_XXX是回调函数名称，lp是回调上下文，CBParamStruct是回调参数，一般由于要回调的参数不止一个，所以定义一个结构体比较方便。

### 2.注册回调函数

注册回调函数原型：void RCF_XXX(SCT_XXX pfn, LPVOID lp);

其中，RCF_XXX是注册函数名，pfn是回调函数名称（是指针），lp是回调上下文。 

> 后记CSDN论坛一个[关于为什么要使用回调函数](https://bbs.csdn.net/topics/390081829)的帖子可能对深入理解回调函数有帮助，毕竟争论出真知。

**参考：**

* [ioleon13的博客](https://www.cnblogs.com/ioleon13/archive/2010/03/02/1676621.html)
* [百度百科回调函数词条](https://baike.baidu.com/item/%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0/7545973?fr=aladdin)

修订记录：2018.12.02  22:46  完成初稿

