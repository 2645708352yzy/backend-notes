[[计算机网络]]
### Socket的创建

服务端线程调用`accept`系统调用后开始`阻塞`，当有客户端连接上来并完成`TCP三次握手`后，`内核`会创建一个对应的`Socket`作为服务端与客户端通信的`内核`接口。

一切皆是文件:内核创建出`Socket`之后，会将这个`Socket`放到当前进程所打开的文件列表中管理起来。

#### 进程中管理文件列表结构

![图片](%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C-Socket%E7%BB%93%E6%9E%84.assets/640.webp)

- `struct tast_struct`是内核中用来表示进程的一个数据结构，它包含了进程的所有信息。

- 进程内打开的所有文件是通过一个数组`fd_array`来进行组织管理，数组的下标即为我们常提到的`文件描述符`，数组中存放的是对应的文件数据结构`struct file`。每打开一个文件，内核都会创建一个`struct file`与之对应，并在`fd_array`中找到一个空闲位置分配给它，数组中对应的下标，就是我们在`用户空间`用到的`文件描述符`。

- 用于封装文件元信息的内核数据结构`struct file`中的`private_data`指针指向具体的`Socket`结构。

- `struct file`中的`file_operations`属性定义了文件的操作函数，不同的文件类型，对应的`file_operations`是不同的，针对`Socket`文件类型，这里的`file_operations`指向`socket_file_ops`。

> 我们在`用户空间`对`Socket`发起的读写等系统调用，进入内核首先会调用的是`Socket`对应的`struct file`中指向的`socket_file_ops`。**比如**：对`Socket`发起`write`写操作，在内核中首先被调用的就是`socket_file_ops`中定义的`sock_write_iter`。`Socket`发起`read`读操作内核中对应的则是`sock_read_iter`。

#### Socket内核结构

![图片](%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C-Socket%E7%BB%93%E6%9E%84.assets/640-1747622847166-3.webp)

在我们进行网络程序的编写时会首先创建一个`Socket`，然后基于这个`Socket`进行`bind`，`listen`，我们先将这个`Socket`称作为`监听Socket`。

1. 当我们调用`accept`后，内核会基于`监听Socket`创建出来一个新的`Socket`专门用于与客户端之间的网络通信。并将`监听Socket`中的`Socket操作函数集合`（`inet_stream_ops`）`ops`赋值到新的`Socket`的`ops`属性中。

   >`监听的 socket`和真正用来网络通信的 `Socket`，是两个 Socket，一个叫作`监听 Socket`，一个叫作`已连接的Socket`。

2. 接着内核会为`已连接的Socket`创建`struct file`并初始化，并把Socket文件操作函数集合（`socket_file_ops`）赋值给`struct file`中的`f_ops`指针。然后将`struct socket`中的`file`指针指向这个新分配申请的`struct file`结构体。

> 内核会维护两个队列：
>
> - 一个是已经完成`TCP三次握手`，连接状态处于`established`的连接队列。内核中为`icsk_accept_queue`。
> - 一个是还没有完成`TCP三次握手`，连接状态处于`syn_rcvd`的半连接队列。

3. 然后调用`socket->ops->accept`，从`Socket内核结构图`中我们可以看到其实调用的是`inet_accept`，该函数会在`icsk_accept_queue`中查找是否有已经建立好的连接，如果有的话，直接从`icsk_accept_queue`中获取已经创建好的`struct sock`。并将这个`struct sock`对象赋值给`struct socket`中的`sock`指针。


> `struct sock`在`struct socket`中是一个非常核心的内核对象，正是在这里定义了我们在介绍`网络包的接收发送流程`中提到的`接收队列`，`发送队列`，`等待队列`，`数据就绪回调函数指针`，`内核协议栈操作函数集合`



- 根据创建`Socket`时发起的系统调用`sock_create`中的`protocol`参数(对于`TCP协议`这里的参数值为`SOCK_STREAM`)查找到对于 tcp 定义的操作方法实现集合 `inet_stream_ops` 和`tcp_prot`。并把它们分别设置到`socket->ops`和`sock->sk_prot`上。