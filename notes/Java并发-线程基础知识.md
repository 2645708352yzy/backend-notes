[[Java并发]]

- [ ] 线程的状态

## 创建和运行线程

### 直接使用 Thread

```java
// 创建线程对象
Thread t = new Thread("t1") {
    @Override
    public void run() {
        // 要执行的任务
    }
};
// 启动线程
t.start();
```

### 使用 Runnable 配合 Thread  (推荐)

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        // 要执行的任务
    }
};
// 创建线程对象
Thread t = new Thread(runnable);
// 启动线程
t.start();
```

#### 示例

```java
// 创建任务对象
Runnable task2 = new Runnable() {
    @Override
    public void run() {
        log.debug("hello");
    }
};

// 参数1 是任务对象; 参数2 是线程名字
Thread t2 = new Thread(task2, "t2");
t2.start();
```

在 Java 8 以后可以使用lambda表达式来精简代码

```java
// 创建任务对象
Runnable task2 = () -> log.debug("hello");

// 参数1 是任务对象; 参数2 是线程名字，推荐
Thread t2 = new Thread(task2, "t2");
t2.start();
```

#### 原理 —— Thread 与 Runnable 的关系

```java
private Runnable target;

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

> 实际，就是在 Thread 中的 run 方法里面调用了 Runnable 方法的 run 方法来执行任务。

### FutureTask 配合 Thread

FutureTask能够接收Callable类型的参数，用来处理有返回结果的情况。

```java
@Slf4j(topic = "c.Test3")
public class FutureAndCallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 1.创建任务对象
        FutureTask<Integer> task = new FutureTask<>(new Callable<>() {
            @Override
            public Integer call() throws Exception {
                log.debug("running ...");
                Thread.sleep(4000);
                return 100;
            }
        });

        // 2.创建线程对象，并关联任务
        Thread t = new Thread(task, "t3");

        t.start();

        // 主线程运行到此处时，就会一直阻塞。直到 task 执行完毕后返回结果
        log.debug("{}",  task.get());
    }
}
```

## 查看进程线程的方法

![image-20250525223255018](Java%E5%B9%B6%E5%8F%91-%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.assets/image-20250525223255018.png)

## 线程运行的原理

### 栈和栈帧

我们都知道 JVM 中由堆、栈、方法区所组成，其中栈内存是给谁用的呢？其实就是线程，每个线程启动后，虚拟机就会为其分配一块栈内存。 

- 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存 
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法 

###  线程上下文切换

导致线程切换的原因

1. 线程的 <mark>cpu 时间片用完</mark> 
2. 垃圾回收（会暂停当前所有的工作线程，让垃圾回收的线程来运行）
3. 有更高优先级的线程需要运行 
4. 线程自发调用了 sleep()、yield()、wait()、join()、park()、synchronized、lock() 等方法 

当上下文切换发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是程序计数器（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，它是线程私有的 

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等 
- 频繁的上下文切换会影响性能

## 线程常见方法

以下是整理后的表格：

| 方法名                    | 是否静态方法 | 功能说明                        | 注意点                                                                                                              |
| ---------------------- | ------ | --------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| `start()`              | N      | 启动一个新线程，在新的线程运行`run`方法中的代码  | - 让线程进入就绪状态，代码不一定立刻运行（取决于CPU调度）<br>- 每个线程对象的`start`方法只能调用一次，多次调用会抛出`IllegalThreadStateException`                 |
| `run()`                | N      | 新线程启动后会调用的方法                | - 若构造`Thread`对象时传递了`Runnable`参数，线程启动后调用`Runnable`中的`run`方法<br>- 可通过创建`Thread`的子类对象覆盖默认行为                         |
| `join()`               | N      | 等待线程运行结束                    | 无                                                                                                                |
| `join(long n)`         | N      | 等待线程运行结果，最多等待`n`毫秒          | 若等待的线程已执行完毕，会提前结束`join`方法                                                                                        |
| `getId()`              | N      | 获取线程长整型的ID                  | ID唯一                                                                                                             |
| `getName()`            | N      | 获取线程名                       | 无                                                                                                                |
| `setName(String name)` | N      | 修改线程名                       | 无                                                                                                                |
| `getPriority()`        | N      | 获取线程优先级                     | 无                                                                                                                |
| `setPriority(int num)` | N      | 修改线程优先级                     | - Java中线程优先级为1~10的整数<br>- 较大优先级能提高线程被CPU调度的概率                                                                    |
| `getState()`           | N      | 获取线程状态                      | Java中线程状态用6个enum表示：`NEW`、`RUNNABLE`、`BLOCKED`、`WAITING`、`TIMED_WAITING`、`TERMINATED`                             |
| `isInterrupted()`      | N      | 判断是否被打断                     | 不会清除打断标记                                                                                                         |
| `isAlive()`            | N      | 判断线程是否存活（未运行完毕）             | 无                                                                                                                |
| `interrupt()`          | N      | 打断线程                        | - 若线程正在`sleep`、`wait`、`join`，会抛出`InterruptedException`并清除打断标记<br>- 若打断正在运行的线程，会设置打断标记<br>- `park`的线程被打断，也会设置打断标记 |
| `interrupted()`        | Y      | 判断当前线程是否被打断                 | 会清除打断标记，将打断标记置为`false`                                                                                           |
| `currentThread()`      | Y      | 获取当前正在执行的线程                 | 无                                                                                                                |
| `sleep(long n)`        | Y      | 让当前执行的线程休眠`n`毫秒，休眠时让出CPU时间片 | 无                                                                                                                |
| `yield()`              | Y      | 提示线程调度器让出当前线程对CPU的使用权       | 主要用于测试和调试                                                                                                        |

## start() 与 run()

### 结论

1. 直接调用run()方法，相当于同步。是在主线程中执行run()方法，并没有启动新的线程来执行！
2. 通过start()方法来启动线程，相当于异步。通过新的线程来间接执行run()中的代码！

### 示例

两者区别代码演示如下：

```java
@Slf4j(topic = "c.Test4")
public class ThreadRunTest {

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                log.debug("running...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t1");

        t1.run();
        log.debug("do other things...");
    }
}
```

```plain
22:36:54.587 c.Test4 [main] - running...
22:36:56.596 c.Test4 [main] - do other things...
```

```java
@Slf4j(topic = "c.Test4")
public class ThreadRunTest {

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                log.debug("running...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t1");

        // 将 run() 改成 start() 	
        t1.start();
        log.debug("do other things...");
    }
}
```

```
22:47:58.687 c.Test4 [t1] - running...
22:47:58.687 c.Test4 [main] - do other things...
```

### 扩展—— 线程执行前后状态信息变化

```java
@Slf4j(topic = "c.Test5")
public class ThreadStateTest {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                log.debug("running...");
            }
        }, "t1");

        // 查看线程执行前后的状态信息
        System.out.println(t1.getState());
        t1.start();
//        t1.start();   不能被多次调用，否则会报错
        System.out.println(t1.getState());
    }
}
```

```
NEW  
RUNNABLE
22:55:49.182 c.Test5 [t1] - running...
```

- NEW：初始状态，线程被创建出来但没有被调用start() 。
- RUNNABLE：运行状态，线程被调用了start()等待运行的状态。

## sleep() 与 yield()

![image-20250525225155932](Java%E5%B9%B6%E5%8F%91-%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.assets/image-20250525225155932.png)

**sleep()方法演示**

```java
@Slf4j(topic = "c.Test6")
public class ThreadSleepTest {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                try {
                    Thread.sleep(2000);  // 单位是毫秒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t1");

        t1.start();

        log.debug("t1 state 状态为：{}", t1.getState());
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 需要让主线程也休眠，不然主线程没有阻塞，获取的 t1 线程的状态为 RUNNABLE 运行状态
        log.debug("t1 state 状态为：{}", t1.getState());
    }
}
```

```
21:53:13.943 c.Test6 [main] - t1 state 状态为：RUNNABLE
21:53:14.459 c.Test6 [main] - t1 state 状态为：TIMED_WAITING
```

**interrupt() 方法演示**

```java
@Slf4j(topic = "c.Test7")
public class ThreadInterruptTest {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                log.debug("enter sleep...");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    log.debug("wake up...");
                    e.printStackTrace();
                }
            }
        }, "t1");

        t1.start();

        // 主线程 1s 后强制唤醒 t1 线程
        Thread.sleep(1000);
        log.debug("interrupt...");
        t1.interrupt();  // 唤醒睡眠线程
    }
}
```

```
22:02:59.140 c.Test7 [t1] - enter sleep...
22:03:00.151 c.Test7 [main] - interrupt...
22:03:00.151 c.Test7 [t1] - wake up...
java.lang.InterruptedException: sleep interrupted
  at java.base/java.lang.Thread.sleep(Native Method)
  at top.zhang.test.ThreadInterruptTest$1.run(ThreadInterruptTest.java:19)
  at java.base/java.lang.Thread.run(Thread.java:834)
```

**Timeunit 中 sleep() 方法演示**

```java
@Slf4j(topic = "c.Test8")
public class TimeUnitTest {
    public static void main(String[] args) throws InterruptedException {
        log.debug("start...");
        // 睡眠一秒
        TimeUnit.SECONDS.sleep(1);
        // 睡眠 10 毫秒
        TimeUnit.MILLISECONDS.sleep(10);
        // 睡眠 2 分钟
        TimeUnit.MINUTES.sleep(2);
        log.debug("end...");
    }
}
```

![image-20250525225614276](Java%E5%B9%B6%E5%8F%91-%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.assets/image-20250525225614276.png)

## 线程优先级

`setPrority(int newPrority)`：设置线程优先级

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
- 如果 cpu 比较忙，那么优先级高的线程会获得更多的时间片，但 cpu 闲时，优先级几乎没作用

总结：
不管是 yield() 还是优先级，他们都不能真正的去控制线程的调度，最终还是由操作系统的任务调度器来决定具体哪个线程分到更多的时间片。

### 应用 解除对CPU的使用

#### sleep() 实现

在没有利用 CPU 来计算时，不要让while(true)空转浪费 CPU，这时可以使用yield()或 sleep() 方法来让出 CPU 的使用权给其他的程序！

```java
while(true) {
    try {
        Thread.sleep(50);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

#### wait() 实现

```java
synchronized(锁对象) {
    while(条件不满足) {
        try {
            锁对象.wait();
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
    // do sth...
}
```

#### 条件变量实现

```java
lock.lock();
try {
    while(条件不满足) {
        try {
            条件变量.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    // do sth...
} finally {
    lock.unlock();
}
```

## join 方法详解

### 为什么需要 join

下面的代码，打印出来的r值是多少？

```java
static int r = 0;
public static void main(String[] args) throws InterruptedException {
    test1();
}

private static void test1() throws InterruptedException {
    log.debug("开始");
    Thread t1 = new Thread(() -> {
        log.debug("开始");
        sleep(1);
        log.debug("结束");
        r = 10;
    });
    t1.start();
    log.debug("结果为:{}", r);
    log.debug("结束");
}
```

**输出**

```
23:07:21.660 c.Test10 [main] - 开始
23:07:21.663 c.Test10 [Thread-0] - 开始
23:07:21.663 c.Test10 [main] - 结果为:0
23:07:21.664 c.Test10 [main] - 结束
23:07:22.670 c.Test10 [Thread-0] - 结束
```

**分析**

- 因为主线程和线程 t1 是并行执行的，t1 线程需要 1 秒之后才能算出 r=10 
- 而主线程一开始就要打印 r 的结果，所以就会打印出 r=0

**解决方法**

用`join()`，加在`t1.start()` 之后即可

```java
@Slf4j(topic = "c.Test10")
public class Test10 {
    static int r = 0;
    public static void main(String[] args) throws InterruptedException {
        test1();
    }

    private static void test1() throws InterruptedException {
        log.debug("开始");
        Thread t1 = new Thread(() -> {
            log.debug("开始");
            sleep(1);
            log.debug("结束");
            r = 10;
        });
        t1.start();
        
        // TODO 哪个线程调用就等待哪个线程结束
        // 主线程等待 t1 线程结束
        t1.join();

        log.debug("结果为:{}", r);
        log.debug("结束");
    }
}
```

## interrupt() 方法：打断线程

### **打断阻塞/等待状态的线程**

打断等待状态/阻塞状态的线程, 会抛出异常信息表示被打断，此时会清空打断状态，打断状态为false！

```java
public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
       log.debug("sleep...");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, "t1");

    t1.start();
    log.debug("interrupt...");
    // 打断正在阻塞的线程
    t1.interrupt();
    log.debug("打断标记：{}", t1.isInterrupted());  // isInterrupted()  判断线程是否被打断 | true：被打断  false：未被打断
}
```

```
22:46:20.766 c.Test11 [t1] - sleep...
22:46:20.766 c.Test11 [main] - interrupt...
22:46:20.769 c.Test11 [main] - 打断标记：false
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at top.zhang.test.Test11.lambda$main$0(Test11.java:18)
	at java.base/java.lang.Thread.run(Thread.java:834)
```



### **打断运行状态的线程**

打断正常运行的线程, 不会清空打断状态，打断状态为true！

```java
@Slf4j(topic = "c.Test12")
public class Test12 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            // 被打断后并不会影响线程的正常运行，只是让线程知道了由其他线程在打断。具体打断后的操作要看具体实现
            while (true) {
                if (Thread.currentThread().isInterrupted()) {
                    // 为 true，则证明被打断了。退出循环
                    log.debug("被打断了，退出循环");
                    break;
                }
            }
        }, "t1");

        t1.start();

        Thread.sleep(1000);
        log.debug("interrupt...");
        t1.interrupt();
    }
}
```

```
23:02:57.524 c.Test12 [main] - interrupt...
23:02:57.526 c.Test12 [t1] - 被打断了，退出循环
```

### **设计模式 —— 两阶段终止**

>  **有两个线程t1、t2。如何在线程t1中优雅的终止t2？**

![image-20250528210333660](Java%E5%B9%B6%E5%8F%91-%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.assets/image-20250528210333660.png)

## Park()方法：阻塞线程

> - park()不是 Thread 类中提供的方法，是LockSupport工具类中提供的方法。
> - 作用也是让当前线程停下来，进入阻塞状态
> - 可以通过interrupt()方法来打断正在park的线程，打断状态变为true
> - park()是不可重入的。一旦打断状态变为true后，再次调用interrupt()方法会失效。一个线程不可多次调用park()

1. LockSupport.park()：让此方法所在的线程阻塞。可以通过interrupt()方法来打断正在阻塞的线程，也可通过unpark()方法
2. LockSupport.unpark(线程名)：让指定的线程执行，取消阻塞

## 线程的状态

![image.png](Java%E5%B9%B6%E5%8F%91-%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.assets/1649066126899-0b988758-8c4a-4e7d-b882-85bfae2d61a8.webp)



