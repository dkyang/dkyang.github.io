---
layout:     post
title:      Muduo源码阅读(1)：Reactor模式介绍
category:   post
description: 开源项目Muduo源码阅读第一篇，介绍其核心的Reactor模式
tags: 开源项目 网络编程 源码阅读
---
Muduo是一个基于Reactor模式的非阻塞网络库，所以弄清楚Reactor模式对理解Muduo的设计非常重要。
## 意义及产生背景
Reactor模式经常被翻译为“反应器”模式，但我认为这个翻译并不夠直观。说白了，所谓react就是对外界事件的一种被动的处理。比如在零下的室外环境中，我们会情不自禁的起鸡皮疙瘩，这就是我们对“感到寒冷”这个事件的反应；再比如我们因中了六合彩而高兴的手舞足蹈，这就是对“中六合彩”这个事件的反应；对于一个软件而言，我们点击了界面上的某个按钮时会弹出对话框，弹出对话框就是对“点击按钮”事件的反应。上述几种情形中都不是主动的发出动作，而是由于事件的到来而触发了反应的产生。软件在很多时候都是对现实生活中各种现象建模再通过计算机完成求解的过程，而Reactor模式就是针对上述这些现象而提出的。

现在让我们用较专门的语言再来阐述Reactor模式。首先联想一下普通函数调用的机制：程序调用某函数，函数执行，程序等待，函数将结果和控制权返回给程序，程序继续处理[1]。这是一种顺序的处理方式，但Reactor并不会主动调用函数，而是首先将相应的处理函数注册到Reactor上，当我们感兴趣的事件发生时再通知Reactor调用之前注册的函数完成处理，这些函数就是所谓的“回调函数”。事件的种类可以多种多样，包括信号、I/O读写、定时、GUI消息等。Reactor模式有许多大名鼎鼎的实现，比如C中的libevent,Java的Netty以及现在很火的Node.js语言等等。

回到网络编程，我们处理的大部分问题其实就是各种各样的事件。服务器接收到客户端的连接、读数据、写数据、接收到错误，这些都是事件。一般的处理是一种主动调用函数的方式，比如最简单的TCP服务器模型：

	socket(...);
	bind(...);
	listen(...);
	for ( ; ; ) {
	    accept(...);
	    if ((childpid = fork()) == 0) {
	        ....
	    }
	}
服务器端在完成基本的套接字建立、绑定、监听后，在for循环中主动的调用accept等待客户端连接到来，如果没有新连接，会阻塞在accept上，如果有新连接，则fork出一个子进程用以完成连接的处理。我们知道，随着并发连接的增多，我们需要fork出很多的子进程来处理。（当然也有其他的客户/服务器编程范式[2]，比如一个客户一个线程）。但这样的方式会带来各种问题[3],比如效率、编程复杂度、移植性等，具体可参考[5]。而Reactor模式在高并发的场景下就有了优势。Reactor模式一般与非阻塞I/O结合，在处理一个事件时不用等待事件的完成，而是转而去处理其他的任务，当这个事件处理完毕后再去通知Reactor，从而用单个线程可以完成多线程才能完成的任务，提高了系统的吞吐量。[4]中用一个餐馆的例子解释了Reactor模式在高并发下的优势，很有意思。具体来说，Reactor模式有如下优点[1,3]：

* 响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的；
* 编程相对简单，可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销；
* 可扩展性，可以方便的通过增加Reactor实例个数来充分利用CPU资源；
* 可复用性，Reactor框架本身与具体事件处理逻辑无关，具有很高的复用性；

## Reactor模式的结构 
Reactor模式包括四个部分的组件[1]：

* Initialization Dispatcher
* Synchronous Event Demultiplexer
* Handles
* Event Handler

其中的Handles用于标识通过操作系统管理的各种资源，包括套接字、打开文件、锁、定时器等，在Unix系统中是文件描述符，在Windows系统中是句柄。Initialization Dispatcher实现了事件的注册、删除、分发的接口，是用于驱动的主模块。Synchronous Event Demultiplexer是它的一个组件，用于等待新事件的发生，其实这就是解复用的系统，包括我们熟知的select、poll、epoll等。当有事件在Handles上发生时，我们可以通过Synchronous Event Demultiplexer得知。此时Synchronous Event Demultiplexer通知Initialization Dispatcher，Dispatcher再根据事件的类型使用相应的Event Handler完成事件的处理。所以Event Handler是最终用于实际处理事件的组件，一般来说会根据事件类型的不同实现各种类型的钩子函数。下图就是Reactor模式的结构：

![Reactor](/images/2014-4-18-muduo-1/reactor.jpg)

具体的处理过程如下：

1. 一个应用将某个Event Handler注册到Initialization Dispatcher上，并且指定其感兴趣的事件，希望当有感兴趣的事件在关联的handle上发生时去通知这个Event Handler。
2. 当Event Handler注册完毕后，Initialization Dispatcher在一个事件循环中通过Synchronous Event Demultiplexer等待事件的发生。比如一个监听套接字等待accept一个新的客户端连接。
3. 当与Handle关联的资源就绪时（比如当一个TCP套接字可读），Synchronous Event Demultiplexer会通知Initialization Dispatcher有事件发生。
4. 此时Initialization Dispatcher找到和这个Handle关联的Event Handler，根据事件的类型调用Event Handler相应的钩子函数完成事件处理。

[1] [libevent源码深度剖析：Reactor模式](http://cpp.ezbty.org/content/science_doc/libevent%E6%BA%90%E7%A0%81%E6%B7%B1%E5%BA%A6%E5%89%96%E6%9E%90%EF%BC%9Areactor%E6%A8%A1%E5%BC%8F)

[2] 《Unix Network Programming》 v3 30章

[3] [Reactor by DC Schmidt](www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf‎)

[4] http://daimojingdeyu.iteye.com/blog/828696

[5] [c10k problem](http://www.kegel.com/c10k.html)
