[[JVM]]

### JVM中的对象

![图片](JVM-9-%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B.assets/8dd01e6aba41677759be5de3fdf6bac4.png)

####  对象头
  - Mark word：存储运行时数据。
  - Klass指针：指向元数据指针
  - 当对象是数组时，记录数组长度的length。

##### 运行时数据：

![图片](JVM-9-%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B.assets/9dc7f15194edd89bea5b0e074e9ca1ed.png)

#####  元数据指针-OOP-Klass

通过这个指针，JVM 知道当前这个对象是哪一个类的实例。
#### 实例数据

- 基本数据类型
- 引用类型

### OOP-Klass模型

> oop：普通对象指针
>
> Klass：用来表示JVM层里某一个对象的具体类型

JVM是如何知道内存中的某一个对象具体是哪个类？

**JVM对象存储方式：**

- 对象实例（oop）保存在堆上
- 对象元数据（Klass）保存在方法区上
- 对象的引用则保存在栈上

示例

![图片](JVM-9-%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B.assets/ed8a3652fa9b21ecebf3d8bc6960d717.png)

![图片](JVM-9-%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B.assets/da2ba2db0d80773242172e9cfbba191d.png)

#### 应用-多态

面向对象编程有三大特性，分别是封装、继承和多态。

Java中的多态有三种写法，分别是重写、接口实现、抽象方法实现。

**函数表功能**

函数表（Function Table）也叫做方法表（Method Table），是Klass的一部分。

通过函数表，Java虚拟机能够根据方法的索引或名称来查找并调用相应的方法。

**多态的实现**

当通过父类的引用调用方法的时候，Java虚拟机会根据实际对象的类型，在函数表里查找对应的方法地址，然后进行方法调用。Java 采用的这种动态绑定机制，是实现多态特性的重要手段之一。

![图片](JVM-9-%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B.assets/42356005e69a5b2bfc604050eb652422.png)
