[[Java并发]]

![image-20250528221020379](Java%E5%B9%B6%E5%8F%91-AQS.assets/image-20250528221020379.png)

`AbstractQueuedSynchronizer`（简称 AQS）是 Java 并发包 `java.util.concurrent` 中实现锁和同步器的基础框架。它通过一个 `volatile int state` 变量来表示共享资源的状态，并使用一个 **FIFO（先进先出）的等待队列** 来管理多个线程对资源的竞争。

AQS 支持两种模式：
- **独占模式（Exclusive）**：如 ReentrantLock
- **共享模式（Shared）**：如 CountDownLatch、Semaphore


## 核心组件

### 1. 状态变量：`volatile int state`

```java
private volatile int state;
```

- `state` 是 AQS 的核心状态变量，用于标识当前共享资源是否被占用。
- 通过 `getState()`、`setState(int newState)` 和 `compareAndSetState(int expect, int update)` 方法进行访问和修改。
- 使用 `volatile` 保证多线程间的可见性，CAS 保证原子性操作。

### 2. 等待队列（CLH 队列）

AQS 内部维护了一个 **FIFO 的双向队列（CLH 队列）**，每个节点代表一个等待获取锁的线程。

- 队列中的节点（Node）包含线程引用、等待状态等信息。
- 当线程尝试获取锁失败时，会被封装成 Node 节点加入队列尾部，并进入阻塞状态。
- 队头节点是即将或正在尝试获取资源的节点。

### 3. 等待状态（waitStatus）

每个 Node 节点都有一个 `waitStatus` 字段，用来标记当前节点线程的等待状态，可能的取值如下：

| 状态值           | 含义                                                 |
| ---------------- | ---------------------------------------------------- |
| `0`              | 默认状态，表示当前节点在队列中处于等待通知状态       |
| `SIGNAL (-1)`    | 表示后继节点需要被唤醒                               |
| `CANCELLED (1)`  | 当前线程因超时或中断被取消                           |
| `CONDITION (-2)` | 表示该节点处于条件队列中                             |
| `PROPAGATE (-3)` | 仅在共享模式下使用，表示后续节点可以继续传播释放操作 |



## 核心方法与流程

### acquire(int arg)

这是 AQS 提供的公共方法，用于线程尝试获取资源（如锁）：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && 
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

#### 流程说明：

1. **tryAcquire(int arg)**  
   - 子类需重写此方法，实现具体的获取资源逻辑（如 CAS 修改 state）。
   - 如果返回 true，表示获取成功；否则进入排队等待。

2. **addWaiter(Node mode)**  
   - 将当前线程包装为 Node 节点，加入到等待队列尾部。

3. **acquireQueued(final Node node, int arg)**  
   - 进入自旋过程，只有当当前节点是队头的下一个节点时才允许尝试获取资源。
   - 其他节点则进入阻塞状态，直到被前驱节点唤醒。

> **优化策略：**
>
> - 仅队头的后继节点会自旋尝试获取锁；
> - 其他节点则调用 `park()` 进入阻塞状态，避免 CPU 空转。



## 释放资源：release(int arg)

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

- `tryRelease(int arg)`：由子类实现，用于释放资源（如减少 state）。
- 如果释放成功，则唤醒等待队列中的下一个节点（即 head 的后继节点）。



## 总结

| 组件                    | 描述                                       |
| ----------------------- | ------------------------------------------ |
| `volatile int state`    | 表示资源状态，通过 CAS 更新，保证并发安全  |
| FIFO 队列               | 管理等待获取资源的线程，保障公平性         |
| `tryAcquire/tryRelease` | 模板方法，由子类实现具体逻辑               |
| 自旋 + 阻塞机制         | 队头后继节点自旋尝试获取锁，其余节点阻塞   |
| 等待状态（waitStatus）  | 控制线程行为，支持中断、条件等待等复杂逻辑 |


## 示例应用场景

- `ReentrantLock`：基于 AQS 实现可重入的独占锁
- `CountDownLatch`：基于 AQS 实现计数器控制线程等待
- `Semaphore`：基于 AQS 实现资源许可控制
- `FutureTask`：基于 AQS 实现异步任务执行
