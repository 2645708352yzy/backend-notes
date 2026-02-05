[[Java]]

# SE

## API

### Object

#### 基本介绍

Object 类是 Java 中的祖宗类，一个类或者默认继承 Object 类，或者间接继承 Object 类，Object 类的方法是一切子类都可以直接使用

Object 类常用方法：

* `public String toString()`：默认是返回当前对象在堆内存中的地址信息：类的全限名@内存地址，例：Student@735b478；
  * 直接输出对象名称，默认会调用 toString() 方法，所以省略 toString() 不写；
  * 如果输出对象的内容，需要重写 toString() 方法，toString 方法存在的意义是为了被子类重写
* `public boolean equals(Object o)`：默认是比较两个对象的引用是否相同
* `protected Object clone()`：创建并返回此对象的副本 

只要两个对象的内容一样，就认为是相等的：

```java
public boolean equals(Object o) {
	// 1.判断是否自己和自己比较，如果是同一个对象比较直接返回true
	if (this == o) return true;
	// 2.判断被比较者是否为null ,以及是否是学生类型。
	if (o == null || this.getClass() != o.getClass()) return false;
	// 3.o一定是学生类型，强制转换成学生，开始比较内容！
	Student student = (Student) o;
	return age == student.age &&
           sex == student.sex &&
           Objects.equals(name, student.name);
}
```

**面试题**：== 和 equals 的区别

* == 比较的是变量（栈）内存中存放的对象的（堆）内存地址，用来判断两个对象的**地址**是否相同，即是否是指相同一个对象，比较的是真正意义上的指针操作
* Object 类中的方法，**默认比较两个对象的引用**，重写 equals 方法比较的是两个对象的**内容**是否相等，所有的类都是继承自 java.lang.Object 类，所以适用于所有对象

hashCode 的作用：

* hashCode 的存在主要是用于查找的快捷性，如 Hashtable，HashMap 等，可以在散列存储结构中确定对象的存储地址
* 如果两个对象相同，就是适用于 equals(java.lang.Object) 方法，那么这两个对象的 hashCode 一定要相同
* 哈希值相同的数据不一定内容相同，内容相同的数据哈希值一定相同



***



#### 深浅克隆

Object 的 clone() 是 protected 方法，一个类不显式去重写 clone()，就不能直接去调用该类实例的 clone() 方法

深浅拷贝（克隆）的概念：

* 浅拷贝 (shallowCopy)：**对基本数据类型进行值传递，对引用数据类型只是复制了引用**，被复制对象属性的所有的引用仍然指向原来的对象，简而言之就是增加了一个指针指向原来对象的内存地址

  **Java 中的复制方法基本都是浅拷贝**：Object.clone()、System.arraycopy()、Arrays.copyOf()

* 深拷贝 (deepCopy)：对基本数据类型进行值传递，对引用数据类型是一个整个独立的对象拷贝，会拷贝所有的属性并指向的动态分配的内存，简而言之就是把所有属性复制到一个新的内存，增加一个指针指向新内存。所以使用深拷贝的情况下，释放内存的时候不会出现使用浅拷贝时释放同一块内存的错误

Cloneable 接口是一个标识性接口，即该接口不包含任何方法（包括 clone），但是如果一个类想合法的进行克隆，那么就必须实现这个接口，在使用 clone() 方法时，若该类未实现 Cloneable 接口，则抛出异常

* Clone & Copy：`Student s = new Student`

  `Student s1 = s`：只是 copy 了一下 reference，s 和 s1 指向内存中同一个 Object，对对象的修改会影响对方

  `Student s2 = s.clone()`：会生成一个新的 Student 对象，并且和 s 具有相同的属性值和方法

* Shallow Clone & Deep Clone：
  
  浅克隆：Object 中的 clone() 方法在对某个对象克隆时对其仅仅是简单地执行域对域的 copy
  
  * 对基本数据类型和包装类的克隆是没有问题的。String、Integer 等包装类型在内存中是**不可以被改变的对象**，所以在使用克隆时可以视为基本类型，只需浅克隆引用即可
  * 如果对一个引用类型进行克隆时只是克隆了它的引用，和原始对象共享对象成员变量

  ![](JavaSE-3-API.assets/Object浅克隆.jpg)

  深克隆：在对整个对象浅克隆后，对其引用变量进行克隆，并将其更新到浅克隆对象中去

  ```java
  public class Student  implements Cloneable{
      private String name;
      private Integer age;
      private Date date;
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          Student s = (Student) super.clone();
          s.date = (Date) date.clone();
          return s;
      }
      //.....
  }
  ```

SDP → 创建型 → 原型模式



***



### Objects

Objects 类与 Object 是继承关系

Objects 的方法：

* `public static boolean equals(Object a, Object b)`：比较两个对象是否相同
  
    ```java
  public static boolean equals(Object a, Object b) {
      // 进行非空判断，从而可以避免空指针异常
      return a == b || a != null && a.equals(b);
  }
  ```

* `public static boolean isNull(Object obj)`：判断变量是否为 null ，为 null 返回 true

* `public static String toString(对象)`：返回参数中对象的字符串表示形式

* `public static String toString(对象, 默认字符串)`：返回对象的字符串表示形式

```java
public class ObjectsDemo {
    public static void main(String[] args) {
        Student s1 = null;
        Student s2 = new Student();
        System.out.println(Objects.equals(s1 , s2));//推荐使用
        // System.out.println(s1.equals(s2)); // 空指针异常
 
        System.out.println(Objects.isNull(s1));
        System.out.println(s1 == null);//直接判断比较好
    }
}

public class Student {
}
```



***



### String

#### 基本介绍

String 被声明为 final，因此不可被继承 **（Integer 等包装类也不能被继承）**

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
 	/** The value is used for character storage. */
    private final char value[];
    /** Cache the hash code for the string */
    private int hash; // Default to 0
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组，并且 String 内部没有改变 value 数组的方法，因此可以**保证 String 不可变，也保证线程安全**

注意：不能改变的意思是**每次更改字符串都会产生新的对象**，并不是对原始对象进行改变

```java
String s = "abc";
s = s + "cd"; //s = abccd 新对象
```



****



#### 常用方法

常用 API：

* `public boolean equals(String s)`：比较两个字符串内容是否相同、区分大小写

* `public boolean equalsIgnoreCase(String anotherString)`：比较字符串的内容，忽略大小写
* `public int length()`：返回此字符串的长度
* `public String trim()`：返回一个字符串，其值为此字符串，并删除任何前导和尾随空格
* `public String[] split(String regex)`：将字符串按给定的正则表达式分割成字符串数组
* `public char charAt(int index)`：取索引处的值
* `public char[] toCharArray()`：将字符串拆分为字符数组后返回
* `public boolean startsWith(String prefix)`：测试此字符串是否以指定的前缀开头
* `public int indexOf(String str)`：返回指定子字符串第一次出现的字符串内的索引，没有返回 -1
* `public int lastIndexOf(String str)`：返回字符串最后一次出现 str 的索引，没有返回 -1
* `public String substring(int beginIndex)`：返回子字符串，以原字符串指定索引处到结尾
* `public String substring(int i, int j)`：指定索引处扩展到 j - 1 的位置，字符串长度为 j - i
* `public String toLowerCase()`：将此 String 所有字符转换为小写，使用默认语言环境的规则
* `public String toUpperCase()`：使用默认语言环境的规则将此 String 所有字符转换为大写
* `public String replace(CharSequence target, CharSequence replacement)`：使用新值，将字符串中的旧值替换，得到新的字符串

```java
String s = 123-78;
s.replace("-","");//12378
```



***



#### 构造方式

构造方法：

* `public String()`：创建一个空白字符串对象，不含有任何内容
* `public String(char[] chs)`：根据字符数组的内容，来创建字符串对象
* `public String(String original)`：根据传入的字符串内容，来创建字符串对象

直接赋值：`String s = "abc"` 直接赋值的方式创建字符串对象，内容就是 abc

- 通过构造方法创建：通过 new 创建的字符串对象，每一次 new 都会申请一个内存空间，虽然内容相同，但是地址值不同，**返回堆内存中对象的引用**
- 直接赋值方式创建：以 `" "` 方式给出的字符串，只要字符序列相同（顺序和大小写），无论在程序代码中出现几次，JVM 都只会**在 String Pool 中创建一个字符串对象**，并在字符串池中维护

`String str = new String("abc")` 创建字符串对象：

* 创建一个对象：字符串池中已经存在 abc 对象，那么直接在创建一个对象放入堆中，返回堆内引用
* 创建两个对象：字符串池中未找到 abc 对象，那么分别在堆中和字符串池中创建一个对象，字符串池中的比较都是采用 equals() 
  <img src="JavaSE-3-API.assets/String构造方法字节码.png" style="zoom: 67%;" />

`new String("a") + new String("b")` 创建字符串对象：

* 对象 1：new StringBuilder()

* 对象 2：new String("a")、对象 3：常量池中的 a

* 对象 4：new String("b")、对象 5：常量池中的 b
  <img src="JavaSE-3-API.assets/String拼接方法字节码.png" style="zoom:67%;" />

*  StringBuilder 的 toString()：

  ```java
  @Override
  public String toString() {
      return new String(value, 0, count);
  }
  ```

  * 对象 6：new String("ab")
  * StringBuilder 的 toString() 调用，**在字符串常量池中没有生成 ab**，new String("ab") 会创建两个对象因为传参数的时候使用字面量创建了对象 ab，当使用数组构造 String 对象时，没有加入常量池的操作



***



#### String Pool

##### 基本介绍

字符串常量池（String Pool / StringTable / 串池）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定，常量池类似于 Java 系统级别提供的**缓存**，存放对象和引用

* StringTable，类似 HashTable 结构，通过 `-XX:StringTableSize` 设置大小，JDK 1.8 中默认 60013
* 常量池中的字符串仅是符号，第一次使用时才变为对象，可以避免重复创建字符串对象
* 字符串**变量**的拼接的原理是 StringBuilder#append，append 方法比字符串拼接效率高（JDK 1.8）
* 字符串**常量**拼接的原理是编译期优化，拼接结果放入常量池
* 可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中



***



##### intern()

JDK 1.8：当一个字符串调用 intern() 方法时，如果 String Pool 中：
* 存在一个字符串和该字符串值相等，就会返回 String Pool 中字符串的引用（需要变量接收）
* 不存在，会把对象的**引用地址**复制一份放入串池，并返回串池中的引用地址，前提是堆内存有该对象，因为 Pool 在堆中，为了节省内存不再创建新对象

JDK 1.6：将这个字符串对象尝试放入串池，如果有就不放入，返回已有的串池中的对象的引用；如果没有会把此对象复制一份，放入串池，把串池中的对象返回

```java
public class Demo {
    // 常量池中的信息都加载到运行时常量池，这时a b ab是常量池中的符号，还不是java字符串对象，是懒惰的
    // ldc #2 会把 a 符号变为 "a" 字符串对象     ldc:反编译后的指令
    // ldc #3 会把 b 符号变为 "b" 字符串对象
    // ldc #4 会把 ab 符号变为 "ab" 字符串对象
    public static void main(String[] args) {
        String s1 = "a"; 	// 懒惰的
        String s2 = "b";
        String s3 = "ab";	// 串池
        
        String s4 = s1 + s2;	// 返回的是堆内地址
        // 原理：new StringBuilder().append("a").append("b").toString()  new String("ab")
        
        String s5 = "a" + "b";  // javac 在编译期间的优化，结果已经在编译期确定为ab

        System.out.println(s3 == s4); // false
        System.out.println(s3 == s5); // true

        String x2 = new String("c") + new String("d"); // new String("cd")
        // 虽然 new，但是在字符串常量池没有 cd 对象，因为 toString() 方法
        x2.intern();
        String x1 = "cd";

        System.out.println(x1 == x2); //true
    }
}
```

- == 比较基本数据类型：比较的是具体的值
- == 比较引用数据类型：比较的是对象地址值

结论：

```java
String s1 = "ab";								// 仅放入串池
String s2 = new String("a") + new String("b");	// 仅放入堆
// 上面两条指令的结果和下面的 效果 相同
String s = new String("ab");
```



****



##### 常见问题

问题一：

```java
public static void main(String[] args) {
    String s = new String("a") + new String("b");//new String("ab")
    //在上一行代码执行完以后，字符串常量池中并没有"ab"

    String s2 = s.intern();
    //jdk6：串池中创建一个字符串"ab"
    //jdk8：串池中没有创建字符串"ab",而是创建一个引用指向 new String("ab")，将此引用返回

    System.out.println(s2 == "ab");//jdk6:true  jdk8:true
    System.out.println(s == "ab");//jdk6:false  jdk8:true
}
```

问题二：

```java
public static void main(String[] args) {
    String str1 = new StringBuilder("58").append("tongcheng").toString();
    System.out.println(str1 == str1.intern());//true，字符串池中不存在，把堆中的引用复制一份放入串池

    String str2 = new StringBuilder("ja").append("va").toString();
    System.out.println(str2 == str2.intern());//false，字符串池中存在，直接返回已经存在的引用
}
```

原因：

* System 类当调用 Version 的静态方法，导致 Version 初始化：

  ```java
  private static void initializeSystemClass() {
      sun.misc.Version.init();
  }
  ```

* Version 类初始化时需要对静态常量字段初始化，被 launcher_name 静态常量字段所引用的 `"java"` 字符串字面量就被放入的字符串常量池：

  ```java
  package sun.misc;
  
  public class Version {
      private static final String launcher_name = "java";
      private static final String java_version = "1.8.0_221";
      private static final String java_runtime_name = "Java(TM) SE Runtime Environment";
      private static final String java_profile_name = "";
      private static final String java_runtime_version = "1.8.0_221-b11";
      //...
  }
  ```



***



##### 内存位置

Java 7 之前，String Pool 被放在运行时常量池中，属于永久代；Java 7 以后，String Pool 被移到堆中，这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误

演示 StringTable 位置：

* `-Xmx10m` 设置堆内存 10m

* 在 JDK8 下设置： `-Xmx10m -XX:-UseGCOverheadLimit`（运行参数在 Run Configurations VM options）

* 在 JDK6 下设置： `-XX:MaxPermSize=10m`

  ```java
  public static void main(String[] args) throws InterruptedException {
      List<String> list = new ArrayList<String>();
      int i = 0;
      try {
          for (int j = 0; j < 260000; j++) {
              list.add(String.valueOf(j).intern());
              i++;
          }
      } catch (Throwable e) {
          e.printStackTrace();
      } finally {
          System.out.println(i);
      }
  }
  ```

![](JavaSE-3-API.assets/JVM-内存图对比.png)



***



#### 优化常量池

两种方式：

* 调整 -XX:StringTableSize=桶个数，数量越少，性能越差

* intern 将字符串对象放入常量池，通过复用字符串的引用，减少内存占用

```java
/**
 * 演示 intern 减少内存占用
 * -XX:StringTableSize=200000 -XX:+PrintStringTableStatistics
 * -Xsx500m -Xmx500m -XX:+PrintStringTableStatistics -XX:StringTableSize=200000
 */
public class Demo1_25 {
    public static void main(String[] args) throws IOException {
        List<String> address = new ArrayList<>();
        System.in.read();
        for (int i = 0; i < 10; i++) {
            //很多数据
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("linux.words"), "utf-8"))) {
                String line = null;
                long start = System.nanoTime();
                while (true) {
                    line = reader.readLine();
                    if(line == null) {
                        break;
                    }
                    address.add(line.intern());
                }
                System.out.println("cost:" +(System.nanoTime()-start)/1000000);
            }
        }
        System.in.read();
    }
}
```



***



#### 不可变好处

* 可以缓存 hash 值，例如 String 用做 HashMap 的 key，不可变的特性可以使得 hash 值也不可变，只要进行一次计算
* String Pool 的需要，如果一个 String 对象已经被创建过了，就会从 String Pool 中取得引用，只有 String 是不可变的，才可能使用 String Pool
* 安全性，String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是
* String 不可变性天生具备线程安全，可以在多个线程中安全地使用
* 防止子类继承，破坏 String 的 API 的使用





***



### StringBuilder

String StringBuffer 和 StringBuilder 区别：

* String : **不可变**的字符序列，线程安全
* StringBuffer : **可变**的字符序列，线程安全，底层方法加 synchronized，效率低
* StringBuilder : **可变**的字符序列，JDK5.0 新增；线程不安全，效率高

相同点：底层使用 char[] 存储

构造方法：

* `public StringBuilder()`：创建一个空白可变字符串对象，不含有任何内容
* `public StringBuilder(String str)`：根据字符串的内容，来创建可变字符串对象

常用API : 

* `public StringBuilder append(任意类型)`：添加数据，并返回对象本身
* `public StringBuilder reverse()`：返回相反的字符序列
* `public String toString()`：通过 toString() 就可以实现把 StringBuilder 转换为 String

存储原理：

```java
String str = "abc";
char data[] = {'a', 'b', 'c'};
StringBuffer sb1 = new StringBuffer();//new byte[16] 
sb1.append('a'); //value[0] = 'a';
```

append 源码：扩容为二倍

```java
public AbstractStringBuilder append(String str) {
    if (str == null) return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
private void ensureCapacityInternal(int minimumCapacity) {
    // 创建超过数组长度就新的char数组，把数据拷贝过去
    if (minimumCapacity - value.length > 0) {
        //int newCapacity = (value.length << 1) + 2;每次扩容2倍+2
        value = Arrays.copyOf(value, newCapacity(minimumCapacity));
    }
}
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
    // 将字符串中的字符复制到目标字符数组中
	// 字符串调用该方法，此时value是字符串的值，dst是目标字符数组
    System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
}
```





****



### Arrays

Array 的工具类 Arrays

常用API：

* `public static String toString(int[] a)`：返回指定数组的内容的字符串表示形式
* `public static void sort(int[] a)`：按照数字顺序排列指定的数组
* `public static int binarySearch(int[] a, int key)`：利用二分查找返回指定元素的索引
* `public static <T> List<T> asList(T... a)`：返回由指定数组支持的列表

```java
public class MyArraysDemo {
      public static void main(String[] args) {
		//按照数字顺序排列指定的数组
        int [] arr = {3,2,4,6,7};
        Arrays.sort(arr);
        System.out.println(Arrays.toString(arr));
		
        int [] arr = {1,2,3,4,5,6,7,8,9,10};
        int index = Arrays.binarySearch(arr, 0);
        System.out.println(index);
        //1,数组必须有序
        //2.如果要查找的元素存在,那么返回的是这个元素实际的索引
        //3.如果要查找的元素不存在,那么返回的是 (-插入点-1)
            //插入点:如果这个元素在数组中,他应该在哪个索引上.
      }
  }
```





***



### Random

用于生成伪随机数。

使用步骤：
1. 导入包：`import java.util.Random`
2. 创建对象：`Random r = new Random()`
3. 随机整数：`int num = r.nextInt(10)`
   * 解释：10 代表的是一个范围，如果括号写 10，产生的随机数就是 0 - 9，括号写 20 的随机数则是 0 - 19
   * 获取 0 - 10：`int num = r.nextInt(10 + 1)`

4. 随机小数：`public double nextDouble()` 从范围 `0.0d` 至 `1.0d` （左闭右开），伪随机地生成并返回



***



### System

System 代表当前系统

静态方法：

* `public static void exit(int status)`：终止 JVM 虚拟机，**非 0 是异常终止**

* `public static long currentTimeMillis()`：获取当前系统此刻时间毫秒值

* `static void arraycopy(Object var0, int var1, Object var2, int var3, int var4)`：数组拷贝
  * 参数一：原数组
  * 参数二：从原数组的哪个位置开始赋值
  * 参数三：目标数组
  * 参数四：从目标数组的哪个位置开始赋值
  * 参数五：赋值几个

```java
public class SystemDemo {
    public static void main(String[] args) {
        //System.exit(0); // 0代表正常终止!!
        long startTime = System.currentTimeMillis();//定义sdf 按照格式输出
        for(int i = 0; i < 10000; i++){输出i}
		long endTime = new Date().getTime();
		System.out.println( (endTime - startTime)/1000.0 +"s");//程序用时

        int[] arr1 = new int[]{10 ,20 ,30 ,40 ,50 ,60 ,70};
        int[] arr2 = new int[6]; // [ 0 , 0 , 0 , 0 , 0 , 0]
        // 变成arrs2 = [0 , 30 , 40 , 50 , 0 , 0 ]
        System.arraycopy(arr1, 2, arr2, 1, 3);
    }
}
```





***



### Date

构造器：

* `public Date()`：创建当前系统的此刻日期时间对象。
*  `public Date(long time)`：把时间毫秒值转换成日期对象

方法：

* `public long getTime()`：返回自 1970 年 1 月 1 日 00:00:00 GMT 以来总的毫秒数。

时间记录的两种方式：

1. Date 日期对象
2. 时间毫秒值：从 `1970-01-01 00:00:00` 开始走到此刻的总的毫秒值，1s = 1000ms

```java
public class DateDemo {
    public static void main(String[] args) {
        Date d = new Date();
        System.out.println(d);//Fri Oct 16 21:58:44 CST 2020
        long time = d.getTime() + 121*1000;//过121s是什么时间
        System.out.println(time);//1602856875485
        
        Date d1 = new Date(time);
        System.out.println(d1);//Fri Oct 16 22:01:15 CST 2020
    }
}
```

```java
public static void main(String[] args){
    Date d = new Date();
    long startTime = d.getTime();
    for(int i = 0; i < 10000; i++){输出i}
    long endTime = new Date().getTime();
    System.out.println( (endTime - startTime) / 1000.0 +"s");
    //运行一万次输出需要多长时间
}
```



***



### DateFormat

DateFormat 作用：

1. 可以把“日期对象”或者“时间毫秒值”格式化成我们喜欢的时间形式（格式化时间）
2. 可以把字符串的时间形式解析成日期对象（解析字符串时间）

DateFormat 是一个抽象类，不能直接使用，使用它的子类：SimpleDateFormat

SimpleDateFormat  简单日期格式化类：

* `public SimpleDateFormat(String pattern)`：指定时间的格式创建简单日期对象
* `public String format(Date date) `：把日期对象格式化成我们喜欢的时间形式，返回字符串
* `public String format(Object time)`：把时间毫秒值格式化成设定的时间形式，返回字符串!
* `public Date parse(String date)`：把字符串的时间解析成日期对象

>yyyy年MM月dd日 HH:mm:ss EEE a" 周几 上午下午

```java
public static void main(String[] args){
	Date date = new Date();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss);
    String time = sdf.format(date);
    System.out.println(time);//2020-10-18 19:58:34
    //过121s后是什么时间
    long time = date.getTime();
    time+=121;
    System.out.println(sdf.formate(time));
    String d = "2020-10-18 20:20:20";//格式一致
    Date newDate = sdf.parse(d);
    System.out.println(sdf.format(newDate)); //按照前面的方法输出
}
```




****



### Calendar

Calendar 代表了系统此刻日期对应的日历对象，是一个抽象类，不能直接创建对象

Calendar 日历类创建日历对象：`Calendar rightNow = Calendar.getInstance()`（**饿汉单例模式**）

Calendar 的方法：

* `public static Calendar getInstance()`：返回一个日历类的对象
* `public int get(int field)`：取日期中的某个字段信息
* `public void set(int field,int value)`：修改日历的某个字段信息
* `public void add(int field,int amount)`：为某个字段增加/减少指定的值
* `public final Date getTime()`：拿到此刻日期对象
* `public long getTimeInMillis()`：拿到此刻时间毫秒值

```java
public static void main(String[] args){
	Calendar rightNow = Calendar.getInsance(); 
	int year = rightNow.get(Calendar.YEAR);//获取年
    int month = rightNow.get(Calendar.MONTH) + 1;//月要+1
    int days = rightNow.get(Calendar.DAY_OF_YEAR);
    rightNow.set(Calendar.YEAR , 2099);//修改某个字段
    rightNow.add(Calendar.HOUR , 15);//加15小时  -15就是减去15小时
    Date date = rightNow.getTime();//日历对象
    long time = rightNow.getTimeInMillis();//时间毫秒值
    //700天后是什么日子
    rightNow.add(Calendar.DAY_OF_YEAR , 701);
    Date date d = rightNow.getTime();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    System.out.println(sdf.format(d));//输出700天后的日期
}
```



***



### LocalDateTime

JDK1.8 新增，线程安全

+ LocalDate       表示日期（年月日）  
+ LocalTime       表示时间（时分秒）
+ LocalDateTime    表示时间+ 日期 （年月日时分秒）

构造方法：

* public static LocalDateTime now()：获取当前系统时间 
* public static LocalDateTime of(年, 月 , 日, 时, 分, 秒)：使用指定年月日和时分秒初始化一个对象

常用API：

| 方法名                                                    | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| public int getYear()                                      | 获取年                                                       |
| public int getMonthValue()                                | 获取月份（1-12）                                             |
| public int getDayOfMonth()                                | 获取月份中的第几天（1-31）                                   |
| public int getDayOfYear()                                 | 获取一年中的第几天（1-366）                                  |
| public DayOfWeek getDayOfWeek()                           | 获取星期                                                     |
| public int getMinute()                                    | 获取分钟                                                     |
| public int getHour()                                      | 获取小时                                                     |
| public LocalDate  toLocalDate()                           | 转换成为一个 LocalDate 对象（年月日）                        |
| public LocalTime toLocalTime()                            | 转换成为一个 LocalTime 对象（时分秒）                        |
| public String format(指定格式)                            | 把一个 LocalDateTime 格式化成为一个字符串                    |
| public LocalDateTime parse(准备解析的字符串, 解析格式)    | 把一个日期字符串解析成为一个 LocalDateTime 对象              |
| public static DateTimeFormatter ofPattern(String pattern) | 使用指定的日期模板获取一个日期格式化器 DateTimeFormatter 对象 |

```java
public class JDK8DateDemo2 {
    public static void main(String[] args) {
        LocalDateTime now = LocalDateTime.now();
        System.out.println(now);

        LocalDateTime localDateTime = LocalDateTime.of(2020, 11, 11, 11, 11, 11);
        System.out.println(localDateTime);
        DateTimeFormatter pattern = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");
        String s = localDateTime.format(pattern);
		LocalDateTime parse = LocalDateTime.parse(s, pattern);
    }
}
```

| 方法名                                      | 说明           |
| ------------------------------------------- | -------------- |
| public LocalDateTime plusYears (long years) | 添加或者减去年 |
| public LocalDateTime withYear(int year)     | 直接修改年     |



**时间间隔** Duration 类API：

| 方法名                                           | 说明                 |
| ------------------------------------------------ | -------------------- |
| public static Period between(开始时间,结束时间)  | 计算两个“时间"的间隔 |
| public int getYears()                            | 获得这段时间的年数   |
| public int getMonths()                           | 获得此期间的总月数   |
| public int getDays()                             | 获得此期间的天数     |
| public long toTotalMonths()                      | 获取此期间的总月数   |
| public static Durationbetween(开始时间,结束时间) | 计算两个“时间"的间隔 |
| public long toSeconds()                          | 获得此时间间隔的秒   |
| public long toMillis()                           | 获得此时间间隔的毫秒 |
| public long toNanos()                            | 获得此时间间隔的纳秒 |

```java
public class JDK8DateDemo9 {
    public static void main(String[] args) {
        LocalDate localDate1 = LocalDate.of(2020, 1, 1);
        LocalDate localDate2 = LocalDate.of(2048, 12, 12);
        Period period = Period.between(localDate1, localDate2);
        System.out.println(period);//P28Y11M11D
		Duration duration = Duration.between(localDateTime1, localDateTime2);
        System.out.println(duration);//PT21H57M58S
    }
}
```



***



### Math

Math 用于做数学运算

Math 类中的方法全部是静态方法，直接用类名调用即可：

| 方法                                         | 说明                              |
| -------------------------------------------- | --------------------------------- |
| public static int abs(int a)                 | 获取参数a的绝对值                 |
| public static double ceil(double a)          | 向上取整                          |
| public static double floor(double a)         | 向下取整                          |
| public static double pow(double a, double b) | 获取 a 的 b 次幂                  |
| public static long round(double a)           | 四舍五入取整                      |
| public static int max(int a,int b)           | 返回较大值                        |
| public static int min(int a,int b)           | 返回较小值                        |
| public static double random()                | 返回值为 double 的正值，[0.0,1.0) |

```java
public class MathDemo {
    public static void main(String[] args) {
        // 1.取绝对值:返回正数。
        System.out.println(Math.abs(10));
        System.out.println(Math.abs(-10.3));
        // 2.向上取整: 5
        System.out.println(Math.ceil(4.00000001)); // 5.0
        System.out.println(Math.ceil(-4.00000001));//4.0
        // 3.向下取整：4
        System.out.println(Math.floor(4.99999999)); // 4.0
        System.out.println(Math.floor(-4.99999999)); // 5.0
        // 4.求指数次方
        System.out.println(Math.pow(2 , 3)); // 2^3 = 8.0
        // 5.四舍五入 10
        System.out.println(Math.round(4.49999)); // 4
        System.out.println(Math.round(4.500001)); // 5
        System.out.println(Math.round(5.5));//6
    }
}
```



****



### DecimalFormat

使任何形式的数字解析和格式化

```java
public static void main(String[]args){
    double pi = 3.1415927;　//圆周率
    //取一位整数
    System.out.println(new DecimalFormat("0").format(pi));　　　//3
    //取一位整数和两位小数
    System.out.println(new DecimalFormat("0.00").format(pi));　//3.14
    //取两位整数和三位小数，整数不足部分以0填补。
    System.out.println(new DecimalFormat("00.000").format(pi));// 03.142
    //取所有整数部分
    System.out.println(new DecimalFormat("#").format(pi));　　　//3
    //以百分比方式计数，并取两位小数
    System.out.println(new DecimalFormat("#.##%").format(pi));　//314.16%

    long c =299792458;　　//光速
    //显示为科学计数法，并取五位小数
    System.out.println(new DecimalFormat("#.#####E0").format(c));//2.99792E8
    //显示为两位整数的科学计数法，并取四位小数
    System.out.println(new DecimalFormat("00.####E0").format(c));//29.9792E7
    //每三位以逗号进行分隔。
    System.out.println(new DecimalFormat(",###").format(c));//299,792,458
    //将格式嵌入文本
    System.out.println(new DecimalFormat("光速大小为每秒,###米。").format(c));

}
```





***



### BigDecimal

Java 在 java.math 包中提供的 API 类，用来对超过16位有效位的数进行精确的运算

构造方法：

* `public static BigDecimal valueOf(double val)`：包装浮点数成为大数据对象。
* `public BigDecimal(double val)`
* `public BigDecimal(String val)`

常用API：

* `public BigDecimal add(BigDecimal value)`：加法运算
* `public BigDecimal subtract(BigDecimal value)`：减法运算 
* `public BigDecimal multiply(BigDecimal value)`：乘法运算 
* `public BigDecimal divide(BigDecimal value)`：除法运算
* `public double doubleValue()`：把 BigDecimal 转换成 double 类型
* `public int intValue()`：转为 int 其他类型相同
* `public BigDecimal divide (BigDecimal value，精确几位，舍入模式)`：除法

```java
public class BigDecimalDemo {
    public static void main(String[] args) {
        // 浮点型运算的时候直接+ - * / 可能会出现数据失真（精度问题）。
        System.out.println(0.1 + 0.2);
        System.out.println(1.301 / 100);
        
        double a = 0.1 ;
        double b = 0.2 ;
        double c = a + b ;
        System.out.println(c);//0.30000000000000004
        
        // 1.把浮点数转换成大数据对象运算
        BigDecimal a1 = BigDecimal.valueOf(a);
        BigDecimal b1 = BigDecimal.valueOf(b);
        BigDecimal c1 = a1.add(b1);//a1.divide(b1);也可以
		System.out.println(c1);

        // BigDecimal只是解决精度问题的手段，double数据才是我们的目的！！
        double d = c1.doubleValue();
    }
}
```

总结：

1. BigDecimal 是用来进行精确计算的
2. 创建 BigDecimal 的对象，构造方法使用参数类型为字符串的
3. 四则运算中的除法，如果除不尽请使用 divide 的三个参数的方法

```java
BigDecimal divide = bd1.divide(参与运算的对象,小数点后精确到多少位,舍入模式);
//参数1：表示参与运算的BigDecimal 对象。
//参数2：表示小数点后面精确到多少位
//参数3：舍入模式  
// BigDecimal.ROUND_UP  进一法
// BigDecimal.ROUND_FLOOR 去尾法
// BigDecimal.ROUND_HALF_UP 四舍五入
```



***



### Regex

#### 概述

正则表达式的作用：是一些特殊字符组成的校验规则，可以校验信息的正确性，校验邮箱、电话号码、金额等。

比如检验 qq 号：

```java
public static boolean checkQQRegex(String qq){
    return qq!=null && qq.matches("\\d{4,}");//即是数字 必须大于4位数
}// 用\\d  是因为\用来告诉它是一个校验类，不是普通的字符 比如 \t \n
```

java.util.regex 包主要包括以下三个类：

- Pattern 类：

  Pattern 对象是一个正则表达式的编译表示。Pattern 类没有公共构造方法，要创建一个 Pattern 对象，必须首先调用其公共静态编译方法，返回一个 Pattern 对象。该方法接受一个正则表达式作为它的第一个参数

- Matcher 类：

  Matcher 对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法，需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象

- PatternSyntaxException：

  PatternSyntaxException 是一个非强制异常类，它表示一个正则表达式模式中的语法错误。



***



#### 字符匹配

##### 普通字符

字母、数字、汉字、下划线、以及没有特殊定义的标点符号，都是“普通字符”。表达式中的普通字符，在匹配一个字符串的时候，匹配与之相同的一个字符。其他统称**元字符**



***



##### 特殊字符

\r\n 是 Windows 中的文本行结束标签，在 Unix/Linux 则是 \n

| 元字符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| \      | 将下一个字符标记为一个特殊字符或原义字符，告诉它是一个校验类，不是普通字符 |
| \f     | 换页符                                                       |
| \n     | 换行符                                                       |
| \r     | 回车符                                                       |
| \t     | 制表符                                                       |
| \\     | 代表 \ 本身                                                  |
| ()     | 使用 () 定义一个子表达式。子表达式的内容可以当成一个独立元素 |



***



##### 标准字符

能够与多种字符匹配的表达式，注意区分大小写，大写是相反的意思，只能校验**单**个字符。

| 元字符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| .      | 匹配任意一个字符（除了换行符），如果要匹配包括 \n 在内的所有字符，一般用 [\s\S] |
| \d     | 数字字符，0~9 中的任意一个，等价于 [0-9]                     |
| \D     | 非数字字符，等价于  [ ^0-9]                                  |
| \w     | 大小写字母或数字或下划线，等价于[a-zA-Z_0-9_]                |
| \W     | 对\w取非，等价于[ ^\w]                                       |
| \s     | 空格、制表符、换行符等空白字符的其中任意一个，等价于[\f\n\r\t\v] |
| \S     | 对 \s 取非                                                   |

\x 匹配十六进制字符，\0 匹配八进制，例如 \xA 对应值为 10 的 ASCII 字符 ，即 \n



***



##### 自定义符

自定义符号集合，[ ] 方括号匹配方式，能够匹配方括号中**任意一个**字符

| 元字符       | 说明                                      |
| ------------ | ----------------------------------------- |
| [ab5@]       | 匹配 "a" 或 "b" 或 "5" 或 "@"             |
| [^abc]       | 匹配 "a","b","c" 之外的任意一个字符       |
| [f-k]        | 匹配 "f"~"k" 之间的任意一个字母           |
| [^A-F0-3]    | 匹配 "A","F","0"~"3" 之外的任意一个字符   |
| [a-d[m-p]]   | 匹配 a 到 d 或者 m 到 p：[a-dm-p]（并集） |
| [a-z&&[m-p]] | 匹配 a 到 z 并且 m 到 p：[a-dm-p]（交集） |
| [^]          | 取反                                      |

* 正则表达式的特殊符号，被包含到中括号中，则失去特殊意义，除了 ^,- 之外，需要在前面加 \

* 标准字符集合，除小数点外，如果被包含于中括号，自定义字符集合将包含该集合。
  比如：[\d. \ -+] 将匹配：数字、小数点、+、-



***



##### 量词字符

修饰匹配次数的特殊符号。

* 匹配次数中的贪婪模式(匹配字符越多越好，默认 ！)，\* 和 + 都是贪婪型元字符。
* 匹配次数中的非贪婪模式（匹配字符越少越好，修饰匹配次数的特殊符号后再加上一个 ? 号）

| 元字符 | 说明                              |
| ------ | --------------------------------- |
| X?     | X 一次或一次也没，有相当于 {0,1}  |
| X*     | X 不出现或出现任意次，相当于 {0,} |
| X+     | X 至少一次，相当于 {1,}           |
| X{n}   | X 恰好 n 次                       |
| {n,}   | X 至少 n 次                       |
| {n,m}  | X 至少 n 次，但是不超过 m 次      |



***



#### 位置匹配

##### 字符边界

本组标记匹配的不是字符而是位置，符合某种条件的位置

| 元字符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| ^      | 与字符串开始的地方匹配（在字符集合中用来求非，在字符集合外用作匹配字符串的开头） |
| $      | 与字符串结束的地方匹配                                       |
| \b     | 匹配一个单词边界                                             |



***



##### 捕获组

捕获组是把多个字符当一个单独单元进行处理的方法，它通过对括号内的字符分组来创建。

在表达式 `((A)(B(C)))`，有四个这样的组：((A)(B(C)))、(A)、(B(C))、(C)（按照括号从左到右依次为 group(1)...）

* 调用 matcher 对象的 groupCount 方法返回一个 int 值，表示 matcher 对象当前有多个捕获组。
* 特殊的组 group(0)、group()，代表整个表达式，该组不包括在 groupCount 的返回值中。 

| 表达式                    | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| \|  (分支结构)            | 左右两边表达式之间 "或" 关系，匹配左边或者右边               |
| ()  (捕获组)              | (1) 在被修饰匹配次数的时候，括号中的表达式可以作为整体被修饰<br/>(2) 取匹配结果的时候，括号中的表达式匹配到的内容可以被单独得到<br/>(3) 每一对括号分配一个编号,()的捕获根据左括号的顺序从 1 开始自动编号。捕获元素编号为零的第一个捕获是由整个正则表达式模式匹配的文本 |
| (?:Expression)   非捕获组 | 一些表达式中，不得不使用( )，但又不需要保存 () 中子表达式匹配的内容，这时可以用非捕获组来抵消使用( )带来的副作用。 |



***



##### 反向引用

反向引用（\number），又叫回溯引用：

* 每一对()会分配一个编号，使用 () 的捕获根据左括号的顺序从1开始自动编号

* 通过反向引用，可以对分组已捕获的字符串进行引用，继续匹配

* **把匹配到的字符重复一遍在进行匹配**

* 应用 1：

  ```java
  String regex = "((\d)3)\1[0-9](\w)\2{2}";
  ```

  * 首先匹配 ((\d)3)，其次 \1 匹配 ((\d)3) 已经匹配到的内容，\2 匹配 (\d)， {2} 指的是 \2 的值出现两次
  * 实例：23238n22（匹配到 2 未来就继续匹配 2）
  * 实例：43438n44

* 应用 2：爬虫

  ```java
  String regex = "<(h[1-6])>\w*?<\/\1>";
  ```

  匹配结果

  ```java
  <h1>x</h1>//匹配
  <h2>x</h2>//匹配
  <h3>x</h1>//不匹配
  ```

  

***



##### 零宽断言

预搜索（零宽断言）（环视）

* 只进行子表达式的匹配，匹配内容不计入最终的匹配结果，是零宽度

* 判断当前位置的前后字符，是否符合指定的条件，但不匹配前后的字符，**是对位置的匹配**

* 正则表达式匹配过程中，如果子表达式匹配到的是字符内容，而非位置，并被保存到最终的匹配结果中，那么就认为这个子表达式是占有字符的；如果子表达式匹配的仅仅是位置，或者匹配的内容并不保存到最终的匹配结果中，那么就认为这个子表达式是**零宽度**的。占有字符还是零宽度，是针对匹配的内容是否保存到最终的匹配结果中而言的

  | 表达式   | 说明                                    |
  | -------- | --------------------------------------- |
  | (?=exp)  | 断言自身出现的位置的后面能匹配表达式exp |
  | (?<=exp) | 断言自身出现的位置的前面能匹配表达式exp |
  | (?!exp)  | 断言此位置的后面不能匹配表达式exp       |
  | (?<!exp) | 断言此位置的前面不能匹配表达式exp       |




***



#### 匹配模式

正则表达式的匹配模式：

* IGNORECASE 忽略大小写模式
  * 匹配时忽略大小写。
  * 默认情况下，正则表达式是要区分大小写的。
* SINGLELINE 单行模式
  * 整个文本看作一个字符串，只有一个开头，一个结尾。
  * 使小数点 "." 可以匹配包含换行符（\n）在内的任意字符。
* MULTILINE 多行模式
  * 每行都是一个字符串，都有开头和结尾。
  * 在指定了 MULTILINE 之后，如果需要仅匹配字符串开始和结束位置，可以使用 \A 和 \Z



***



#### 分组匹配

Pattern 类：

* `static Pattern compile(String regex)`：将给定的正则表达式编译为模式
* `Matcher matcher(CharSequence input)`：创建一个匹配器，匹配给定的输入与此模式
* `static boolean matches(String regex, CharSequence input)`：编译正则表达式，并匹配输入

Matcher 类：

* `boolean find()`：扫描输入的序列，查找与该模式匹配的下一个子序列
* `String group()`：返回与上一个匹配的输入子序列，同 group(0)，匹配整个表达式的子字符串
* `String group(int group)`：返回在上一次匹配操作期间由给定组捕获的输入子序列 
* `int groupCount()`：返回此匹配器模式中捕获组的数量

```java
public class Demo01{
	public static void main(String[] args) {
		//表达式对象
		Pattern p = Pattern.compile("\\w+");
		//创建Matcher对象
		Matcher m = p.matcher("asfsdf2&&3323");
		//boolean b = m.matches();//尝试将整个字符序列与该模式匹配
		//System.out.println(b);//false
		//boolean b2 = m.find();//该方法扫描输入的序列，查找与该模式匹配的下一个子序列
		//System.out.println(b2);//true
		
		//System.out.println(m.find());
		//System.out.println(m.group());//asfsdf2
		//System.out.println(m.find());
		//System.out.println(m.group());//3323
		
		while(m.find()){
			System.out.println(m.group());	//group(),group(0)匹配整个表达式的子字符串
			System.out.println(m.group(0));
		}
		
	}
}
```

```java
public class Demo02 {
	public static void main(String[] args) {
		//在这个字符串：asfsdf23323，是否符合指定的正则表达式：\w+
		//表达式对象
        Pattern p = Pattern.compile("(([a-z]+)([0-9]+))");//不需要加多余的括号
		//创建Matcher对象
		Matcher m = p.matcher("aa232**ssd445");
	
		while(m.find()){
			System.out.println(m.group());//aa232  ssd445
			System.out.println(m.group(1));//aa232  ssd445
			System.out.println(m.group(2));//aa     ssd
            System.out.println(m.group(3));//232    445 
		}
	}
}
```

* 正则表达式改为 `"(([a-z]+)(?:[0-9]+))"`   没有 group(3) 因为是非捕获组
* 正则表达式改为 `"([a-z]+)([0-9]+)"`  没有 group(3)    aa232  - aa  --232



***



#### 应用

##### 基本验证

```java
public static void main(String[] args){
	System.out.println("a".matches("[abc]"));//true判断a是否在abc
    System.out.println("a".matches("[^abc]"));//false 判断a是否在abc之外的
    System.out.println("a".matches("\\d")); //false 是否a是整数
    System.out.println("a".matches("\\w")); //true 是否是字符
    System.out.println("你".matches("\\w")); // false
    System.out.println("aa".matches("\\w"));//false 只能检验单个字符
    
    // 密码 必须是数字 字母 下划线 至少 6位
	System.out.println("ssds3c".matches("\\w{6,}")); // true
    // 验证。必须是数字和字符  必须是4位
    System.out.println("dsd22".matches("[a-zA-Z0-9]{4}")); // false
    System.out.println("A3dy".matches("[a-zA-Z0-9]{4}")); // true
}
```



***



##### 验证号码

```java
//1开头 第二位是2-9的数字
public static void checkPhone(String phone){
    if(phone.matches("1[3-9]\\d{9}")){
        System.out.println("手机号码格式正确！");
    } else {.......}
}
//1111@qq.com  zhy@pic.com.cn
public static void checkEmail(String email){
    if(email.matches("\\w{1,}@\\w{1,}(\\.\\w{2,5}){1,2}")){
        System.out.println("邮箱格式正确！");
    }// .是任意字符 \\.就是点
}
```



***



##### 查找替换

* `public String[] split(String regex)`：按照正则表达式匹配的内容进行分割字符串，反回一个字符串数组
* `public String replaceAll(String regex,String newStr)`：按照正则表达式匹配的内容进行替换

```java
//数组分割
public static void main(String[] args) {
	// 1.split的基础用法
	String names = "风清扬,张无忌,周芷若";
	// 以“，”分割成字符串数组
    String[] nameArrs = names.split(",");

    // 2.split集合正则表达式做分割
    String names1 = "风清扬lv434fda324张无忌87632fad2342423周芷若";
    // 以匹配正则表达式的内容为分割点分割成字符串数组
	String[] nameArrs1 = names1.split("\\w+");
    
	// 使用正则表达式定位出内容，替换成/
	System.out.println(names1.replaceAll("\\w+","/"));//风清扬/张无忌/周芷若

	String names3 = "风清扬,张无忌,周芷若";
	System.out.println(names3.replaceAll(",","-"));//风清扬-张无忌-周芷若
}
```



***



##### 搜索号码

找出所有 189 和 132 开头的手机号

```java
public class RegexDemo {
    public static void main(String[] args) {
        String rs = "189asjk65as1891898777745gkkkk189745612318936457894";
        String regex = "(?=((189|132)\\d{8}))";
        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(rs);
        while (matcher.find()) {
            System.out.println(matcher.group(1));
        }
    }
}
```





***





