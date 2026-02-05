[[Netty]]

## Reactor

`Reactor`是利用`NIO`对`IO线程`进行不同的分工：

- 使用前边我们提到的`IO多路复用模型`比如`select,poll,epoll,kqueue`,进行IO事件的注册和监听。
- 将监听到`就绪的IO事件`分发`dispatch`到各个具体的处理`Handler`中进行相应的`IO事件处理`。

通过`IO多路复用技术`就可以不断的监听`IO事件`，不断的分发`dispatch`，就像一个`反应堆`一样，看起来像不断的产生`IO事件`，因此我们称这种模式为`Reactor`模型。

### 单Reactor单线程

![图片](Netty-Reactor%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.assets/640.webp)

`Reactor模型`是依赖`IO多路复用技术`实现监听`IO事件`，从而源源不断的产生`IO就绪事件`，在Linux系统下我们使用`epoll`来进行`IO多路复用`，我们以Linux系统为例：

- 单`Reactor`意味着只有一个`epoll`对象，用来监听所有的事件，比如`连接事件`，`读写事件`。
- `单线程`意味着只有一个线程来执行`epoll_wait`获取`IO就绪`的`Socket`，然后对这些就绪的`Socket`执行读写，以及后边的业务处理也依然是这个线程。

### 单Reactor多线程

![图片](Netty-Reactor%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.assets/640-1747559075673-3.webp)

- 这种模式下，也是只有一个`epoll`对象来监听所有的`IO事件`，一个线程来调用`epoll_wait`获取`IO就绪`的`Socket`。
- 但是当`IO就绪事件`产生时，这些`IO事件`对应处理的业务`Handler`，我们是通过线程池来执行。这样相比`单Reactor单线程`模型提高了执行效率，充分发挥了多核CPU的优势。

### 主从Reactor多线程

做任何事情都要区分`事情的优先级`，我们应该`优先高效`的去做`优先级更高`的事情，而不是一股脑不分优先级的全部去做。

于是，`主从Reactor多线程`模型就产生了：

![图片](Netty-Reactor%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.assets/640-1747559122771-6.webp)

- 我们由原来的`单Reactor`变为了`多Reactor`。`主Reactor`用来优先`专门`做优先级最高的事情，也就是迎接客人（`处理连接事件`），对应的处理`Handler`就是图中的`acceptor`。
- 当创建好连接，建立好对应的`socket`后，在`acceptor`中将要监听的`read事件`注册到`从Reactor`中，由`从Reactor`来监听`socket`上的`读写`事件。
- 最终将读写的业务逻辑处理交给线程池处理。

### 总结Reactor线程模型运行机制

![image-20250519143658041](Netty-Reactor%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.assets/image-20250519143658041.png)

-   **连接注册**：Channel 建立后，注册至 Reactor 线程中的 Selector 选择器。
-   **事件轮询**：轮询 Selector 选择器中已注册的所有 Channel 的 I/O 事件。
-   **事件分发**：为准备就绪的 I/O 事件分配相应的处理线程。
-   **任务处理**：Reactor 线程还负责任务队列中的非 I/O 任务，每个 Worker
    线程从各自维护的任务队列中取出任务异步执行。

## Netty的IO模型

![图片](Netty-Reactor%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.assets/640-1747559173775-9.webp)

### EventLoop的实现

在 Netty 中 EventLoop 可以理解为 Reactor 线程模型的事件处理引擎，每个EventLoop 线程都维护一个 Selector 选择器和任务队列taskQueue。它主要负责处理 I/O 事件、普通任务和定时任务。

NioEventLoop的run()方法：

- 事件轮询select
- IO事件处理 processSelectedKeys(); 
- 任务处理 runAllTasks();

#### 事件处理机制

- BossEventLoopGroup 负责监听客户端的 Accept事件。当事件触发时，将事件注册至WorkerEventLoopGroup 中的一个NioEventLoop 上。每新建一个 Channel， 只选择一个 NioEventLoop
  与其绑定。
- NioEventLoop 完成数据读取后，会调用绑定的 ChannelPipeline进行事件传播。

#### 任务处理机制

NioEventLoop处理的任务类型基本可以分为三类。

1. **普通任务**：通过 NioEventLoop 的 execute() 方法向任务队列taskQueue 中添加任务。例如 Netty 在写数据时会封装 WriteAndFlushTask提交给 taskQueue。taskQueue 的实现类是多生产者单消费者队列
   MpscChunkedArrayQueue，在多线程并发添加任务时，可以保证线程安全。

2. **定时任务**：通过调用 NioEventLoop 的 schedule() 方法向定时任务队列scheduledTaskQueue添加一个定时任务，用于周期性执行该任务。例如，心跳消息发送等。定时任务队列scheduledTaskQueue 采用优先队列 PriorityQueue 实现。
3. **尾部队列**：tailTasks 相比于普通任务队列优先级较低，在每次执行完taskQueue中任务后会去获取尾部队列中任务执行。尾部任务并不常用，主要用于做一些收尾工作，例如统计事件循环的执行时间、监控信息上报等。
