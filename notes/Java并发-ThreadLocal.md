[[Java并发]]

**通过为每个线程提供一个线程独享的变量副本的方式**，**以用空间换时间的思想从另一个角度解决了线程安全问题。**

ThreadLocal 也叫线程局部变量,为每个线程提供一个独立的变量副本，来避免变量共享的思路.

## ThreadLocal是什么？

![图片](Java%E5%B9%B6%E5%8F%91-ThreadLocal.assets/651588e16f0c23da74cdf98025a88c94.jpg)

ThreadLocalMap 用数组存储数据:包含了一个 **Entry 数组**，每个 Entry 对象包含一个 ThreadLocal 变量的引用和一个对应的值。

### ThreadLocalMap中Entry的设计原理

Entry继承自弱引用类WeakRefernce。

key是弱引用，value是强引用。





## FastThreadLocal

