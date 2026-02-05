[[JVM]]

反射（Reflection）是JVM提供的运行时机制 ，它允许程序在**运行期**借助 Reflection API 动态加载类或获取任何类的内部信息，动态创建对象并调用其属性，即使对象类型在编译期还是未知的。

#### 反射的实现原理

依赖类加载机制，思考：java.lang.Class对象的作用？

##### **反射API**

![图片](JVM-7-%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6.assets/75ab73d599f94c9f57481593545f59ae.png)

![img](JVM-7-%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6.assets/1dbe246e0a292bae6cddae3d650ace41.jpg)

##### 反射API的使用-获取Class对象

1. `Class.forName(String className)`
2. `.class`语法：必须在编译期就知道具体的类
3. Object类的方法` getClass()`

![图片](JVM-7-%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6.assets/b5e8b30dc56ae5a23e7d311784e73f52.png)

##### 反射API的使用-Constructor类

| 方法名                  | 返回内容                                                     | 优点                                         | 缺点                               |
| ----------------------- | ------------------------------------------------------------ | -------------------------------------------- | ---------------------------------- |
| getConstructor          | 返回指定参数类型的public构造函数                             | 安全性较高                                   | 无法获取private和protected构造函数 |
| getConstructors         | 返回类中的所有public构造函数                                 | 使用简单便捷                                 | 无法获取非public构造函数           |
| getDeclaredConstructor  | 返回指定参数类型的所有构造函数（包括public、protected、default、private） | 可以获取所有声明的构造函数，访问范围最全面   | 使用不当可能造成安全问题           |
| getDeclaredConstructors | 返回类中的所有构造函数（包括public、protected、default、private） | 可以获取所有声明的构造函数，最全面的获取方法 | 使用不当可能造成安全问题           |


#### 反射执行步骤

1. 先获取想要操作的类的 Class 对象。
2. 用 Class 类中的方法，获得我们需要的Method、Field类对象。
3. 借助反射API，操作这些对象，完成定制。

#### 反射使用场景

- AOP
- SpringBean
- MyBatis

