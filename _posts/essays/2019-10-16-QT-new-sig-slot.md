---
layout: post
title: QT5信号槽的一些特殊用法
category: QT
---
---
>信号槽最常用的方式，就是1对1connect，之后出发操作。这里总结了一些特殊的便捷操作，例如：QT5信号槽新的连接方式，多个控件信号连接到一个槽，在信号槽中使用lambda表达式等。

## 一 QT5支持的信号槽新语法
### 1.1   信号语法对比

```
老语法：使用SIGNAL及SLOT宏来强转
connect(sender,SIGNAL(signal()),receiver,SLOT(slot()));
新语法：第2，4个参数从函数变为函数名
connect(sender, &Sender::valueChanged, receiver, &Receiver::updateValue);
最简单的例子：第一个参数是信号发送者，第二个参数具体信号，第三个参数信号接收者，第四个参数是槽函数
connect(&ui->btn_test, &QpushButton::click, pobj, &obj::somefun);
```

### 1.2新语法的优点

- 编译期间检查信号和槽是否存在，它们的类型，及Q_OBJECT是否丢失，出错直接编译报错。老语法无法在编译器检查错误。
- 新语法的信号可以连接QObject的任何普通成员方法，不仅仅是定义的槽函数。
- 新语法在使用自定义类型参数时，不需要再前置注册这些参数。

## 二  多个信号连接到同一个槽
### 1.1 多控件连接

```
{
    ....
    connect(&ui->btn_test, &QpushButton::click, pobj, &obj::slotfun);
    connect(&ui->btn_start, &QpushButton::click, pobj, &obj::slotfun);
    connect(&ui->spinbox_test, &QspinBox::valueChanged, pobj, &obj::slotfun);
    .....
}
void obj::slotfun()
{
    QObject* obj = sender();
    if (QStringLiteral("btn_test") == obj->objectname())
    {
        ....
    }
    else if (QStringLiteral("btn_start") == obj->objectname())
    {
        ....
    }
    else if (QStringLiteral("spinbox_test") == obj->objectname())
    {
        ....
    }
}
```

​    按照以上方式，可以实现多控件连接同一槽函数，简化代码实现，在一个槽函数中做统一处理。

## 三  信号槽+lambda表达式

​    lambda表达式是一种匿名函数，代码中如果槽函数的功能比较单一且规模很小，可以使用lambda表达式简化代码。

```
....
connect(&ui->btn_test, &QpushButton::click, [](){std::cout<<"just test lambda<<std::endl;"});
```

