[[Java并发]]
![image-20250528222550309](Java%E5%B9%B6%E5%8F%91-ReetrantLock.assets/image-20250528222550309.png)

## Lock的抽象

![image-20250528222643868](Java%E5%B9%B6%E5%8F%91-ReetrantLock.assets/image-20250528222643868.png)

## ReentrantLock 原理详解

### 一、概述

`ReentrantLock` 是 Java 提供的一个可重入的互斥锁，位于 `java.util.concurrent.locks` 包下。它比内置锁 `synchronized` 更加灵活，支持尝试获取锁、超时、中断等高级特性。

其核心特点是：**可重入性（Reentrancy）**

---

### 二、可重入性的含义

#### 什么是可重入？

> 可重入是指一个线程在持有锁的情况下，可以再次进入被该锁保护的代码块。

这避免了死锁的发生，并简化了多层方法调用中锁的使用。

#### 示例：

```java
ReentrantLock lock = new ReentrantLock();

public void methodA() {
    lock.lock();
    try {
        // do something
        methodB();  // 调用另一个需要相同锁的方法
    } finally {
        lock.unlock();
    }
}

public void methodB() {
    lock.lock();
    try {
        // do something else
    } finally {
        lock.unlock();
    }
}
```

- 在 `methodA` 中已经获取了锁；
- 再次调用 `methodB` 仍能成功获取锁；
- 因为 `ReentrantLock` 支持同一个线程多次获取同一把锁。

#### 实现原理

- 每次成功获取锁，内部会维护一个计数器（state），表示当前线程的重入次数。
- 每次释放锁，计数器减一，直到为 0 时才真正释放锁。

---

### 三、公平锁 vs 非公平锁

`ReentrantLock` 支持两种模式的锁：**公平锁（Fair）** 和 **非公平锁（Nonfair）**

#### 1. 公平锁（Fair）

- 线程按照请求锁的顺序来获取锁。
- 后来的线程必须等待前面的线程获取并释放锁后才能尝试获取。
- 更加“公平”，但性能略低。

#### 2. 非公平锁（默认）

- 不保证线程获取锁的顺序。
- 当锁被释放时，新到达的线程可能直接抢占锁，而不必等待队列中的线程。
- 性能更高，但可能导致某些线程“饥饿”。

#### 构造方式：

```java
// 默认是非公平锁
ReentrantLock lock = new ReentrantLock();

// 显式指定为公平锁
ReentrantLock fairLock = new ReentrantLock(true);
```

---

### 四、公平锁与非公平锁的实现差异

#### AQS 中的 `tryAcquire` 方法

`ReentrantLock` 的公平性和非公平性主要体现在 `tryAcquire` 方法的实现上。

##### 1. 非公平锁逻辑（NonfairSync）

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 直接尝试 CAS 获取锁，不检查队列是否有等待线程
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 可重入逻辑
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

##### 2. 公平锁逻辑（FairSync）

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 多了一个 hasQueuedPredecessors 判断：队列中是否已有等待者？
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 可重入逻辑
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

---

### 五、性能与适用场景对比

| 特性         | 公平锁                     | 非公平锁                 |
| ------------ | -------------------------- | ------------------------ |
| 是否按序获取 | 是                         | 否                       |
| 性能         | 较低（需遍历队列判断顺序） | 更高（允许插队）         |
| 线程饥饿问题 | 无                         | 存在一定风险             |
| 适用场景     | 对顺序敏感，如资源调度系统 | 高并发环境，对性能要求高 |

---

### 六、总结

| 特性              | 描述                                                     |
| ----------------- | -------------------------------------------------------- |
| **可重入性**      | 同一线程可多次获取同一把锁，避免死锁                     |
| **公平锁**        | 严格按照线程排队顺序获取锁，适用于对顺序有严格要求的场景 |
| **非公平锁**      | 允许插队，性能更高，是默认行为                           |
| **基于 AQS 实现** | 使用 AQS 的状态管理机制（state）和等待队列进行资源协调   |

### 补充-为什么非公平锁效率更高？

非公平锁的性能通常被认为比公平锁更高，主要是由于以下几个原因：

1. **减少上下文切换**：减少了线程从等待状态到运行状态的上下文切换次数，因为线程不需要排队等待前一个线程完成操作后才开始尝试获取锁。
2. **降低队列管理是有开销的**
3. **提高资源利用率**：非公平策略使得那些刚刚准备好执行的线程能够尽快获取锁并执行，而不是强制它们必须等待已经在队列中的线程先执行。这样可以更有效地利用CPU时间，特别是在高并发环境下，有助于提升系统的整体吞吐量。

