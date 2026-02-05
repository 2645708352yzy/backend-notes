[[Java]]

# SE

## 异常

### 基本介绍

异常：程序在编译或者执行的过程中可能出现的问题，Java 为常见的代码异常都设计一个类来代表

错误：Error ，程序员无法处理的错误，只能重启系统，比如内存奔溃，JVM 本身的奔溃

Java 中异常继承的根类是：Throwable

```
异常的体系:
         Throwable(根类，不是异常类)
      /              \
    Error           Exception（异常，需要研究和处理）
                    /            \
                   编译时异常     RuntimeException(运行时异常)
```

Exception 异常的分类:

* 编译时异常：继承自 Exception 的异常或者其子类，编译阶段就会报错
* 运行时异常：继承自 RuntimeException 的异常或者其子类，编译阶段是不会出错的，在运行阶段出错



***



### 处理过程

异常的产生默认的处理过程解析：（自动处理的过程）

1. 默认会在出现异常的代码那里自动的创建一个异常对象：ArithmeticException（算术异常）
2. 异常会从方法中出现的点这里抛出给调用者，调用者最终抛出给 JVM 虚拟机
3. 虚拟机接收到异常对象后，先在控制台直接输出**异常栈**信息数据
4. 直接从当前执行的异常点终止当前程序
5. 后续代码没有机会执行了，因为程序已经死亡

```java
public class ExceptionDemo {
    public static void main(String[] args) {
        System.out.println("程序开始。。。。。。。。。。");
        chu( 10 ,0 );
        System.out.println("程序结束。。。。。。。。。。");//不执行
    }
    public static void chu(int a , int b){
        int c = a / b ;// 出现了运行时异常,自动创建异常对象：ArithmeticException
        System.out.println("结果是："+c);
    }
}
```



***



### 编译异常

#### 基本介绍

编译时异常：继承自 Exception 的异常或者其子类，没有继承 RuntimeException，编译时异常是编译阶段就会报错

编译时异常的作用是什么：在编译阶段就爆出一个错误，目的在于提醒，请检查并注意不要出 BUG

```java
public static void main(String[] args) throws ParseException {
	String date = "2015-01-12 10:23:21";
	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	Date d = sdf.parse(date);
	System.out.println(d);
}
```



****



#### 处理机制

##### throws

在出现编译时异常的地方层层把异常抛出去给调用者，调用者最终抛出给 JVM 虚拟机，JVM 虚拟机输出异常信息，直接终止掉程序，这种方式与默认方式是一样的

**Exception 是异常最高类型可以抛出一切异常**

```java
public static void main(String[] args) throws Exception {
    System.out.println("程序开始。。。。");
    String s = "2013-03-23 10:19:23";
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Date date = sdf.parse(s);
    System.out.println("程序结束。。。。。");
}
```



***



##### try/catch

可以处理异常，并且出现异常后代码也不会死亡

* 捕获异常和处理异常的格式：**捕获处理**

  ```java
  try{
    // 监视可能出现异常的代码！
  }catch(异常类型1 变量){
    // 处理异常
  }catch(异常类型2 变量){
    // 处理异常
  }...finall{
  //资源释放
  }
  ```

* 监视捕获处理异常写法：Exception 可以捕获处理一切异常类型
  
  ```java
  try{
      // 可能出现异常的代码！
  }catch (Exception e){
      e.printStackTrace(); // **直接打印异常栈信息**
  }
  ```

**Throwable成员方法:**

* `public String getMessage()`：返回此 throwable 的详细消息字符串
* `public String toString()`：返回此可抛出的简短描述
* `public void printStackTrace()`：把异常的错误信息输出在控制台

```java
public static void main(String[] args) {
    System.out.println("程序开始。。。。");
    try {
        String s = "2013-03-23 10:19:23";
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date date = sdf.parse(s);
        InputStream is = new FileInputStream("D:/meinv.png");
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("程序结束。。。。。");
}
```



***



##### 规范做法

在出现异常的地方把异常一层一层的抛出给最外层调用者，最外层调用者集中捕获处理

```java
public class ExceptionDemo{
	public static void main(String[] args){
        System.out.println("程序开始。。。。");
        try {
            parseDate("2013-03-23 10:19:23");
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("程序结束。。。。");
    }
    public static void parseDate(String time) throws Exception{...}
}
```



***



### 运行异常

#### 基本介绍

继承自 RuntimeException 的异常或者其子类，编译阶段是不会出错的，是在运行时阶段可能出现的错误，运行时异常编译阶段可以处理也可以不处理，代码编译都能通过

**常见的运行时异常**：

1. 数组索引越界异常：ArrayIndexOutOfBoundsException
2. 空指针异常：NullPointerException，直接输出没问题，调用空指针的变量的功能就会报错
3. 类型转换异常：ClassCastException
4. 迭代器遍历没有此元素异常：NoSuchElementException
5. 算术异常（数学操作异常）：ArithmeticException
6. 数字转换异常：NumberFormatException



****



#### 处理机制

运行时异常在编译阶段是不会报错，在运行阶段才会出错，运行时出错了程序还是会停止，运行时异常也建议要处理，运行时异常是自动往外抛出的，不需要手工抛出

**运行时异常的处理规范**：直接在最外层捕获处理即可，底层会自动抛出

```java
public class ExceptionDemo{
    public static void main(String[] args){
        System.out.println("程序开始。。。。");
        try{
            chu(10 / 0);//ArithmeticException: / by zero
            System.out.println("操作成功！");//没输出
        }catch (Exception e){
            e.printStackTrace();
            System.out.println("操作失败！");//输出了
        }
        System.out.println("程序结束。。。。");//输出了
    }
    
    public static void chu(int a , int b)  { System.out.println( a / b );}
}
```



***



### Finally

用在捕获处理的异常格式中的，放在最后面

```java
try{
    // 可能出现异常的代码！
}catch(Exception e){
    e.printStackTrace();
}finally{
    // 无论代码是出现异常还是正常执行，最终一定要执行这里的代码！！
}
try: 1次。
catch：0-N次  (如果有finally那么catch可以没有!!)
finally: 0-1次
```

**finally 的作用**：可以在代码执行完毕以后进行资源的释放操作

资源：资源都是实现了 Closeable 接口的，都自带 close() 关闭方法

注意：如果在 finally 中出现了 return，会吞掉异常

```java
public class FinallyDemo {
    public static void main(String[] args) {
        System.out.println(chu());//一定会输出 finally,优先级比return高
    }

    public static int chu(){
        try{
            int a = 10 / 2 ;
            return a ;
        }catch (Exception e){
            e.printStackTrace();
            return -1;
        }finally {
            System.out.println("=====finally被执行");
            //return 111; // 不建议在finally中写return，会覆盖前面所有的return值!
        }
    }
    public static void test(){
        InputStream is = null;
        try{
            is = new FileInputStream("D:/cang.png");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println("==finally被执行===");
            // 回收资源。用于在代码执行完毕以后进行资源的回收操作！
            try {
                if(is!=null)is.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```



***



### 自定义

自定义异常:

* 自定义编译时异常：定义一个异常类继承 Exception，重写构造器，在出现异常的地方用 throw new 自定义对象抛出
* 自定义运行时异常：定义一个异常类继承 RuntimeException，重写构造器，在出现异常的地方用 throw new 自定义对象抛出

throws：用在方法上，用于抛出方法中的异常

throw:  用在出现异常的地方，创建异常对象且立即从此处抛出

```java
//需求：认为年龄小于0岁，大于200岁就是一个异常。
public class ExceptionDemo {
    public static void main(String[] args) {
        try {
            checkAge(101);
        } catch (AgeIllegalException e) {
            e.printStackTrace();
        }
    }

    public static void checkAge(int age) throws ItheimaAgeIllegalException {
        if(age < 0 || age > 200){//年龄在0-200之间
            throw new AgeIllegalException("/ age is illegal!");
            //throw new AgeIllegalRuntimeException("/ age is illegal!");
        }else{
            System.out.println("年龄是：" + age);
        }
    }
}

public class AgeIllegalException extends Exception{
    Alt + Insert->Constructor 
}//编译时异常
public class AgeIllegalRuntimeException extends RuntimeException{
	public AgeIllegalRuntimeException() {
    }

    public AgeIllegalRuntimeException(String message) {
        super(message);
    }
}//运行时异常
```



***



### 处理规范

异常的语法注意：

1. 运行时异常被抛出可以不处理，可以自动抛出；**编译时异常必须处理**；按照规范都应该处理
2. **重写方法申明抛出的异常，子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型**
3. 方法默认都可以自动抛出运行时异常， throws RuntimeException 可以省略不写
4. 当多异常处理时，捕获处理，前面的异常类不能是后面异常类的父类
5. 在 try/catch 后可以追加 finally 代码块，其中的代码一定会被执行，通常用于资源回收操作

异常的作用：

1. 可以处理代码问题，防止程序出现异常后的死亡

2. 提高了程序的健壮性和安全性

```java
public class Demo{
    public static void main(String[] args){
        //请输入一个合法的年龄
        while(true){
            try{
                Scanner sc = new Scanner(System.in);
                System.out.println("请您输入您的年年龄：");
                int age = sc.nextInt();
                System.out.println("年龄："+age);
                break;
            }catch(Exception e){
                System.err.println("您的年龄是瞎输入的！");
            }
        }
    }
}
```





***





