# netty

## 介绍

Netty 是一个**异步的、基于事件驱动的网络应用框架**，用以快速开发高性能、高可靠性的网络 IO 程序。

Netty 主要针对在 **TCP 协议下，面向 Clients 端的高并发应用**，或者 Peer-to-Peer 场景下的大量数据持续传输的
应用。
Netty **本质是一个 NIO 框架**，适用于服务器通讯相关的多种应用场景

因为 Netty5 出现重大 bug，已经被官网废弃了，目前推荐使用的是 Netty4.x 的稳定版本

## 应用

阿里分布式服务框架 **Dubbo** 的 RPC 框架使用 Dubbo 协议进行节点间通信，**Dubbo 协议默认使用 Netty 作为基础通信组件**，用于实现各进程节点之间的内部通信

地图服务器之间可以方便的通过 Netty 进行高性能的通信及游戏、手游。

## BIO

I/O 模型简单的理解：就是**用什么样的通道进行数据的发送和接收**，很大程度上决定了程序通信的性能
Java 共支持 3 种网络编程模型/IO 模式：**BIO、NIO、AIO**

### 介绍

BIO(blocking I/O) ： **同步阻塞**，服务器实现模式为一个连接一个线程，即**客户端有连接请求时服务器端就需要启动一个线程进行处理**，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善(实现多个客户连接服务器)。

适用于**<u>*连接数目比较小且固定的架构*</u>**，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，程序简单易理解

### 流程

1)服务器端启动一个 ServerSocket
2)客户端启动 Socket 对服务器进行通信，默认情况下服务器端需要对每个客户 建立一个线程与之通讯

3)客户端发出请求后, 先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝
4)如果有响应，客户端线程会等待请求结束后，在继续执行

## NIO

### 介绍

java non-blocking IO  **同步非阻塞的**

NIO 有三大核心部分：**Channel(通道)，Buffer(缓冲区), Selector(选择器选择器)**

NIO 是 **面向缓冲区 ，或者面向 块 编程的编程的。**

通俗理解：NIO 是可以做到用一个线程来处理多个操作的。假设有 10000 个请求过来,根据实际情况，可以分配50 或者 100 个线程来处理。不像之前的阻塞 IO 那样，非得分配 10000 个。
**HTTP2.0 使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比 HTTP1.1 大了好**几个数量级**

###  NIO 的 Selector 、 Channel 和 Buffer 的关系

<img src="D:\dailySoftWare\typora\md\img\TIM截图20210308155056.png" alt="TIM截图20210308155056" style="zoom:50%;" />

1)每个 channel 都会对应一个 Buffer
2)**Selector 对应一个线程， 一个线程对应多个 channel(连接)**
3)该图反应了有三个 channel 注册到 该 selector //程序
4)程序切换到哪个 channel 是有事件决定的, Event 就是一个重要的概念
5)Selector 会根据不同的事件，在各个通道上切换
6)**Buffer 就是一个内存块 ， 底层是有一个数组**
7)**数据的读取写入是通过 Buffer**, 这个和 BIO , BIO 中要么是输入流，或者是输出流, 不能双向，但是 **NIO 的 Buffer 是可以读也可以写, 需要 flip 方法切换**
**channel 是双向的, 可以返回底层操作系统的情况**, 比如 Linux ， 底层的操作系统通道就是双向的.

### buffer

缓冲区（Buffer）：缓冲区**本质上是一个可以读写数据的内存块**，可以理解成是**一个容器对象容器对象(含数组含数组)**，该对象提供了一组方法，可以更轻松地使用内存块，，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化

Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer。

### channel

1)NIO 的通道类似于流，但有些区别如下：
通道可以**同时进行读写**，而流只能读或者只能写
通道可以实现**异步读写数据**
通道可以从缓冲读数据，也可以写数据到缓冲:
2)BIO 中的 stream 是单向的，例如 FileInputStream 对象只能进行读取数据的操作，而 **NIO 中的通道(Channel)是双向的，可以读操作，也可以写操作**。
3)Channel 在 NIO 中是一个接口

```java
public interface Channel extends Closeable{}
```

4)常 用 的Channel 类 有 ： FileChannel 、 DatagramChannel 、 ServerSocketChannel 和SocketChannel 。【ServerSocketChanne 类似 ServerSocket , SocketChannel 类似 Socket】
5)<u>**FileChannel 用 于 文 件 的 数 据 读 写， DatagramChannel 用 于UDP 的 数 据读 写 ， ServerSocketChannel 和SocketChannel 用于 TCP 的数据读写**</u>

### selector

Selector 能够检测多个注册的通道上是否有事件发生(注意:**多个 Channel 以事件的方式可以注册到同一个Selector**)，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。

```java
selector.select()//阻塞
selector.select(1000);//阻塞 1000 毫秒，在 1000 毫秒后返回
selector.wakeup();//唤醒 selector
selector.selectNow();//不阻塞，立马返还
```

### SelectionKey

表示 Selector 和网络通道的注册关系

int OP_ACCEPT：有新的网络连接可以 accept，值为 16
int OP_CONNECT：代表连接已经建立，值为 8
int OP_READ：代表读操作，值为 1
int OP_WRITE：代表写操作，值为 4
源码中：
public static final int OP_READ = 1 << 0;
public static final int OP_WRITE = 1 << 2;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_ACCEPT = 1 << 4;

### ServerSocketChannel

 ServerSocketChannel 在服务器端监听新的客户端 Socket 连接

### SocketChannel

网络 IO 通道，具体负责进行读写操作



## NIO 非阻塞 网络编程原理分析图

<img src="D:\dailySoftWare\typora\md\img\TIM截图20210316091031.png" alt="TIM截图20210316091031" style="zoom:50%;" />



1)当**客户端连接时，会通过 ServerSocketChannel 得到 SocketChannel**
2)**Selector 进行监听**select 方法, 返回有事件发生的通道的个数.
3)将 **socketChannel 注册到 Selector** 上, register(Selector sel, int ops), 一个 selector 上可以注册多个 SocketChannel
4)**注册后返回一个 SelectionKey, 会和该 Selector 关联**(集合)
5) **(有事件发生)，进一步得到各个 SelectionKey**
6)在**通过 SelectionKey反向获取 SocketChannel , 方法 channel()**
7)可以通过得到的 channel, 完成业务处理



## 零拷贝

零拷贝描述的是**CPU不执行拷贝数据**从一个存储区域到另一个存储区域的任务，这通常用于通过网络传输一个文件时以减少CPU周期和内存带宽。

#### mmp

mmap 通过内存映射，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网
络传输时，就可以减少内核空间到用户空间的拷贝次数

#### sendFile

Linux 2.1 版本 提供了 sendFile 函数，其基本原理如下：数据根本不经过用户态，直接从内核缓冲区进入到
Socket Buffer，同时，由于和用户态完全无关，就减少了一次上下文切换

#### 比较

1)mmap 适合小数据量读写，sendFile 适合大文件传输。
2)mmap 需要 4 次上下文切换，3 次数据拷贝；sendFile 需要 3 次上下文切换，最少 2 次数据拷贝。
3)sendFile 可以利用 DMA 【directmemory access 直接内存拷贝(不使用 CPU)】方式，减少 CPU 拷贝，mmap 则不能（必须从内核拷贝到 Socket 缓冲区）。





## AIO

Asynchronous I/O，即 AIO

AIO 即 NIO2.0，叫做**异步不阻塞的 IO**。AIO 引入异步通道的概念，采用了 **Proactor** 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于**连接数较多且连接时间较长的应用**





## NIO和BIO比较

1)**BIO 以流的方式处理数据,**而 **NIO 以块的方式处理数据**,块 I/O 的效率比流 I/O 高很多
2)BIO 是阻塞的，**NIO 则是非阻塞的**
3)BIO 基于字节流和字符流进行操作，而 **NIO 基于 Channel(通道)和 Buffer(缓冲区)进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。**Selector(选择器)用于监听多个通道的事件（比如：连接请求，数据到达等）**，因此使用单个线程就可以监听多个客户端通道

## 线程模型

目前存在的线程模型有：传统阻塞 I/O 服务模型、Reactor 模式

Reactor 的数量和处理资源池线程的数量不同，有 3 种典型的实现
单 Reactor 单线程；
单 Reactor 多线程；
主从 Reactor 多线程

Netty 线程模式(**Netty 主要基于主从 Reactor 多线程模型做了一定的改进，其中主从 Reactor 多线程模型有多个 Reactor**)

### 主从Reactor多线程

<img src="D:\dailySoftWare\typora\md\img\TIM截图20210316093907.png" alt="TIM截图20210316093907" style="zoom:50%;" />

1)Reactor 主线程 **MainReactor 对象通过 select 监听连接事件, 收到事件后，通过 Acceptor 处理连接事件**
2)当 Acceptor处理连接事件后，**MainReactor 将连接分配给 SubReactor**
3)**subreactor 将连接加入到连接队列进行监听,并创建 handler 进行各种事件处理**
4)当有新事件发生时， **subreactor 就会调用对应的 handler 处理**
5)**handler 通过 read 读取数据**，**分发给后面的 worker** 线程处理
6)**worker 线程池分配独立的 worker 线程进行业务处理**，并返回结果
7)handler 收到响应的结果后，再通过 send 将结果返回给 client
8)Reactor 主线程可以对应多个 Reactor 子线程, 即 MainRecator 可以关联多个 SubReactor



**<u>*Netty 主要基于主从 Reactors 多线程模型（如图）做了一定的改进*</u>**











