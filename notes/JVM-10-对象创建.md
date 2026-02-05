[[JVM]]

### 对象创建的标准流水线

![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/2b50cb50ee2c7c7487c51445c4b61837.png)



### 对象创建的两段论

#### Java的语言层和JVM层是两个层级

![image-20250517102525630](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/image-20250517102525630.png)

##### JVM层

- 类加载（获取图纸）：类加载首先会读取 .class 里的数据，把里面的数据放到运行时的方法区中，并在运行时的堆上同步创建一个 java.lang.Class 对象。
- 空间分配（选址）：JVM需要分配可用的内存空间给待创建的Java对象。
- 初始化默认值
>分配内存空间给Java对象的方式
>
>- 指针碰撞：![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/169c6f43be7d7064480a058325b82b41.png)
>
>- 空闲列表：维护一个全局的空闲列表，用来记录当前JVM堆的使用情况。![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/473651bbaf8f3a230e7cbfd81e5d02fd.png)
>- TLAB（Thread Local Allocation Buffer）：为每个线程分配一块区域以减少同步竞争

##### Java语言层

- 对象初始化：对已创建的对象执行初始化，包括执行变量初始化、执行初始化语句块和构造器方法。

### 对象创建快捷流水线

![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/921670f62de3479b96ae84444eafda07.png)

这条全新的流水线主要由四个节点构成。

![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/45edac79a256a23126385b9105af8dde.png)

#### 对象逃逸分析

如果一个对象的使用超出当前方法，就叫做对象逃逸。

**对象逃逸分析示例：**

```java
public Book EscapeExample() {
    return new Book("Java Book"); //方法中创建的对象的使用范围超过了这个方法的范畴。
}
```

![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/7b19a118d38ea1d7ae3c0723bfa4eb68.png)

#### 标量替换

未发生逃逸的话，JVM实际创建这个对象的时候，可以不在堆上，而是改为直接创建这个方法所使用的成员变量来直接替代。

**标量替换：**把一个对象拆散，将其成员变量恢复到基本类型来访问就称为标量替换。

```java
public String EscapeExample() {
    Book book = new Book("Book"); 
    return book.name 
}
```

```java
public String EscapeExample() {
    Book book = new Book("Book"); 
    return book.name 
}
```

#### 栈上分配

栈上分配指方法中声明的变量和创建的对象，直接从当前线程所使用的栈上分配空间，而不在堆上创建对象。

#### 同步消除

当一个对象经过了逃逸分析、标量替换以及栈上分配后，说明这个对象不会逃逸出当前线程。因此针对这个对象的所有同步措施都可以被删除掉。

![图片](JVM-10-%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA.assets/196aecb014076613434a31690a28900b.png)