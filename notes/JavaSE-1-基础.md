[[Java]]

# SE

## 基础

### 数据

#### 变量类型

|          |    成员变量    |       局部变量       |          静态变量           |
| :------: | :------------: | :------------------: | :-------------------------: |
| 定义位置 | 在类中，方法外 | 方法中或者方法的形参 |       在类中，方法外        |
| 初始化值 | 有默认初始化值 |  无，赋值后才能使用  |       有默认初始化值        |
| 调用方法 |    对象调用    |                      |     对象调用，类名调用      |
| 存储位置 |      堆中      |         栈中         | 方法区（JDK8 以后移到堆中） |
| 生命周期 |  与对象共存亡  |     与方法共存亡     |         与类共存亡          |
|   别名   |    实例变量    |                      |    类变量，静态成员变量     |

静态变量只有一个，成员变量是类中的变量，局部变量是方法中的变量



初学时笔记内容参考视频：https://www.bilibili.com/video/BV1TE41177mP，随着学习的深入又增加很多知识





***



#### 数据类型

##### 基本类型

Java 语言提供了八种基本类型。六种数字类型（四个整数型，两个浮点型），一种字符类型，还有一种布尔型

**byte：**

- byte 数据类型是 8 位、有符号的，以二进制补码表示的整数，**8 位一个字节**，首位是符号位
- 最小值是 -128（-2^7）、最大值是 127（2^7-1）
- 默认值是 `0`
- byte 类型用在大型数组中节约空间，主要代替整数，byte 变量占用的空间只有 int 类型的四分之一
- 例子：`byte a = 100，byte b = -50`

**short：**

- short 数据类型是 16 位、有符号的以二进制补码表示的整数
- 最小值是 -32768（-2^15）、最大值是 32767（2^15 - 1）
- short 数据类型也可以像 byte 那样节省空间，一个 short 变量是 int 型变量所占空间的二分之一
- 默认值是 `0`
- 例子：`short s = 1000，short r = -20000`

**int：**

- int 数据类型是 32 位 4 字节、有符号的以二进制补码表示的整数
- 最小值是 -2,147,483,648（-2^31）、最大值是 2,147,483,647（2^31 - 1）
- 一般地整型变量默认为 int 类型
- 默认值是 `0`
- 例子：`int a = 100000, int b = -200000`

**long：**

- long 数据类型是 64 位 8 字节、有符号的以二进制补码表示的整数
- 最小值是 -9,223,372,036,854,775,808（-2^63）、最大值是 9,223,372,036,854,775,807（2^63 -1）
- 这种类型主要使用在需要比较大整数的系统上
- 默认值是 ` 0L`
- 例子： `long a = 100000L，Long b = -200000L`，L 理论上不分大小写，但是若写成 I 容易与数字 1 混淆，不容易分辩

**float：**

- float 数据类型是单精度、32 位、符合 IEEE 754 标准的浮点数
- float 在储存大型浮点数组的时候可节省内存空间
- 默认值是 `0.0f`
- 浮点数不能用来表示精确的值，如货币
- 例子：`float f1 = 234.5F`

**double：**

- double 数据类型是双精度、64 位、符合 IEEE 754 标准的浮点数
- 浮点数的默认类型为 double 类型
- double 类型同样不能表示精确的值，如货币
- 默认值是 `0.0d`
- 例子：`double d1 = 123.4`

**boolean：**

- boolean 数据类型表示一位的信息
- 只有两个取值：true 和 false
- JVM 规范指出 boolean 当做 int 处理，boolean 数组当做 byte 数组处理，这样可以得出 boolean 类型单独使用占了 4 个字节，在数组中是 1 个字节
- 默认值是 `false`
- 例子：`boolean one = true`

**char：**

- char 类型是一个单一的 16 位**两个字节**的 Unicode 字符
- 最小值是 `\u0000`（即为 0）
- 最大值是 `\uffff`（即为 65535）
- char 数据类型可以**存储任何字符**
- 例子：`char c = 'A'`，`char c = '张'`



****



##### 上下转型

* float 与 double：

  Java 不能隐式执行**向下转型**，因为这会使得精度降低，但是可以向上转型

  ```java
  //1.1字面量属于double类型，不能直接将1.1直接赋值给 float 变量，因为这是向下转型
  float f = 1.1;//报错
  //1.1f 字面量才是 float 类型
  float f = 1.1f;
  ```

  ```java
  float f1 = 1.234f;
  double d1 = f1;
  
  double d2 = 1.23;
  float f2 = (float) d2;//向下转型需要强转
  ```

  ```java
  int i1 = 1245;
  long l1 = i1;
  
  long l2 = 1234;
  int i2 = (int) l2;
  ```

* 隐式类型转换：

  字面量 1 是 int 类型，比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型

  使用 += 或者 ++ 运算符会执行类型转换：

  ```java
  short s1 = 1;
  s1 += 1;	//s1++;
  //上面的语句相当于将 s1 + 1 的计算结果进行了向下转型
  s1 = (short) (s1 + 1);
  ```
  
  



***



##### 引用类型

引用数据类型：类，接口，数组都是引用数据类型，又叫包装类

包装类的作用：

* 包装类作为类首先拥有了 Object 类的方法
* 包装类作为引用类型的变量可以**存储 null 值**


```java
基本数据类型                包装类（引用数据类型）
byte                      Byte
short                     Short
int                       Integer
long                      Long

float                     Float
double                    Double
char                      Character
boolean                   Boolean
```
Java 为包装类做了一些特殊功能，具体来看特殊功能主要有：

* 可以把基本数据类型的值转换成字符串类型的值
  1. 调用 toString() 方法
  2. 调用 Integer.toString(基本数据类型的值) 得到字符串
  3. 直接把基本数据类型 + 空字符串就得到了字符串（推荐使用）

* 把字符串类型的数值转换成对应的基本数据类型的值（**重要**）

  1. Xxx.parseXxx("字符串类型的数值") → `Integer.parseInt(numStr)`
  2. Xxx.valueOf("字符串类型的数值")   → `Integer.valueOf(numStr)` （推荐使用）

  ```java
  public class PackageClass02 {
      public static void main(String[] args) {
          // 1.把基本数据类型的值转成字符串
          Integer it = 100 ;
          // a.调用toString()方法。
          String itStr = it.toString();
          System.out.println(itStr+1);//1001
          // b.调用Integer.toString(基本数据类型的值)得到字符串。
          String itStr1 = Integer.toString(it);
          System.out.println(itStr1+1);//1001
          // c.直接把基本数据类型+空字符串就得到了字符串。
          String itStr2 = it + "";
          System.out.println(itStr2+1);// 1001
  
          // 2.把字符串类型的数值转换成对应的基本数据类型的值
          String numStr = "23";
          int numInt = Integer.valueOf(numStr);
          System.out.println(numInt+1);//24
  
          String doubleStr = "99.9";
          double doubleDb = Double.valueOf(doubleStr);
          System.out.println(doubleDb+0.1);//100.0
      }
  }
  ```




***



##### 类型对比

* 有了基本数据类型，为什么还要引用数据类型？

  > 引用数据类型封装了数据和处理该数据的方法，比如 Integer.parseInt(String) 就是将 String 字符类型数据转换为 Integer 整型
  >
  > Java 中大部分类和方法都是针对引用数据类型，包括泛型和集合

* 引用数据类型那么好，为什么还用基本数据类型？

  > 引用类型的对象要多储存对象头，对基本数据类型来说空间浪费率太高。逻辑上来讲，Java 只有包装类就够了，为了运行速度，需要用到基本数据类型；优先考虑运行效率的问题，所以二者同时存在是合乎情理的

* Java 集合不能存放基本数据类型，只存放对象的引用？

  > 不能放基本数据类型是因为不是 Object 的子类。泛型思想，如果不用泛型要写很多参数类型不同的但功能相同的函数（方法重载）

* ==

  > == 比较基本数据类型：比较的是具体的值
  > == 比较引用数据类型：比较的是对象地址值



***



#### 装箱拆箱

**自动装箱**：可以直接把基本数据类型的值或者变量赋值给包装类

**自动拆箱**：可以把包装类的变量直接赋值给基本数据类型

```java
public class PackegeClass {
    public static void main(String[] args) {
        int a = 12 ;
        Integer a1 = 12 ;  // 自动装箱
        Integer a2 = a ;   // 自动装箱
        Integer a3 = null; // 引用数据类型的默认值可以为null

        Integer c = 100 ;
        int c1 = c ;      // 自动拆箱

        Integer it = Integer.valueOf(12);  	// 手工装箱！
        // Integer it1 = new Integer(12); 	// 手工装箱！
        Integer it2 = 12;

        Integer it3 = 111 ;
        int it33 = it3.intValue(); // 手工拆箱
    }
}
```

**自动装箱**反编译后底层调用 `Integer.valueOf()` 实现，源码：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        // 【缓存池】，本质上是一个数组
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

自动拆箱调用 `java.lang.Integer#intValue`，源码：

```java
public int intValue() {
    return value;
}
```



***



#### 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123)：每次都会新建一个对象

- Integer.valueOf(123)：会使用缓存池中的对象，多次调用取得同一个对象的引用

  ```java
  Integer x = new Integer(123);
  Integer y = new Integer(123);
  System.out.println(x == y);    // false
  Integer z = Integer.valueOf(123);
  Integer k = Integer.valueOf(123);
  System.out.println(z == k);   // true
  ```

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象

**基本类型对应的缓存池如下：**

- Boolean values true and false
- all byte values
- Short values between -128 and 127
- Long values between -128 and 127
- Integer values between -128 and 127
- Character in the range \u0000 to \u007F (0 and 127)

在 jdk 1.8 所有的数值类缓冲池中，**Integer 的缓存池 IntegerCache 很特殊，这个缓冲池的下界是 -128，上界默认是 127**，但是上界是可调的，在启动 JVM 时通过 `AutoBoxCacheMax=<size>` 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.Integer.IntegerCache 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界

```java
Integer x = 100;				// 自动装箱，底层调用 Integer.valueOf(1)
Integer y = 100;
System.out.println(x == y);   	// true

Integer x = 1000;
Integer y = 1000;
System.out.println(x == y);   	// false，因为缓存池最大127

int x = 1000;
Integer y = 1000;
System.out.println(x == y);		// true，因为 y 会调用 intValue 【自动拆箱】返回 int 原始值进行比较
```



***



#### 输入数据

语法：`Scanner sc = new Scanner(System.in)`

* next()：遇到了空格，就不再录入数据了，结束标记：空格、tab 键
* nextLine()：可以将数据完整的接收过来，结束标记：回车换行符

一般使用 `sc.nextInt()` 或者 `sc.nextLine()` 接受整型和字符串，然后转成需要的数据类型

* Scanner：`BufferedReader br = new BufferedReader(new InputStreamReader(System.in))`
* print：`PrintStream.write()`

> 使用引用数据类型的API

```java
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    while (sc.hasNextLine()) {
        String msg = sc.nextLine();
    }
}
```





****



### 数组

#### 初始化

数组就是存储数据长度固定的容器，存储多个数据的数据类型要一致，**数组也是一个对象**

创建数组：

* 数据类型[] 数组名：`int[] arr`  （常用）
* 数据类型 数组名[]：`int arr[]`

静态初始化：

* 数据类型[] 数组名 = new 数据类型[]{元素1,元素2,...}：`int[] arr = new int[]{11,22,33}`
* 数据类型[] 数组名 = {元素1,元素2,...}：`int[] arr = {44,55,66}`

动态初始化

* 数据类型[] 数组名 = new 数据类型[数组长度]：`int[] arr = new int[3]`



#### 元素访问

* **索引**：每一个存储到数组的元素，都会自动的拥有一个编号，从 **0** 开始。这个自动编号称为数组索引（index），可以通过数组的索引访问到数组中的元素

* **访问格式**：数组名[索引]，`arr[0]`
* **赋值：**`arr[0] = 10`



***



#### 内存分配

内存是计算机中的重要器件，临时存储区域，作用是运行程序。编写的程序是存放在硬盘中，在硬盘中的程序是不会运行的，必须放进内存中才能运行，运行完毕后会清空内存，Java 虚拟机要运行程序，必须要对内存进行空间的分配和管理

| 区域名称   | 作用                                                       |
| ---------- | ---------------------------------------------------------- |
| 寄存器     | 给 CPU 使用                                                |
| 本地方法栈 | JVM 在使用操作系统功能的时候使用                           |
| 方法区     | 存储可以运行的 class 文件                                  |
| 堆内存     | 存储对象或者数组，new 来创建的，都存储在堆内存             |
| 方法栈     | 方法运行时使用的内存，比如 main 方法运行，进入方法栈中执行 |

内存分配图：**Java 数组分配在堆内存**

* 一个数组内存图

  ![](JavaSE-1-基础.assets/数组内存分配-一个数组内存图.png)

* 两个数组内存图

  ![](JavaSE-1-基础.assets/数组内存分配-两个数组内存图.png)

* 多个数组指向相同内存图

  ![](JavaSE-1-基础.assets/数组内存分配-多个数组指向一个数组内存图.png)

***



#### 数组异常

* 索引越界异常：ArrayIndexOutOfBoundsException 

* 空指针异常：NullPointerException 

  ```java
  public class ArrayDemo {
      public static void main(String[] args) {
          int[] arr = new int[3];
          //把null赋值给数组
          arr = null;
          System.out.println(arr[0]);
      }
  }
  ```

  arr = null，表示变量 arr 将不再保存数组的内存地址，也就不允许再操作数组，因此运行的时候会抛出空指针异常。在开发中，空指针异常是不能出现的，一旦出现了，就必须要修改编写的代码

  解决方案：给数组一个真正的堆内存空间引用即可
  
  

***



#### 二维数组

二维数组也是一种容器，不同于一维数组，该容器存储的都是一维数组容器

初始化：

* 动态初始化：数据类型[][] 变量名 = new 数据类型[m] [n]，`int[][] arr = new int[3][3]`
  
  * m 表示这个二维数组，可以存放多少个一维数组，行
  * n 表示每一个一维数组，可以存放多少个元素，列
* 静态初始化
  * 数据类型[][] 变量名 = new 数据类型 [][]{{元素1, 元素2...} , {元素1, 元素2...} 
  * 数据类型[][] 变量名 = {{元素1, 元素2...}, {元素1, 元素2...}...}
  * `int[][] arr = {{11,22,33}, {44,55,66}}`

遍历：

```java
public class Test1 {
    /*
        步骤:
            1. 遍历二维数组，取出里面每一个一维数组
            2. 在遍历的过程中，对每一个一维数组继续完成遍历，获取内部存储的每一个元素
     */
    public static void main(String[] args) {
        int[][] arr = {{11, 22, 33}, {33, 44, 55}};
        // 1. 遍历二维数组，取出里面每一个一维数组
        for (int i = 0; i < arr.length; i++) {
            //System.out.println(arr[i]);
            // 2. 在遍历的过程中，对每一个一维数组继续完成遍历，获取内部存储的每一个元素
            //int[] temp = arr[i];
            for (int j = 0; j < arr[i].length; j++) {
                System.out.println(arr[i][j]);
            }
        }
    }
}
```





****



### 运算

* i++ 与 ++i 的区别？
  
  i++ 表示先将 i 放在表达式中运算，然后再加 1，++i 表示先将 i 加 1，然后再放在表达式中运算

* || 和 |，&& 和& 的区别，逻辑运算符

  **& 和| 称为布尔运算符，位运算符；&& 和 || 称为条件布尔运算符，也叫短路运算符**

  如果 && 运算符的第一个操作数是 false，就不需要考虑第二个操作数的值了，因为无论第二个操作数的值是什么，其结果都是 false；同样，如果第一个操作数是 true，|| 运算符就返回 true，无需考虑第二个操作数的值；但 & 和 | 却不是这样，它们总是要计算两个操作数。为了提高性能，**尽可能使用 && 和 || 运算符**

* 异或 ^：两位相异为 1，相同为 0，又叫不进位加法

* 同或：两位相同为 1，相异为 0

* switch：从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象

  ```java
  String s = "a";
  switch (s) {
      case "a":
          System.out.println("aaa");
          break;
      case "b":
          System.out.println("bbb");
          break;
      default:
          break;
  }
  ```
  
  switch 不支持 long、float、double，switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么用 if 比较合适
  
* break：跳出一层循环

* 移位运算：计算机里一般用**补码表示数字**，正数、负数的表示区别就是最高位是 0 还是 1
  
  * 正数的原码反码补码相同，最高位为 0
  
    ```java
    100:	00000000  00000000  00000000  01100100
    ```
  
  * 负数：
    原码：最高位为 1，其余位置和正数相同
    反码：保证符号位不变，其余位置取反
    补码：保证符号位不变，其余位置取反后加 1，即反码 +1
  
    ```java
    -100 原码:	10000000  00000000  00000000  01100100	//32位
    -100 反码:	11111111  11111111  11111111  10011011
    -100 补码:	11111111  11111111  11111111  10011100
    ```
  
    补码 → 原码：符号位不变，其余位置取反加 1
  
  运算符：
  
  * `>>` 运算符：将二进制位进行右移操作，相当于除 2
  * `<<` 运算符：将二进制位进行左移操作，相当于乘 2
  * `>>>` 运算符：无符号右移，忽略符号位，空位都以 0 补齐
  
  运算规则：
  
  * 正数的左移与右移，空位补 0
  
  * 负数原码的左移与右移，空位补 0
    
    负数反码的左移与右移，空位补 1
    
    负数补码，左移低位补 0（会导致负数变为正数的问题，因为移动了符号位），右移高位补 1
    
  * 无符号移位，空位补 0



****



### 参数

#### 形参实参

形参：

* 形式参数，用于定义方法的时候使用的参数，只能是变量
* 形参只有在方法被调用的时候，虚拟机才分配内存单元，方法调用结束之后便会释放所分配的内存单元

实参：调用方法时传递的数据可以是常量，也可以是变量



#### 可变参数

可变参数用在形参中可以接收多个数据，在方法内部**本质上就是一个数组**

格式：数据类型... 参数名称

作用：传输参数非常灵活，可以不传输参数、传输一个参数、或者传输一个数组

可变参数的注意事项：

* 一个形参列表中可变参数只能有一个
* 可变参数必须放在形参列表的**最后面**

```java
public static void main(String[] args) {
	sum(); // 可以不传输参数。
	sum(10); // 可以传输一个参数。
	sum(10,20,30); // 可以传输多个参数。
	sum(new int[]{10,30,50,70,90}); // 可以传输一个数组。
}

public static void sum(int... nums){
	int sum = 0;
	for(int i : a) {
		sum += i;
	}
	return sum;
}
```



***



### 方法

#### 方法概述

方法（method）是将具有独立功能的代码块组织成为一个整体，使其具有特殊功能的代码集

注意：方法必须先创建才可以使用，该过程成为方法定义，方法创建后并不是直接可以运行的，需要手动使用后才执行，该过程成为方法调用

在方法内部定义的叫局部变量，局部变量不能加 static，包括 protected、private、public 这些也不能加

原因：局部变量是保存在栈中的，而静态变量保存于方法区（JDK8 在堆中），局部变量出了方法就被栈回收了，而静态变量不会，所以**在局部变量前不能加 static 关键字**，静态变量是定义在类中，又叫类变量



***



#### 定义调用

定义格式：

```java
public static 返回值类型 方法名(参数) {
	//方法体;
	return 数据 ;
}
```

调用格式：

```java
数据类型 变量名 = 方法名 (参数) ;
```

* 方法名：调用方法时候使用的标识
* 参数：由数据类型和变量名组成，多个参数之间用逗号隔开
* 方法体：完成功能的代码块
* return：如果方法操作完毕，有数据返回，用于把数据返回给调用者

如果方法操作完毕

* void 类型的方法，直接调用即可，而且方法体中一般不写 return
* 非 void 类型的方法，推荐用变量接收调用

原理：每个方法在被调用执行的时候，都会进入栈内存，并且拥有自己独立的内存空间，方法内部代码调用完毕之后，会从栈内存中弹栈消失



***



#### 注意事项

* 方法不能嵌套定义

  ```java
  public class MethodDemo {
  	public static void main(String[] args) {
  	}
  	public static void methodOne() {
  		public static void methodTwo() {
  			// 这里会引发编译错误!!!
  		}
  	}
  }
  ```

* void 表示无返回值，可以省略 return，也可以单独的书写 return，后面不加数据

  ```java
  public static void methodTwo() {
  	//return 100; 编译错误，因为没有具体返回值类型
  	return;
  	//System.out.println(100); return语句后面不能跟数据或代码
  }
  ```

  

***



#### 方法重载

##### 重载介绍

方法重载指同一个类中定义的多个方法之间的关系，满足下列条件的多个方法相互构成重载：

1. 多个方法在**同一个类**中
2. 多个方法具有**相同的方法名**
3. 多个方法的**参数不相同**，类型不同或者数量不同

重载仅对应方法的定义，与方法的调用无关，调用方式参照标准格式

重载仅针对同一个类中方法的名称与参数进行识别，与返回值无关，**不能通过返回值来判定两个方法是否构成重载**

原理：JVM → 运行机制 → 方法调用 → 多态原理

```java
public class MethodDemo {
	public static void fn(int a) {
		//方法体
	}
    
	public static int fn(int a) { /*错误原因：重载与返回值无关*/
		//方法体
	}
    
    public static void fn(int a, int b) {/*正确格式*/
		//方法体
	}
}
```



***



##### 方法选取

重载的方法在编译过程中即可完成识别，方法调用时 Java 编译器会根据所传入参数的声明类型（注意与实际类型区分）来选取重载方法。选取的过程共分为三个阶段：

* 一阶段：在不考虑对基本类型自动装拆箱 (auto-boxing，auto-unboxing)，以及可变长参数的情况下选取重载方法
* 二阶段：如果第一阶段中没有找到适配的方法，那么在允许自动装拆箱，但不允许可变长参数的情况下选取重载方法
* 三阶段：如果第二阶段中没有找到适配的方法，那么在允许自动装拆箱以及可变长参数的情况下选取重载方法

如果 Java 编译器在同一个阶段中找到了多个适配的方法，那么会选择一个最为贴切的，而决定贴切程度的一个关键就是形式参数类型的继承关系，**一般会选择形参为参数类型的子类的方法，因为子类时更具体的实现**：

```java
public class MethodDemo {
    void invoke(Object obj, Object... args) { ... }
    void invoke(String s, Object obj, Object... args) { ... }

    invoke(null, 1); 	// 调用第二个invoke方法，选取的第二阶段
    invoke(null, 1, 2); // 调用第二个invoke方法，匹配第一个和第二个，但String是Object的子类
    
    invoke(null, new Object[]{1}); // 只有手动绕开可变长参数的语法糖，才能调用第一个invoke方法
    							   // 可变参数底层是数组，JVM->运行机制->代码优化
}
```

因此不提倡可变长参数方法的重载



***



##### 继承重载

除了同一个类中的方法，重载也可以作用于这个类所继承而来的方法。如果子类定义了与父类中**非私有方法**同名的方法，而且这两个方法的参数类型不同，那么在子类中，这两个方法同样构成了重载

* 如果这两个方法都是静态的，那么子类中的方法隐藏了父类中的方法
* 如果这两个方法都不是静态的，且都不是私有的，那么子类的方法重写了父类中的方法，也就是**多态**



***



#### 参数传递

Java 的参数是以**值传递**的形式传入方法中

值传递和引用传递的区别在于传递后会不会影响实参的值：**值传递会创建副本**，引用传递不会创建副本

* 基本数据类型：形式参数的改变，不影响实际参数
  
  每个方法在栈内存中，都会有独立的栈空间，方法运行结束后就会弹栈消失
  
  ```java
  public class ArgsDemo01 {
  	public static void main(String[] args) {
  		int number = 100;
  		System.out.println("调用change方法前：" + number);//100
  		change(number);
  		System.out.println("调用change方法后：" + number);//100
  	}
  	public static void change(int number) {
  		number = 200;
  	}
  }
  ```
  
* 引用类型：形式参数的改变，影响实际参数的值
  
  **引用数据类型的传参，本质上是将对象的地址以值的方式传递到形参中**，内存中会造成两个引用指向同一个内存的效果，所以即使方法弹栈，堆内存中的数据也已经是改变后的结果
  
  ```java
  public class PassByValueExample {
      public static void main(String[] args) {
          Dog dog = new Dog("A");
          func(dog);
          System.out.println(dog.getName());	// B
      }
      private static void func(Dog dog) {
          dog.setName("B");
      }
  }
  class Dog {
      String name;//.....
  }
  ```
  
  



***



### 枚举

枚举是 Java 中的一种特殊类型，为了做信息的标志和信息的分类

定义枚举的格式：

```java
修饰符 enum 枚举名称{
	第一行都是罗列枚举实例的名称。
}
```

枚举的特点：

* 枚举类是用 final 修饰的，枚举类不能被继承
* 枚举类默认继承了 java.lang.Enum 枚举类
* 枚举类的第一行都是常量，必须是罗列枚举类的实例名称
* 枚举类相当于是多例设计模式
* 每个枚举项都是一个实例，是一个静态成员变量

| 方法名                                            | 说明                                 |
| ------------------------------------------------- | ------------------------------------ |
| String name()                                     | 获取枚举项的名称                     |
| int ordinal()                                     | 返回枚举项在枚举类中的索引值         |
| int compareTo(E  o)                               | 比较两个枚举项，返回的是索引值的差值 |
| String toString()                                 | 返回枚举常量的名称                   |
| static <T> T  valueOf(Class<T> type,String  name) | 获取指定枚举类中的指定名称的枚举值   |
| values()                                          | 获得所有的枚举项                     |

* 源码分析：

  ```java
  enum Season {
      SPRING , SUMMER , AUTUMN , WINTER;
  }
  // 枚举类的编译以后源代码：
  public final class Season extends java.lang.Enum<Season> {
  	public static final Season SPRING = new Season();
  	public static final Season SUMMER = new Season();
  	public static final Season AUTUMN = new Season();
  	public static final Season WINTER = new Season();
  
  	public static Season[] values();
  	public static Season valueOf(java.lang.String);
  }
  ```

* API 使用

  ```java
  public class EnumDemo {
      public static void main(String[] args){
          // 获取索引
          Season s = Season.SPRING;
          System.out.println(s);	//SPRING
          System.out.println(s.ordinal()); // 0，该值代表索引，summer 就是 1
          s.s.doSomething();
          // 获取全部枚举
          Season[] ss = Season.values();
          for(int i = 0; i < ss.length; i++){
              System.out.println(ss[i]);
          }
          
          int result = Season.SPRING.compareTo(Season.WINTER);
          System.out.println(result);//-3
      }
  }
  enum Season {
      SPRING , SUMMER , AUTUMN , WINTER;
      
      public void doSomething() {
          System.out.println("hello ");
      }
  }
  ```






***



### Debug

Debug 是供程序员使用的程序调试工具，它可以用于查看程序的执行流程，也可以用于追踪程序执行过程来调试程序。

加断点 → Debug 运行 → 单步运行 → 看 Debugger 窗口 → 看 Console 窗口

![](JavaSE-1-基础.assets/Debug按键说明.png)

<img src="JavaSE-1-基础.assets/Debug条件断点.png" alt="Debug条件断点" style="zoom:50%;" />






