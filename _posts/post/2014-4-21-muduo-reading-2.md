---
layout:     post
title:      Muduo源码阅读(2)：Muduo中的Reactor模式
category:   post
description: 介绍Muduo各模块与Reactor模式各组件的对应关系
tags: 开源项目 网络编程 源码阅读
---
Muduo是一个以Reactor模式为核心的网络库，所以Reactor模式的各个组件在Muduo中都有对应的部分。虽然在具体实现和理念上muduo有它自身的特点，但其内在的组织结构是高度符合Reactor模式的。可以说理解了Reactor模式，就会对muduo的设计有比较清楚的认识。muduo的相关介绍请见[1]及作者的相关博文。下图是muduo的类图，各个模块与Reactor模式中各组件的对应关系是：

 1. `Initialization Dispatcher` —— EventLoop 
 2. `Synchronous Event Demultiplexer` —— Poller 
 3. `Handles` —— FileDescriptor、Channel 
 4. `Event Handler` —— TcpConnection、Acceptor、Connector
![class diagram](/images/2014-4-21-muduo-2/muduo_uml.gif)

上述的对应关系并不完全严格，但从功能上有这样大体的关系。其实仅仅从这两幅图的结构上大家也可以看到一定的相似之处吧。

让我们看这幅类图上的TcpConnection、Acceptor、Connector这三个类，功能分别是：

* `TcpConnection`：封装一次Tcp连接的过程，同时我们可以在这个类中指定连接建立后对服务的处理过程。
* `Acceptor`：用于初始化服务端的socket套接字，包括监听、绑定等，完成对新连接的accept。
* `Connector`： 封装客户端的主动连接过程。FIXME
这三个类实现的是客户端/服务器编程中典型的功能，但为什么要将功能划分成这样的三个类实际上也是有章可循的。

在说明这种设计背后的原因时，先让我们考虑另外一种设计方式。我们知道在面向连接的协议中有两种角色，即主动请求连接的角色和被动接受连接的角色，对应到TCP协议中，就是客户端和服务端。对于套接字编程来说，二者都需要建立套接字并绑定到某个地址上。我们可以只定义一个类Socket同时描述客户端及服务器的套接字，使用时仅仅通过调用不同的构造函数及接口函数来区分二者。考虑下面的类定义：

	class Socket 
	{
	public:
	    // 创建监听套接字
	    Socket(unsigned int port);
	    // 创建客户端连接套接字
	    Socket(const char *pHostname, unsigned int port);
	    int bind_listen();
	    int send(...);
	    int recv(...);
	    int accept(...);
	private:
	    int _fd;
	    struct sockaddr_in _sockAddress;
	}
有了上述的定义后，服务端的伪代码可以这样编写：

	Socket sock(port);
	sock.bind_listen();
	while (true) 
	{
	    sock.accept();
	    server_process();
	}
客户端可以有如下结构:

	Socket sock(pHostname, port);
	sock.connect();
	client_process();
但这种对客户端服务器不加区分的设计并不好，毕竟二者具有不同的初始化过程，而且`server_process()`和`client_process()`实际上是和连接的初始化过程无关的。很明显，上述的设计将客户端初始化、服务端初始化、服务处理这三部分混杂在了一起。要优化这样的一个设计，很直接的解决思想就是解耦，将上述三个部分分开。

ACE网络库的作者Schmit针对这个问题提出了一种`Acceptor and Connector`的模式[2]，专门用于将通信协议中的被动、主动这两种初始化角色和连接建立后要完成的任务解耦。客户端连接的主动初始化、服务端连接的被动初始化、服务的处理都可以视为事件的到来（这里怎么描述FIXME），所以回想Reactor模式，我们自然可以将这三种任务作为Event Handler实现，当特定事件到来时触发这三种任务的处理。FIXME:以下详述这三个模块的处理机制

如果我们深入Muduo中Acceptor、Connector、TcpConnection的源代码，会发现这三个类有以下几个共同点：

 1. 包含Channel类型的成员变量，也是除了EventLoop之外，Muduo中仅有的包含Channel的三种类型。
 2. 有类似handleRead()、handleWrite()的私有函数。 
 3. 有回调函数作为成员变量，并且有设置该回调函数的接口。

有这几个共同点并非偶然，它们与Event Handler的实现关系非常密切。具体的实现过程是这样的：这三个类中的回调函数成员变量会在`handle*()`类型的函数中调用，用于处理读写等功能； 而`handle*()`函数会被绑定到Channel成员变量channel_ 中，这样channel_ 实际上就拥有了特定类型的钩子函数。通过Reactor机制，会在特定类型事件发生时调用相应的钩子函数，这样Acceptor、Connector、TcpConnection就通过事件驱动的方式完成了实现。

上述过程与Muduo的回调函数实现机制密切相关，我们会在以后的文章中介绍这方面的内容。而在schmit的原始论文中采用的是继承方式，Acceptor、Connector以及ServiceHandler都继承了EventHandler接口，这里不再展开说了，有兴趣的可以看下他的论文[2]。

可以看到这样的一种“模式”相对上文举的将几个部分混杂一起的例子要好很多。我们完全可以根据需要编写函数指定被动、主动初始化，服务处理过程。三者之间基本没有耦合，在保证了模块间良好结构的同时也具有很好的灵活性。

[1] http://blog.csdn.net/solstice/article/details/5848547

[2] Acceptor and Connector by DC Schmit