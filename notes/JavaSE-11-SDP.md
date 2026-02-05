[[Java]]

# SE

## SDP

### 单例模式

#### 基本介绍

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一，提供了一种创建对象的最佳方式

单例设计模式分类两种：

* 饿汉式：类加载就会导致该单实例对象被创建	

* 懒汉式：类加载不会导致该单实例对象被创建，而是首次使用该对象时才会创建



***



#### 饿汉式

饿汉式在类加载的过程导致该单实例对象被创建，**虚拟机会保证类加载的线程安全**，但是如果只是为了加载该类不需要实例，则会造成内存的浪费

* 静态变量的方式：

  ```java
  public final class Singleton {
      // 私有构造方法
      private Singleton() {}
      // 在成员位置创建该类的对象
      private static final Singleton instance = new Singleton();
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          return instance;
      }
      
      // 解决序列化问题
      protected Object readResolve() {
      	return INSTANCE;
      }
  }
  ```

  * 加 final 修饰，所以不会被子类继承，防止子类中不适当的行为覆盖父类的方法，破坏了单例

  * 防止反序列化破坏单例的方式：

    * 对单例声明 transient，然后实现 readObject(ObjectInputStream in) 方法，复用原来的单例

      条件：访问权限为 private/protected、返回值必须是 Object、异常可以不抛

    * 实现 readResolve() 方法，当 JVM 从内存中反序列化地组装一个新对象，就会自动调用 readResolve 方法返回原来单例

  * 构造方法设置为私有，防止其他类无限创建对象，但是不能防止反射破坏

  * 静态变量初始化在类加载时完成，**由 JVM 保证线程安全**，能保证单例对象创建时的安全

  * 提供静态方法而不是直接将 INSTANCE 设置为 public，体现了更好的封装性、提供泛型支持、可以改进成懒汉单例设计

* 静态代码块的方式：

  ```java
  public class Singleton {
      // 私有构造方法
      private Singleton() {}
      
      // 在成员位置创建该类的对象
      private static Singleton instance;
      static {
          instance = new Singleton();
      }
      
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          return instance;
      }
  }
  ```

* 枚举方式：枚举类型是所用单例实现中**唯一一种不会被破坏**的单例实现模式

  ```java
  public enum Singleton {
      INSTANCE;
      public void doSomething() {
          System.out.println("doSomething");
      }
  }
  public static void main(String[] args) {
      Singleton.INSTANCE.doSomething();
  }
  ```

  * 问题1：枚举单例是如何限制实例个数的？每个枚举项都是一个实例，是一个静态成员变量
  * 问题2：枚举单例在创建时是否有并发问题？否
  * 问题3：枚举单例能否被反射破坏单例？否，反射创建对象时判断是枚举类型就直接抛出异常
  * 问题4：枚举单例能否被反序列化破坏单例？否
  * 问题5：枚举单例属于懒汉式还是饿汉式？**饿汉式**
  * 问题6：枚举单例如果希望加入一些单例创建时的初始化逻辑该如何做？添加构造方法

  反编译结果：

  ```java
  public final class Singleton extends java.lang.Enum<Singleton> { // Enum实现序列化接口
  	public static final Singleton INSTANCE = new Singleton();
  }
  ```

  



***



#### 懒汉式

* 线程不安全

  ```java
  public class Singleton {
      // 私有构造方法
      private Singleton() {}
  
      // 在成员位置创建该类的对象
      private static Singleton instance;
  
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          if(instance == null) {
              // 多线程环境，会出现线程安全问题，可能多个线程同时进入这里
              instance = new Singleton();
          }
          return instance;
      }
  }
  ```

* 双端检锁机制

  在多线程的情况下，可能会出现空指针问题，出现问题的原因是 JVM 在实例化对象的时候会进行优化和指令重排序操作，所以需要使用 `volatile` 关键字

  ```java
  public class Singleton { 
      // 私有构造方法
      private Singleton() {}
      private static volatile Singleton instance;
  
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          // 第一次判断，如果instance不为null，不进入抢锁阶段，直接返回实例
          if(instance == null) {
              synchronized (Singleton.class) {
                  // 抢到锁之后再次判断是否为null
                  if(instance == null) {
                      instance = new Singleton();
                  }
              }
          }
          return instance;
      }
  }
  ```

* 静态内部类方式

  ```java
  public class Singleton {
      // 私有构造方法
      private Singleton() {}
  
      private static class SingletonHolder {
          private static final Singleton INSTANCE = new Singleton();
      }
  
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          return SingletonHolder.INSTANCE;
      }
  }
  ```

  * 内部类属于懒汉式，类加载本身就是懒惰的，首次调用时加载，然后对单例进行初始化

    类加载的时候方法不会被调用，所以不会触发 getInstance 方法调用 invokestatic 指令对内部类进行加载；加载的时候字节码常量池会被加入类的运行时常量池，解析工作是将常量池中的符号引用解析成直接引用，但是解析过程不一定非得在类加载时完成，可以延迟到运行时进行，所以静态内部类实现单例会**延迟加载**

  * 没有线程安全问题，静态变量初始化在类加载时完成，由 JVM 保证线程安全



***



#### 破坏单例

##### 反序列化

将单例对象序列化再反序列化，对象从内存反序列化到程序中会重新创建一个对象，通过反序列化得到的对象是不同的对象，而且得到的对象不是通过构造器得到的，**反序列化得到的对象不执行构造器**

* Singleton

  ```java
  public class Singleton implements Serializable {	//实现序列化接口
      // 私有构造方法
      private Singleton() {}
      private static class SingletonHolder {
          private static final Singleton INSTANCE = new Singleton();
      }
  
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          return SingletonHolder.INSTANCE;
      }
  }
  ```

* 序列化

  ```java
  public class Test {
      public static void main(String[] args) throws Exception {
          //往文件中写对象
          //writeObject2File();
          //从文件中读取对象
          Singleton s1 = readObjectFromFile();
          Singleton s2 = readObjectFromFile();
          //判断两个反序列化后的对象是否是同一个对象
          System.out.println(s1 == s2);
      }
  
      private static Singleton readObjectFromFile() throws Exception {
          //创建对象输入流对象
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream("C://a.txt"));
          //第一个读取Singleton对象
          Singleton instance = (Singleton) ois.readObject();
          return instance;
      }
      
      public static void writeObject2File() throws Exception {
          //获取Singleton类的对象
          Singleton instance = Singleton.getInstance();
          //创建对象输出流
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("C://a.txt"));
          //将instance对象写出到文件中
          oos.writeObject(instance);
      }
  }
  ```

* 解决方法：

  在 Singleton 类中添加 `readResolve()` 方法，在反序列化时被反射调用，如果定义了这个方法，就返回这个方法的值，如果没有定义，则返回新创建的对象

  ```java
  private Object readResolve() {
      return SingletonHolder.INSTANCE;
  }
  ```

  ObjectInputStream 类源码分析：

  ```java
  public final Object readObject() throws IOException, ClassNotFoundException{
      //...
  	Object obj = readObject0(false);//重点查看readObject0方法
  }
  
  private Object readObject0(boolean unshared) throws IOException {
      try {
  		switch (tc) {
  			case TC_OBJECT:
  				return checkResolve(readOrdinaryObject(unshared));
          }
      } 
  }
  private Object readOrdinaryObject(boolean unshared) throws IOException {
  	// isInstantiable 返回true，执行 desc.newInstance()，通过反射创建新的单例类
      obj = desc.isInstantiable() ? desc.newInstance() : null; 
      // 添加 readResolve 方法后 desc.hasReadResolveMethod() 方法执行结果为true
      if (obj != null && handles.lookupException(passHandle) == null && desc.hasReadResolveMethod()) {
      	// 通过反射调用 Singleton 类中的 readResolve 方法，将返回值赋值给rep变量
      	// 多次调用ObjectInputStream类中的readObject方法，本质调用定义的readResolve方法，返回的是同一个对象。
      	Object rep = desc.invokeReadResolve(obj);
      }
      return obj;
  }
  ```



***



##### 反射破解

* 反射

  ```java
  public class Test {
      public static void main(String[] args) throws Exception {
          //获取Singleton类的字节码对象
          Class clazz = Singleton.class;
          //获取Singleton类的私有无参构造方法对象
          Constructor constructor = clazz.getDeclaredConstructor();
          //取消访问检查
          constructor.setAccessible(true);
  
          //创建Singleton类的对象s1
          Singleton s1 = (Singleton) constructor.newInstance();
          //创建Singleton类的对象s2
          Singleton s2 = (Singleton) constructor.newInstance();
  
          //判断通过反射创建的两个Singleton对象是否是同一个对象
          System.out.println(s1 == s2);	//false
      }
  }
  ```

* 反射方式破解单例的解决方法：

  ```java
  public class Singleton {
      private static volatile Singleton instance;
      
      // 私有构造方法
      private Singleton() {
          // 反射破解单例模式需要添加的代码
          if(instance != null) {
              throw new RuntimeException();
          }
      }
      
      // 对外提供静态方法获取该对象
      public static Singleton getInstance() {
          if(instance != null) {
              return instance;
          }
          synchronized (Singleton.class) {
              if(instance != null) {
                  return instance;
              }
              instance = new Singleton();
              return instance;
          }
      }
  }
  ```





***



#### Runtime

Runtime 类就是使用的单例设计模式中的饿汉式

```java
public class Runtime {    
    private static Runtime currentRuntime = new Runtime();    
    public static Runtime getRuntime() {        
        return currentRuntime;    
    }   
    private Runtime() {}    
    ...
}
```

使用 Runtime

```java
public class RuntimeDemo {
    public static void main(String[] args) throws IOException {
        //获取Runtime类对象
        Runtime runtime = Runtime.getRuntime();

        //返回 Java 虚拟机中的内存总量。
        System.out.println(runtime.totalMemory());
        //返回 Java 虚拟机试图使用的最大内存量。
        System.out.println(runtime.maxMemory());

        //创建一个新的进程执行指定的字符串命令，返回进程对象
        Process process = runtime.exec("ipconfig");
        //获取命令执行后的结果，通过输入流获取
        InputStream inputStream = process.getInputStream();
        byte[] arr = new byte[1024 * 1024* 100];
        int b = inputStream.read(arr);
        System.out.println(new String(arr,0,b,"gbk"));
    }
}
```





****



### 代理模式

#### 静态代理

代理模式：由于某些原因需要给某对象提供一个代理以控制对该对象的访问，访问对象不适合或者不能直接引用为目标对象，代理对象作为访问对象和目标对象之间的中介

Java 中的代理按照代理类生成时机不同又分为静态代理和动态代理，静态代理代理类在编译期就生成，而动态代理代理类则是在 Java 运行时动态生成，动态代理又有 JDK 代理和 CGLib 代理两种

代理（Proxy）模式分为三种角色：

* 抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法
* 真实主题（Real Subject）类： 实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象
* 代理（Proxy）类：提供了与真实主题相同的接口，其内部含有对真实主题的引用，可以访问、控制或扩展真实主题的功能

买票案例，火车站是目标对象，代售点是代理对象

* 卖票接口：

  ```java
  public interface SellTickets {
      void sell();
  }
  ```

* 火车站，具有卖票功能，需要实现SellTickets接口

  ```java
  public class TrainStation implements SellTickets {
      public void sell() {
          System.out.println("火车站卖票");
      }
  }
  ```

* 代售点：

  ```java
  public class ProxyPoint implements SellTickets {
      private TrainStation station = new TrainStation();
  
      public void sell() {
          System.out.println("代理点收取一些服务费用");
          station.sell();
      }
  }
  ```

* 测试类：

  ```java
  public class Client {
      public static void main(String[] args) {
          ProxyPoint pp = new ProxyPoint();
          pp.sell();
      }
  }
  ```

  测试类直接访问的是 ProxyPoint 类对象，也就是 ProxyPoint 作为访问对象和目标对象的中介



****



#### JDK

##### 使用方式

Java 中提供了一个动态代理类 Proxy，Proxy 并不是代理对象的类，而是提供了一个创建代理对象的静态方法 newProxyInstance() 来获取代理对象

`static Object newProxyInstance(ClassLoader loader,Class[] interfaces,InvocationHandler h) `

* 参数一：类加载器，负责加载代理类。传入类加载器，代理和被代理对象要用一个类加载器才是父子关系，不同类加载器加载相同的类在 JVM 中都不是同一个类对象

* 参数二：被代理业务对象的**全部实现的接口**，代理对象与真实对象实现相同接口，知道为哪些方法做代理

* 参数三：代理真正的执行方法，也就是代理的处理逻辑

代码实现：

* 代理工厂：创建代理对象

  ```java
  public class ProxyFactory {
      private TrainStation station = new TrainStation();
  	//也可以在参数中提供 getProxyObject(TrainStation station)
      public SellTickets getProxyObject() {
          //使用 Proxy 获取代理对象
          SellTickets sellTickets = (SellTickets) Proxy.newProxyInstance(
              	station.getClass().getClassLoader(),
                  station.getClass().getInterfaces(),
                  new InvocationHandler() {
                      public Object invoke(Object proxy, Method method, Object[] args) {
                          System.out.println("代理点(JDK动态代理方式)");
                          //执行真实对象
                          Object result = method.invoke(station, args);
                          return result;
                      }
                  });
          return sellTickets;
      }
  }
  ```

* 测试类：

  ```java
  public class Client {
      public static void main(String[] args) {
          //获取代理对象
          ProxyFactory factory = new ProxyFactory();
          //必须时代理ji
          SellTickets proxyObject = factory.getProxyObject();
          proxyObject.sell();
      }
  }
  ```



***



##### 实现原理

JDK 动态代理方式的优缺点：

- 优点：可以为任意的接口实现类对象做代理，也可以为被代理对象的所有接口的所有方法做代理，动态代理可以在不改变方法源码的情况下，实现对方法功能的增强，提高了软件的可扩展性，Java 反射机制可以生成任意类型的动态代理类
- 缺点：**只能针对接口或者接口的实现类对象做代理对象**，普通类是不能做代理对象的
- 原因：**生成的代理类继承了 Proxy**，Java 是单继承的，所以 JDK 动态代理只能代理接口

ProxyFactory 不是代理模式中的代理类，而代理类是程序在运行过程中动态的在内存中生成的类，可以通过 Arthas 工具查看代理类结构：

* 代理类（$Proxy0）实现了 SellTickets 接口，真实类和代理类实现同样的接口
* 代理类（$Proxy0）将提供了的匿名内部类对象传递给了父类
* 代理类（$Proxy0）的修饰符是 public final

```java
// 程序运行过程中动态生成的代理类
public final class $Proxy0 extends Proxy implements SellTickets {
    private static Method m3;

    public $Proxy0(InvocationHandler invocationHandler) {
        super(invocationHandler);//InvocationHandler对象传递给父类
    }

    static {
        m3 = Class.forName("proxy.dynamic.jdk.SellTickets").getMethod("sell", new Class[0]);
    }

    public final void sell() {
        // 调用InvocationHandler的invoke方法
        this.h.invoke(this, m3, null);
    }
}

// Java提供的动态代理相关类
public class Proxy implements java.io.Serializable {
	protected InvocationHandler h;
	 
	protected Proxy(InvocationHandler h) {
        this.h = h;
    }
}
```

执行流程如下：

1. 在测试类中通过代理对象调用 sell() 方法
2. 根据多态的特性，执行的是代理类（$Proxy0）中的 sell() 方法
3. 代理类（$Proxy0）中的 sell() 方法中又调用了 InvocationHandler 接口的子实现类对象的 invoke 方法
4. invoke 方法通过反射执行了真实对象所属类（TrainStation）中的 sell() 方法



****



##### 源码解析

```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h){
    // InvocationHandler 为空则抛出异常
    Objects.requireNonNull(h);

    // 复制一份 interfaces
    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    // 从缓存中查找 class 类型的代理对象，会调用 ProxyClassFactory#apply 方法
    Class<?> cl = getProxyClass0(loader, intfs);
	//proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory())
 
    try {
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }

        // 获取代理类的构造方法，根据参数 InvocationHandler 匹配获取某个构造器
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        // 构造方法不是 pubic 的需要启用权限，暴力p
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    // 设置可访问的权限
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
       	// cons 是构造方法，并且内部持有 InvocationHandler，在 InvocationHandler 中持有 target 目标对象
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {}
}
```

Proxy 的静态内部类：

```java
private static final class ProxyClassFactory {
    // 代理类型的名称前缀
    private static final String proxyClassNamePrefix = "$Proxy";

    // 生成唯一数字使用，结合上面的代理类型名称前缀一起生成
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

	//参数一：Proxy.newInstance 时传递的
    //参数二：Proxy.newInstance 时传递的接口集合
    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
		
        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        // 遍历接口集合
        for (Class<?> intf : interfaces) {
            Class<?> interfaceClass = null;
            try {
                // 加载接口类到 JVM
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            // 如果 interfaceClass 不是接口 直接报错，保证集合内都是接口
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            // 保证接口 interfaces 集合中没有重复的接口
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }

        // 生成的代理类的包名
        String proxyPkg = null;   
        // 【生成的代理类访问修饰符 public final】 
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        // 检查接口集合内的接口，看看有没有某个接口的访问修饰符不是 public 的  如果不是 public 的接口，
        // 生成的代理类 class 就必须和它在一个包下，否则访问出现问题
        for (Class<?> intf : interfaces) {
            // 获取访问修饰符
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                // 获取当前接口的全限定名 包名.类名
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                // 获取包名
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        // 获取唯一的编号
        long num = nextUniqueNumber.getAndIncrement();
        // 包名+ $proxy + 数字，比如 $proxy1
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        // 【生成二进制字节码，这个字节码写入到文件内】，就是编译好的 class 文件
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
        try {
            // 【使用加载器加载二进制到 jvm】，并且返回 class
            return defineClass0(loader, proxyName, proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) { }
    }
}
```





***



#### CGLIB

CGLIB 是一个功能强大，高性能的代码生成包，为没有实现接口的类提供代理，为 JDK 动态代理提供了补充（$$Proxy）

* CGLIB 是第三方提供的包，所以需要引入 jar 包的坐标：

  ```xml
  <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib</artifactId>
      <version>2.2.2</version>
  </dependency>
  ```

* 代理工厂类：

  ```java
  public class ProxyFactory implements MethodInterceptor {
      private TrainStation target = new TrainStation();
  
      public TrainStation getProxyObject() {
          //创建Enhancer对象，类似于JDK动态代理的Proxy类，下一步就是设置几个参数
          Enhancer enhancer = new Enhancer();
          //设置父类的字节码对象
          enhancer.setSuperclass(target.getClass());
          //设置回调函数
          enhancer.setCallback(new MethodInterceptor() {
              @Override
              public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
  				System.out.println("代理点收取一些服务费用(CGLIB动态代理方式)");
          		Object o = methodProxy.invokeSuper(obj, args);
          		return null;//因为返回值为void
              }
          });
          //创建代理对象
          TrainStation obj = (TrainStation) enhancer.create();
          return obj;
      }
  }
  ```

CGLIB 的优缺点

* 优点：
  * CGLIB 动态代理**不限定**是否具有接口，可以对任意操作进行增强
  * CGLIB 动态代理无需要原始被代理对象，动态创建出新的代理对象
  * **JDKProxy 仅对接口方法做增强，CGLIB 对所有方法做增强**，包括 Object 类中的方法，toString、hashCode 等
* 缺点：CGLIB 不能对声明为 final 的类或者方法进行代理，因为 CGLIB 原理是**动态生成被代理类的子类，继承被代理类**





****



#### 方式对比

三种方式对比：

* 动态代理和静态代理：

  * 动态代理将接口中声明的所有方法都被转移到一个集中的方法中处理（InvocationHandler.invoke），在接口方法数量比较多的时候，可以进行灵活处理，不需要像静态代理那样每一个方法进行中转

  * 静态代理是在编译时就已经将接口、代理类、被代理类的字节码文件确定下来
  * 动态代理是程序**在运行后通过反射创建字节码文件**交由 JVM 加载

* JDK 代理和 CGLIB 代理：

  JDK 动态代理采用 `ProxyGenerator.generateProxyClass()` 方法在运行时生成字节码；CGLIB 底层采用 ASM 字节码生成框架，使用字节码技术生成代理类。在 JDK1.6之前比使用 Java 反射效率要高，到 JDK1.8 的时候，JDK 代理效率高于 CGLIB 代理。所以如果有接口或者当前类就是接口使用 JDK 动态代理，如果没有接口使用 CGLIB 代理

代理模式的优缺点：

* 优点：
  * 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用
  * **代理对象可以增强目标对象的功能，被用来间接访问底层对象，与原始对象具有相同的 hashCode**
  * 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度

* 缺点：增加了系统的复杂度

代理模式的使用场景：

* 远程（Remote）代理：本地服务通过网络请求远程服务，需要实现网络通信，处理其中可能的异常。为了良好的代码设计和可维护性，将网络通信部分隐藏起来，只暴露给本地服务一个接口，通过该接口即可访问远程服务提供的功能

* 防火墙（Firewall）代理：当你将浏览器配置成使用代理功能时，防火墙就将你的浏览器的请求转给互联网，当互联网返回响应时，代理服务器再把它转给你的浏览器

* 保护（Protect or Access）代理：控制对一个对象的访问，如果需要，可以给不同的用户提供不同级别的使用权限







***





# JVM

