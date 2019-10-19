---
layout: post
title: QT信号槽ConnectionType的选择
category: QT
---
---
> qt使用connect函数连接信号槽时，最后一个参数enum Qt::ConnectionType往往被忽略而使用默认值，一般情况下这么做是没有问题的，但涉及到跨线程模块时，则需要小心指定ConnectionType，否则会达不到预期目的。

## 一  enum Qt::ConnectionType  

### 1.1   参数描述

* Qt::AutoConnection  (Default) If the receiver lives in the thread that emits the signal, Qt::DirectConnection is used. Otherwise, Qt::QueuedConnection is used. The connection type is determined when the signal is emitted.
* Qt::DirectConnection  The slot is invoked immediately when the signal is emitted. The slot is executed in the signalling thread.
* Qt::QueuedConnection  The slot is invoked when control returns to the event loop of the receiver's thread. The slot is executed in the receiver's thread.
* Qt::BlockingQueuedConnection Same as Qt::QueuedConnection, except that the signalling thread blocks until the slot returns. This connection must not be used if the receiver lives in the signalling thread, or else the application will deadlock.
* Qt::UniqueConnection  This is a flag that can be combined with any one of the above connection types, using a bitwise OR. When Qt::UniqueConnection is set, QObject::connect() will fail if the connection already exists (i.e. if the same signal is already connected to the same slot for the same pair of objects). This flag was introduced in Qt 4.6.

​       上面是官方手册给出的描述，该参数的默认值是AutoConnection ，此时ConnectionType受信号槽是否在同一线程限制，如果在同一线程中，则使用DirectConnection  ，否则连接类型为QueuedConnection。

​       DirectConnection是指信号emit时立即执行相应的槽函数，且槽函数在信号发射的的线程中执行。槽函数执行完毕后再执行emit后面的代码(相当于直接函数调用)

​       QueuedConnection连接信号槽时，信号会置入队列中，当槽函数所在的线程取得事件控制权后，再取出信号执行。

​      **如上，跨线程使用信号槽时，一定要指明连接类型。如果需要子线程立即响应信号，就需要用直接连接的方式，将槽函数放在信号所在线程执行。其他情况再可以使用队列连接的方式来进性**