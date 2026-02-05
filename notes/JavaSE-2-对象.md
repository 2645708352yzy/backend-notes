[[Java]]

# SE

## 对象

### 概述

Java 是一种面向对象的高级编程语言

面向对象三大特征：**封装，继承，多态**

两个概念：类和对象

* 类：相同事物共同特征的描述，类只是学术上的一个概念并非真实存在的，只能描述一类事物
* 对象：是真实存在的实例， 实例 == 对象，**对象是类的实例化**
* 结论：有了类和对象就可以描述万千世界所有的事物，必须先有类才能有对象



***



### 类

#### 定义

定义格式

```java
修饰符 class 类名{
}
```

1. 类名的首字母建议大写，满足驼峰模式，比如 StudentNameCode
2. 一个 Java 代码中可以定义多个类，按照规范一个 Java 文件一个类
3. 一个 Java 代码文件中，只能有一个类是 public 修饰，**public 修饰的类名必须成为当前 Java 代码的文件名称**

```java
类中的成分:有且仅有五大成分
修饰符 class 类名{
		1.成员变量(Field):  	描述类或者对象的属性信息的。
        2.成员方法(Method):		描述类或者对象的行为信息的。
		3.构造器(Constructor):	 初始化一个对象返回。
		4.代码块
		5.内部类
	  }
类中有且仅有这五种成分，否则代码报错！
public class ClassDemo {
    System.out.println(1);//报错
}
```



***



#### 构造器

构造器格式：

```java
修饰符 类名(形参列表){

}
```

作用：初始化类的一个对象返回

分类：无参数构造器，有参数构造器

注意：**一个类默认自带一个无参数构造器**，写了有参数构造器默认的无参数构造器就消失，还需要用无参数构造器就要重新写

构造器初始化对象的格式：类名 对象名称 = new 构造器

* 无参数构造器的作用：初始化一个类的对象（使用对象的默认值初始化）返回
* 有参数构造器的作用：初始化一个类的对象（可以在初始化对象的时候为对象赋值）返回



------



### 包

包：分门别类的管理各种不同的技术，便于管理技术，扩展技术，阅读技术

定义包的格式：`package 包名`，必须放在类名的最上面

导包格式：`import 包名.类名`

相同包下的类可以直接访问；不同包下的类必须导包才可以使用



***



### 封装

封装的哲学思维：合理隐藏，合理暴露

封装最初的目的：提高代码的安全性和复用性，组件化

封装的步骤：

1. **成员变量应该私有，用 private 修饰，只能在本类中直接访问**
2. **提供成套的 getter 和 setter 方法暴露成员变量的取值和赋值**

使用 private 修饰成员变量的原因：实现数据封装，不想让别人使用修改你的数据，比较安全



***



### this

this 关键字的作用：

* this 关键字代表了当前对象的引用
* this 出现在方法中：**哪个对象调用这个方法 this 就代表谁**
* this 可以出现在构造器中：代表构造器正在初始化的那个对象
* this 可以区分变量是访问的成员变量还是局部变量



------



### static

#### 基本介绍

Java 是通过成员变量是否有 static 修饰来区分是类的还是属于对象的

按照有无 static 修饰，成员变量和方法可以分为：

* 成员变量：
  * 静态成员变量（类变量）：static 修饰的成员变量，属于类本身，**与类一起加载一次，只有一个**，直接用类名访问即可
  * 实例成员变量：无 static 修饰的成员变量，属于类的每个对象的，**与类的对象一起加载**，对象有多少个，实例成员变量就加载多少个，必须用类的对象来访问
  
* 成员方法：
  * 静态方法：有 static 修饰的成员方法称为静态方法也叫类方法，属于类本身的，直接用类名访问即可
  * 实例方法：无 static 修饰的成员方法称为实例方法，属于类的每个对象的，必须用类的对象来访问



****



#### static 用法

成员变量的访问语法：

* 静态成员变量：只有一份可以被类和类的对象**共享访问**
  * 类名.静态成员变量（同一个类中访问静态成员变量可以省略类名不写）
  * 对象.静态成员变量（不推荐）

* 实例成员变量：
  * 对象.实例成员变量（先创建对象）

成员方法的访问语法：

* 静态方法：有 static 修饰，属于类

  * 类名.静态方法（同一个类中访问静态成员可以省略类名不写）
  * 对象.静态方法（不推荐，参考 JVM → 运行机制 → 方法调用）
  
* 实例方法：无 static 修饰，属于对象

  * 对象.实例方法
  
  ```java
  public class Student {
      // 1.静态方法：有static修饰，属于类，直接用类名访问即可！
      public static void inAddr(){ }
      // 2.实例方法：无static修饰，属于对象，必须用对象访问！
      public void eat(){}
      
      public static void main(String[] args) {
          // a.类名.静态方法
          Student.inAddr();
          inAddr();
          // b.对象.实例方法
          // Student.eat(); // 报错了！
          Student sea = new Student();
          sea.eat();
      }
  }
  ```



***



#### 两个问题

内存问题：

* 栈内存存放 main 方法和地址

* 堆内存存放对象和变量

* 方法区存放 class 和静态变量（jdk8 以后移入堆）

访问问题：

* 实例方法是否可以直接访问实例成员变量？可以，因为它们都属于对象
* 实例方法是否可以直接访问静态成员变量？可以，静态成员变量可以被共享访问
* 实例方法是否可以直接访问实例方法? 可以，实例方法和实例方法都属于对象
* 实例方法是否可以直接访问静态方法？可以，静态方法可以被共享访问
* 静态方法是否可以直接访问实例变量？ 不可以，实例变量**必须用对象访问**！！
* 静态方法是否可以直接访问静态变量？ 可以，静态成员变量可以被共享访问。
* 静态方法是否可以直接访问实例方法? 不可以，实例方法必须用对象访问！！
* 静态方法是否可以直接访问静态方法？可以，静态方法可以被共享访问！！



------



### 继承

#### 基本介绍

继承是 Java 中一般到特殊的关系，是一种子类到父类的关系

* 被继承的类称为：父类/超类
* 继承父类的类称为：子类

继承的作用：

* **提高代码的复用**，相同代码可以定义在父类中
* 子类继承父类，可以直接使用父类这些代码（相同代码重复利用）
* 子类得到父类的属性（成员变量）和行为（方法），还可以定义自己的功能，子类更强大

继承的特点：

1. 子类的全部构造器默认先访问父类的无参数构造器，再执行自己的构造器
2. **单继承**：一个类只能继承一个直接父类
3. 多层继承：一个类可以间接继承多个父类（家谱）
4. 一个类可以有多个子类
5. 一个类要么默认继承了 Object 类，要么间接继承了 Object 类，**Object 类是 Java 中的祖宗类**

继承的格式：

```java
子类 extends 父类{

}
```

子类不能继承父类的东西：

* 子类不能继承父类的构造器，子类有自己的构造器
* 子类是不能可以继承父类的私有成员的，可以反射暴力去访问继承自父类的私有成员
* 子类是不能继承父类的静态成员，父类静态成员只有一份可以被子类共享访问，**共享并非继承**

```java
public class ExtendsDemo {
    public static void main(String[] args) {
        Cat c = new Cat();
        // c.run();
        Cat.test();
        System.out.println(Cat.schoolName);
    }
}

class Cat extends Animal{
}

class Animal{
    public static String schoolName ="seazean";
    public static void test(){}
    private void run(){}
}
```



***



#### 变量访问

继承后成员变量的访问特点：**就近原则**，子类有找子类，子类没有找父类，父类没有就报错

如果要申明访问父类的成员变量可以使用：super.父类成员变量，super指父类引用

```java
public class ExtendsDemo {
    public static void wmain(String[] args) {
        Wolf w = new Wolf();w
        w.showName();
    }
}

class Wolf extends Animal{
    private String name = "子类狼";
    public void showName(){
        String name = "局部名称";
        System.out.println(name); // 局部name
        System.out.println(this.name); // 子类对象的name
        System.out.println(super.name); // 父类的
        System.out.println(name1); // 父类的
        //System.out.println(name2); // 报错。子类父类都没有
    }
}

class Animal{
    public String name = "父类动物名称";
    public String name1 = "父类";
}
```



***



#### 方法访问

子类继承了父类就得到了父类的方法，**可以直接调用**，受权限修饰符的限制，也可以重写方法

方法重写：子类重写一个与父类申明一样的方法来**覆盖**父类的该方法

方法重写的校验注解：@Override

* 方法加了这个注解，那就必须是成功重写父类的方法，否则报错
* @Override 优势：可读性好，安全，优雅

**子类可以扩展父类的功能，但不能改变父类原有的功能**，重写有以下三个限制：

- 子类方法的访问权限必须大于等于父类方法
- 子类方法的返回类型必须是父类方法返回类型或为其子类型
- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型

继承中的隐藏问题：

- 子类和父类方法都是静态的，那么子类中的方法会隐藏父类中的方法
- 在子类中可以定义和父类成员变量同名的成员变量，此时子类的成员变量隐藏了父类的成员变量，在创建对象为对象分配内存的过程中，**隐藏变量依然会被分配内存**

```java
public class ExtendsDemo {
    public static void main(String[] args) {
        Wolf w = new Wolf();
        w.run();
    }
}
class Wolf extends Animal{
    @Override
    public void run(){}//
}
class Animal{
    public void run(){}
}
```



***



#### 常见问题

* 为什么子类构造器会先调用父类构造器？
  
  1. 子类的构造器的第一行默认 super() 调用父类的无参数构造器，写不写都存在
  2. 子类继承父类，子类就得到了父类的属性和行为。调用子类构造器初始化子类对象数据时，必须先调用父类构造器初始化继承自父类的属性和行为
  3. 参考 JVM → 类加载 → 对象创建
  
  ```java
  class Animal {
      public Animal() {
          System.out.println("==父类Animal的无参数构造器==");
      }
  }
  
  class Tiger extends Animal {
      public Tiger() {
          super(); // 默认存在的，根据参数去匹配调用父类的构造器。
          System.out.println("==子类Tiger的无参数构造器==");
      }
      public Tiger(String name) {
          //super();  默认存在的，根据参数去匹配调用父类的构造器。
          System.out.println("==子类Tiger的有参数构造器==");
      }
  }
  ```
  
* **为什么 Java 是单继承的？**
  
  答：反证法，假如 Java 可以多继承，请看如下代码：
  
  ```java
  class A{
  	public void test(){
  		System.out.println("A");
  	}
  }
  class B{
  	public void test(){
  		System.out.println("B");
  	}
  }
  class C extends A , B {
  	public static void main(String[] args){
  		C c = new C();
          c.test(); 
          // 出现了类的二义性！所以Java不能多继承！！
  	}
  }
  ```
  
  



------



### super

继承后 super 调用父类构造器，父类构造器初始化继承自父类的数据。


总结与拓展：

* this 代表了当前对象的引用（继承中指代子类对象）：this.子类成员变量、this.子类成员方法、**this(...)** 可以根据参数匹配访问本类其他构造器
* super 代表了父类对象的引用（继承中指代了父类对象空间）：super.父类成员变量、super.父类的成员方法、super(...) 可以根据参数匹配访问父类的构造器

注意：

* this(...) 借用本类其他构造器，super(...) 调用父类的构造器
* this(...) 或 super(...) 必须放在构造器的第一行，否则报错
* this(...) 和 super(...) **不能同时出现**在构造器中，因为构造函数必须出现在第一行上，只能选择一个

```java
public class ThisDemo {
    public static void main(String[] args) {
        // 需求：希望如果不写学校默认就是”张三“！
        Student s1 = new Student("天蓬元帅", 1000 );
        Student s2 = new Student("齐天大圣", 2000, "清华大学" );
    }
}
class Study extends Student {
   public Study(String name, int age, String schoolName) {
        super(name , age , schoolName) ; 
       // 根据参数匹配调用父类构造器
   }
}

class Student{
    private String name ;
    private int age ;
    private String schoolName ;

    public Student() {
    }
    public Student(String name , int age){
        // 借用兄弟构造器的功能！
        this(name , age , "张三");
    }
	public Student(String name, int age, String schoolName) {
        this.name = name;
        this.age = age;
        this.schoolName = schoolName;
    }
	// .......get + set
}
```



***



### final

#### 基本介绍

final 用于修饰：类，方法，变量

* final 修饰类，类不能被继承了，类中的方法和变量可以使用
* final 可以修饰方法，方法就不能被重写
* final 修饰变量总规则：变量有且仅能被赋值一次

final 和 abstract 的关系是**互斥关系**，不能同时修饰类或者同时修饰方法



***



#### 修饰变量

##### 静态变量

final 修饰静态成员变量，变量变成了常量

常量：有 public static final 修饰，名称字母全部大写，多个单词用下划线连接

final 修饰静态成员变量可以在哪些地方赋值：

1. 定义的时候赋值一次

2. 可以在静态代码块中赋值一次

```java
public class FinalDemo {
	//常量：public static final修饰，名称字母全部大写，下划线连接。
    public static final String SCHOOL_NAME = "张三" ;
    public static final String SCHOOL_NAME1;

    static{
        //SCHOOL_NAME = "java";//报错
        SCHOOL_NAME1 = "张三1";
    }
}
```



***



##### 实例变量

final 修饰变量的总规则：有且仅能被赋值一次

final 修饰实例成员变量可以在哪些地方赋值 1 次：

1. 定义的时候赋值一次
2. 可以在实例代码块中赋值一次
3. 可以在每个构造器中赋值一次

```java
public class FinalDemo {
    private final String name = "张三" ;
    private final String name1;
    private final String name2;
    {
        // 可以在实例代码块中赋值一次。
        name1 = "张三1";
    }
	//构造器赋值一次
    public FinalDemo(){
        name2 = "张三2";
    }
    public FinalDemo(String a){
        name2 = "张三2";
    }

    public static void main(String[] args) {
        FinalDemo f1 = new FinalDemo();
        //f1.name = "张三1"; // 第二次赋值 报错！
    }
}
```



***



### 抽象类

#### 基本介绍

> 父类知道子类要完成某个功能，但是每个子类实现情况不一样

抽象方法：没有方法体，只有方法签名，必须用 abstract 修饰的方法就是抽象方法

抽象类：拥有抽象方法的类必须定义成抽象类，必须用 abstract 修饰，**抽象类是为了被继承**

一个类继承抽象类，**必须重写抽象类的全部抽象方法**，否则这个类必须定义成抽象类

```java
public class AbstractDemo {
    public static void main(String[] args) {
        Dog d = new Dog();
        d.run();
    }
}

class Dog extends Animal{
    @Override
    public void run() { 
        System.out.println("🐕跑"); 
    }
}

abstract class Animal{
    public abstract void run();
}
```



***



#### 常见问题

一、抽象类是否有构造器，是否可以创建对象?

* 抽象类有构造器，但是抽象类不能创建对象，类的其他成分它都具备，构造器提供给子类继承后调用父类构造器使用
* 抽象类中存在抽象方法，但不能执行，**抽象类中也可没有抽象方法**

> 抽象在学术上本身意味着不能实例化

```java
public class AbstractDemo {
    public static void main(String[] args) {
        //Animal a = new Animal(); 抽象类不能创建对象！
        //a.run(); // 抽象方法不能执行
    }
}
abstract class Animal{
    private String name;
    public static String schoolName = "张三";
    public Animal(){ }

    public abstract void run();
    //普通方法
    public void go(){ }
}
```

二、static 与 abstract 能同时使用吗？

答：不能，被 static 修饰的方法属于类，是类自己的东西，不是给子类来继承的，而抽象方法本身没有实现，就是用来给子类继承



***



#### 存在意义

**被继承**，抽象类就是为了被子类继承，否则抽象类将毫无意义（核心）

抽象类体现的是"模板思想"：**部分实现，部分抽象**，可以使用抽象类设计一个模板模式

```java
//作文模板
public class ExtendsDemo {
    public static void main(String[] args) {
        Student xiaoMa = new Student();
        xiaoMa.write();
    }
}
class Student extends Template{
    @Override
    public String writeText() {return "\t内容"}
}
// 1.写一个模板类：代表了作文模板。
abstract class Template{
    private String title = "\t\t\t\t\t标题";
    private String start = "\t开头";
    private String last = "\t结尾";
    public void write(){
        System.out.println(title+"\n"+start);
        System.out.println(writeText());
        System.out.println(last);
    }
    // 正文部分定义成抽象方法，交给子类重写！！
    public abstract String writeText();
}
```



***



### 接口

#### 基本介绍

接口是 Java 语言中一种引用类型，是方法的集合。

接口是更加彻底的抽象，接口中只有抽象方法和常量，没有其他成分

```java
 修饰符 interface 接口名称{
	// 抽象方法
	// 默认方法
	// 静态方法
	// 私有方法
}
```

* 抽象方法：接口中的抽象方法默认会加上 public abstract 修饰，所以可以省略不写

* 静态方法：静态方法必须有方法体

* 常量：是 public static final 修饰的成员变量，仅能被赋值一次，值不能改变。常量的名称规范上要求全部大写，多个单词下划线连接，public static final 可以省略不写

  ```java
  public interface InterfaceDemo{
      //public static final String SCHOOL_NAME = "张三";
  	String SCHOOL_NAME = "张三";
      
      //public abstract void run();
      void run();//默认补充
  }
  ```




***




#### 实现接口

**接口是用来被类实现的。**

* 类与类是继承关系：一个类只能直接继承一个父类，单继承
* 类与接口是实现关系：一个类可以实现多个接口，多实现，接口不能继承类
* 接口与接口继承关系：**多继承**

```java
修饰符 class 实现类名称 implements 接口1,接口2,接口3,....{

}
修饰符 interface 接口名 extend 接口1,接口2,接口3,....{
    
}
```

实现多个接口的使用注意事项：

1. 当一个类实现多个接口时，多个接口中存在同名的静态方法并不会冲突，只能通过各自接口名访问静态方法

2. 当一个类实现多个接口时，多个接口中存在同名的默认方法，实现类必须重写这个方法

3. 当一个类既继承一个父类，又实现若干个接口时，父类中成员方法与接口中默认方法重名，子类**就近选择执行父类**的成员方法

4. 接口中，没有构造器，**不能创建对象**，接口是更彻底的抽象，连构造器都没有，自然不能创建对象

   ```java
   public class InterfaceDemo {
       public static void main(String[] args) {
           Student s = new Student();
           s.run();
           s.rule();
       }
   }
   class Student implements Food, Person{
       @Override
       public void eat() {}
       
       @Override
       public void run() {}
   }
   interface Food{
       void eat();
   }
   interface Person{
       void run();
   }
   //可以直接 interface Person extend Food,
   //然后 class Student implements Person 效果一样
   ```

   

***



#### 新增功能

jdk1.8 以后新增的功能：

* 默认方法（就是普通实例方法）
  * 必须用 default 修饰，默认会 public 修饰
  * 必须用接口的实现类的对象来调用
  * 必须有默认实现
* 静态方法
  * 默认会 public 修饰
  * 接口的静态方法必须用接口的类名本身来调用
  * 调用格式：ClassName.method()
  * 必须有默认实现
* 私有方法：JDK 1.9 才开始有的，只能在**本类中**被其他的默认方法或者私有方法访问

```java
public class InterfaceDemo {
    public static void main(String[] args) {
        // 1.默认方法调用：必须用接口的实现类对象调用。
        Man m = new Man();
        m.run();
        m.work();

        // 2.接口的静态方法必须用接口的类名本身来调用。
        InterfaceJDK8.inAddr();
    }
}
class Man implements InterfaceJDK8 {
    @Override
    public void work() {
        System.out.println("工作中。。。");
    }
}

interface InterfaceJDK8 {
    //抽象方法！！
    void work();
    // a.默认方法（就是之前写的普通实例方法）
    // 必须用接口的实现类的对象来调用。
    default void run() {
        go();
        System.out.println("开始跑步🏃‍");
    }

    // b.静态方法
    // 注意：接口的静态方法必须用接口的类名本身来调用
    static void inAddr() {
        System.out.println("我们在武汉");
    }
    
    // c.私有方法（就是私有的实例方法）: JDK 1.9才开始有的。
    // 只能在本接口中被其他的默认方法或者私有方法访问。
    private void go() {
        System.out.println("开始。。");
    }
}
```



***



#### 对比抽象类

| **参数**           | **抽象类**                                                   | **接口**                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 默认的方法实现     | 可以有默认的方法实现                                         | 接口完全是抽象的，jdk8 以后有默认的实现                      |
| 实现               | 子类使用 **extends** 关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 | 子类使用关键字 **implements** 来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器             | 抽象类可以有构造器                                           | 接口不能有构造器                                             |
| 与正常Java类的区别 | 除了不能实例化抽象类之外，和普通 Java 类没有任何区别         | 接口是完全不同的类型                                         |
| 访问修饰符         | 抽象方法有 **public**、**protected** 和 **default** 这些修饰符 | 接口默认修饰符是 **public**，别的修饰符需要有方法体          |
| main方法           | 抽象方法可以有 main 方法并且我们可以运行它                   | jdk8 以前接口没有 main 方法，不能运行；jdk8 以后接口可以有 default 和 static 方法，可以运行 main 方法 |
| 多继承             | 抽象方法可以继承一个类和实现多个接口                         | 接口可以继承一个或多个其它接口，接口不可继承类               |
| 速度               | 比接口速度要快                                               | 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法     |
| 添加新方法         | 如果往抽象类中添加新的方法，可以给它提供默认的实现，因此不需要改变现在的代码 | 如果往接口中添加方法，那么必须改变实现该接口的类             |





------



### 多态

#### 基本介绍

多态的概念：同一个实体同时具有多种形式同一个类型的对象，执行同一个行为，在不同的状态下会表现出不同的行为特征

多态的格式：

* 父类类型范围 > 子类类型范围

```java
父类类型 对象名称 = new 子类构造器;
接口	  对象名称 = new 实现类构造器;
```

多态的执行：

* 对于方法的调用：**编译看左边，运行看右边**（分派机制）
* 对于变量的调用：**编译看左边，运行看左边**

多态的使用规则：

* 必须存在继承或者实现关系
* 必须存在父类类型的变量引用子类类型的对象
* 存在方法重写

多态的优势：
* 在多态形式下，右边对象可以实现组件化切换，便于扩展和维护，也可以实现类与类之间的**解耦**
* 父类类型作为方法形式参数，传递子类对象给方法，可以传入一切子类对象进行方法的调用，更能体现出多态的**扩展性**与便利性

多态的劣势： 
* 多态形式下，不能直接调用子类特有的功能，因为编译看左边，父类中没有子类独有的功能，所以代码在编译阶段就直接报错了

```java
public class PolymorphicDemo {
    public static void main(String[] args) {
        Animal c = new Cat();
        c.run();
        //c.eat();//报错  编译看左边 需要强转
        go(c);
        go(new Dog);   
    }
    //用 Dog或者Cat 都没办法让所有动物参与进来，只能用Anima
    public static void go(Animal d){}
    
}
class Dog extends Animal{}

class Cat extends Animal{
    public void eat();
    @Override
    public void run(){}
}

class Animal{
    public void run(){}
}
```



***



#### 上下转型

>基本数据类型的转换：
>
>1. 小范围类型的变量或者值可以直接赋值给大范围类型的变量
>2. 大范围类型的变量或者值必须强制类型转换给小范围类型的变量

引用数据类型的**自动**类型转换语法：子类类型的对象或者变量可以自动类型转换赋值给父类类型的变量

**父类引用指向子类对象**

- **向上转型 (upcasting)**：通过子类对象（小范围）实例化父类对象（大范围），这种属于自动转换
- **向下转型 (downcasting)**：通过父类对象（大范围）实例化子类对象（小范围），这种属于强制转换

```java
public class PolymorphicDemo {
    public static void main(String[] args){
        Animal a = new Cat();	// 向上转型
        Cat c = (Cat)a;			// 向下转型
    }
}
class Animal{}
class Cat extends Animal{}
```



***



#### instanceof

instanceof：判断左边的对象是否是右边的类的实例，或者是其直接或间接子类，或者是其接口的实现类

* 引用类型强制类型转换：父类类型的变量或者对象强制类型转换成子类类型的变量，否则报错
* 强制类型转换的格式：**类型 变量名称 = (类型)(对象或者变量)**
* 有继承/实现关系的两个类型就可以进行强制类型转换，编译阶段一定不报错，但是运行阶段可能出现类型转换异常 ClassCastException

```java
public class Demo{
    public static void main(String[] args){
		Aniaml a = new Dog();
		//Dog d = (Dog)a;
        //Cat c = (Cat)a; 编译不报错，运行报ClassCastException错误
        if(a instanceof Cat){
            Cat c = (Cat)a; 
        } else if(a instanceof Dog) {
            Dog d = (Dog)a;
        }
    }
}
class Dog extends Animal{}
class Cat extends Animal{}
class Animal{}
```



***



### 内部类

#### 概述

内部类是类的五大成分之一：成员变量，方法，构造器，代码块，内部类

概念：定义在一个类里面的类就是内部类

作用：提供更好的封装性，体现出组件思想，**间接解决类无法多继承引起的一系列问题**

分类：静态内部类、实例内部类（成员内部类）、局部内部类、**匿名内部类**（重点）



***



#### 静态内部类

定义：有 static 修饰，属于外部类本身，会加载一次

静态内部类中的成分研究：

* 类有的成分它都有，静态内部类属于外部类本身，只会加载一次
* 特点与外部类是完全一样的，只是位置在别人里面
* 可以定义静态成员

静态内部类的访问格式：外部类名称.内部类名称

静态内部类创建对象的格式：外部类名称.内部类名称 对象名称 = new 外部类名称.内部类构造器

静态内部类的访问拓展：

* 静态内部类中是否可以直接访问外部类的静态成员?	可以，外部类的静态成员只有一份，可以被共享
* 静态内部类中是否可以直接访问外部类的实例成员?	不可以，外部类的成员必须用外部类对象访问

```java
public class Demo{
    public static void main(String[] args){
        Outter.Inner in = new Outter.Inner();
    }
}

static class Outter{
    public static int age;
    private double salary;
    public static class Inner{
         //拥有类的所有功能 构造器 方法 成员变量
         System.out.println(age);
         //System.out.println(salary);报错
	}
}
```



***



#### 实例内部类

定义：无 static 修饰的内部类，属于外部类的每个对象，跟着外部类对象一起加载

实例内部类的成分特点：实例内部类中不能定义静态成员，其他都可以定义

实例内部类的访问格式：外部类名称.内部类名称

创建对象的格式：外部类名称.内部类名称 对象名称 = new 外部类构造器.new 内部构造器

* `Outter.Inner in = new Outter().new Inner()`

**实例内部类可以访问外部类的全部成员**

* 实例内部类中可以直接访问外部类的静态成员，外部类的静态成员可以被共享访问
* 实例内部类中可以访问外部类的实例成员，实例内部类属于外部类对象，可以直接访问外部类对象的实例成员



***



#### 局部内部类

局部内部类：定义在方法中，在构造器中，代码块中，for 循环中定义的内部类

局部内部类中的成分特点：只能定义实例成员，不能定义静态成员

```java
public class InnerClass{
	public static void main(String[] args){
        String name;
        class{}
    }
    public static void test(){
		class Animal{}
		class Cat extends Animal{}  
	}
}
```



***



#### 匿名内部类

匿名内部类：没有名字的局部内部类

匿名内部类的格式：

```java
new 类名|抽象类|接口(形参){
	//方法重写。
}
```
 匿名内部类的特点：

* 匿名内部类不能定义静态成员
* 匿名内部类一旦写出来，就会立即创建一个匿名内部类的对象返回
* **匿名内部类的对象的类型相当于是当前 new 的那个的类型的子类类型**
* 匿名内部类引用局部变量必须是**常量**，底层创建为内部类的成员变量（原因：JVM → 运行机制 → 代码优化）

```java
public class Anonymity {
    public static void main(String[] args) {
        Animal a = new Animal(){
            @Override
            public void run() {
                System.out.println("猫跑的贼溜~~");
                //System.out.println(n);
            }
        };
        a.run();
        a.go();
    }
}
abstract class Animal{
    public abstract void run();

    public void go(){
        System.out.println("开始go~~~");
    }
}
```



***



### 权限符

权限修饰符：有四种**（private -> 缺省 -> protected - > public ）**
可以修饰成员变量，修饰方法，修饰构造器，内部类，不同修饰符修饰的成员能够被访问的权限将受到限制

| 四种修饰符访问权限 | private | 缺省 | protected | public |
| ------------------ | :-----: | :--: | :-------: | :----: |
| 本类中             |    √    |  √   |     √     |   √    |
| 本包下的子类中     |    X    |  √   |     √     |   √    |
| 本包下其他类中     |    X    |  √   |     √     |   √    |
| 其他包下的子类中   |    X    |  X   |     √     |   √    |
| 其他包下的其他类中 |    X    |  X   |     X     |   √    |

protected 用于修饰成员，表示在继承体系中成员对于子类可见

* 基类的 protected 成员是包内可见的，并且对子类可见
* 若子类与基类不在同一包中，那么子类实例可以访问其从基类继承而来的 protected 方法（重写），而不能访问基类实例的 protected 方法





***



### 代码块

#### 静态代码块

静态代码块的格式：

 ```java
static {
}
 ```

* 静态代码块特点： 
  * 必须有 static 修饰，只能访问静态资源
  * 会与类一起优先加载，且自动触发执行一次
* 静态代码块作用：
  * 可以在执行类的方法等操作之前先在静态代码块中进行静态资源的初始化 
  * **先执行静态代码块，在执行 main 函数里的操作**

```java
public class CodeDemo {
    public static String schoolName ;
    public static ArrayList<String> lists = new ArrayList<>();

    // 静态代码块,属于类，与类一起加载一次!
    static {
        System.out.println("静态代码块被触发执行~~~~~~~");
        // 在静态代码块中进行静态资源的初始化操作
        schoolName = "张三";
        lists.add("3");
        lists.add("4");
        lists.add("5");
    }
    public static void main(String[] args) {
        System.out.println("main方法被执行");
        System.out.println(schoolName);
        System.out.println(lists);
    }
}
/*静态代码块被触发执行~~~~~~~
main方法被执行
张三
[3, 4, 5] */
```



***



#### 实例代码块

实例代码块的格式：

```java
{

}
```

* 实例代码块的特点：
  * 无 static 修饰，属于对象
  * 会与类的对象一起加载，每次创建类的对象的时候，实例代码块都会被加载且自动触发执行一次
  * 实例代码块的代码在底层实际上是提取到每个构造器中去执行的
  
* 实例代码块的作用：实例代码块可以在创建对象之前进行实例资源的初始化操作

```java
public class CodeDemo {
    private String name;
    private ArrayList<String> lists = new ArrayList<>();
    {
        name = "代码块";
        lists.add("java");
        System.out.println("实例代码块被触发执行一次~~~~~~~~");
    }
    public CodeDemo02(){ }//构造方法
    public CodeDemo02(String name){}

    public static void main(String[] args) {
        CodeDemo c = new CodeDemo();//实例代码块被触发执行一次
        System.out.println(c.name);
        System.out.println(c.lists);
        new CodeDemo02();//实例代码块被触发执行一次
    }
}
```





***






