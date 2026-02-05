[[Java动态代理]]

## 实现思路

要完全自己实现动态代理，我们需要：

1. 定义一个`MyInvocationHandler`接口
2. 创建`MyProxy`类，提供静态方法来生成代理对象
3. **动态生成代理类的字节码**（这是关键部分）
4. 加载生成的字节码并创建代理对象实例

我们将使用Java的`ClassLoader`和字节码操作库来实现这一功能。我选择使用`javassist`库，它是一个流行的Java字节码操作工具，相对容易使用。

## 实现原理解析

这个实现完全不依赖Java内置的`Proxy`类，而是使用Javassist库动态生成代理类的字节码。主要步骤如下：

1. **定义接口**：创建`MyInvocationHandler`接口，类似于Java的`InvocationHandler`。

2. **动态生成代理类**：
   - 使用Javassist创建一个新的类
   - 让这个类实现所有指定的接口
   - 为接口中的每个方法生成代理方法
   - 在代理方法中，调用`MyInvocationHandler`的`invoke`方法

3. **处理方法调用**：
   - 在每个代理方法中，创建对应的`Method`对象
   - 将方法调用委托给`MyInvocationHandler`
   - 处理返回值，特别是基本类型的转换

4. **创建代理实例**：
   - 将生成的字节码加载到JVM中
   - 创建代理类的实例并返回

        

**核心功能**：

`MyProxy` 的核心功能是 `newProxyInstance` 方法，它能够为一个或多个接口创建代理对象。当你调用代理对象的方法时，实际上会调用你提供的 `MyInvocationHandler` 的 `invoke` 方法，从而允许你在方法调用前后执行自定义逻辑（例如日志记录、权限检查、事务管理等）。

**代码结构和逻辑详解**：

1.  **包和导入 (Package and Imports)**:
    ```java
    package com.myproxy;
    
    import javassist.*;
    import java.lang.reflect.Constructor;
    import java.lang.reflect.Method;
    import java.util.concurrent.atomic.AtomicLong;
    ```
    *   `com.myproxy`: 定义了类的包名。
    *   `javassist.*`: 导入 Javassist 库的相关类，用于动态创建和修改类。
    *   `java.lang.reflect.*`: 导入 Java 反射相关的类，用于获取方法、构造函数等信息。
    *   `java.util.concurrent.atomic.AtomicLong`: 用于生成唯一的代理类名后缀，确保线程安全。

2.  **类定义和静态字段 (Class Definition and Static Field)**:
    
    ```java
    public class MyProxy {
        // 用于生成唯一的代理类名
        private static final AtomicLong proxyClassCounter = new AtomicLong(0);
        // ...
    }
    ```
    *   `proxyClassCounter`: 这是一个原子长整型变量，每次创建代理类时，它都会自增，用于生成一个唯一的代理类名，避免命名冲突。例如生成的类名可能是 `$MyProxy0`, `$MyProxy1` 等。
    
3.  **`newProxyInstance` 方法**:
    这是创建代理实例的核心静态方法。
    
    ```java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          final MyInvocationHandler handler) throws Exception {
        // ...
    }
    ```
    *   **参数**:
        *   `loader`: `ClassLoader` 对象，用于加载生成的代理类。
        *   `interfaces`: `Class<?>[]` 数组，代理类需要实现的接口列表。代理对象将会实现这里指定的所有接口。
        *   `handler`: `MyInvocationHandler` 实例。这是一个自定义的接口（项目中应该还有一个 `MyInvocationHandler.java` 文件），当代理对象的方法被调用时，`handler` 的 `invoke` 方法会被执行。
    
    *   **参数检查**:
        ```java
        if (loader == null || interfaces == null || handler == null) {
            throw new IllegalArgumentException("参数不能为null");
        }
        if (interfaces.length == 0) {
            throw new IllegalArgumentException("至少需要一个接口");
        }
        ```
        确保传入的参数不为 `null`，并且至少指定一个接口。
    
    *   **生成代理类名**:
        ```java
        String proxyClassName = "com.myproxy.generated.$MyProxy" + proxyClassCounter.getAndIncrement();
        ```
        生成一个唯一的代理类全名，例如 `com.myproxy.generated.$MyProxy0`。
    
    *   **创建 `ClassPool` 和 `CtClass`**:
        
        ```java
        ClassPool classPool = ClassPool.getDefault();
        CtClass proxyClass = classPool.makeClass(proxyClassName);
        ```
        *   `ClassPool`: Javassist 中用于管理 `CtClass` 对象的容器。`ClassPool.getDefault()` 获取一个默认的类池。
        *   `CtClass`: Javassist 中代表一个类的对象。`classPool.makeClass(proxyClassName)` 创建一个新的 `CtClass` 对象，代表我们将要动态生成的代理类。
        
    *   **添加 `handler` 字段**:
        
        ```java
        CtField handlerField = CtField.make("private com.myproxy.MyInvocationHandler handler;", proxyClass);
        proxyClass.addField(handlerField);
        ```
        向动态生成的代理类中添加一个私有字段 `handler`，类型为 `com.myproxy.MyInvocationHandler`。这个字段将保存传入的 `handler` 实例。
        
    *   **添加构造函数**:
        ```java
        CtConstructor constructor = new CtConstructor(new CtClass[]{classPool.get("com.myproxy.MyInvocationHandler")}, proxyClass);
        constructor.setBody("{ this.handler = $1; }");
        proxyClass.addConstructor(constructor);
        ```
        为代理类添加一个构造函数，该构造函数接收一个 `MyInvocationHandler` 类型的参数，并将其赋值给代理类内部的 `handler` 字段。
        *   `new CtClass[]{classPool.get("com.myproxy.MyInvocationHandler")}`: 定义构造函数的参数类型。
        *   `"{ this.handler = $1; }"`: 构造函数体。`$1` 表示构造函数的第一个参数。
    
    *   **实现接口并生成代理方法 (核心逻辑)**:
        ```java
        for (Class<?> interfaceClass : interfaces) {
            proxyClass.addInterface(classPool.get(interfaceClass.getName()));
        
            for (Method method : interfaceClass.getMethods()) {
                // ... (方法生成逻辑)
            }
        }
        ```
        这部分代码遍历所有需要实现的接口，并为每个接口中的每个方法生成对应的代理方法。
    
        *   `proxyClass.addInterface(classPool.get(interfaceClass.getName()));`: 让代理类实现当前遍历到的接口。
    
        *   **为接口中的每个方法创建代理方法**:
            ```java
            // 创建方法参数类型数组
            CtClass[] paramTypes = new CtClass[method.getParameterTypes().length];
            for (int i = 0; i < paramTypes.length; i++) {
                paramTypes[i] = classPool.get(method.getParameterTypes()[i].getName());
            }
            
            // 创建方法
            CtMethod ctMethod = new CtMethod(
                    classPool.get(method.getReturnType().getName()), // 返回类型
                    method.getName(),                               // 方法名
                    paramTypes,                                     // 参数类型
                    proxyClass                                      // 所属类
            );
            ```
            这里使用 Javassist 的 `CtMethod` 来定义新方法。它需要指定返回类型、方法名、参数类型和该方法所属的 `CtClass`。
    
        *   **生成方法体 (Method Body Generation)**:
            这是动态代理最复杂的部分，它构造了代理方法内部的 Java 代码字符串。
            ```java
            StringBuilder methodBody = new StringBuilder();
            methodBody.append("{\n");
            
            // 1. 获取被代理方法的 Method 对象
            methodBody.append("    java.lang.reflect.Method method = ")
                    .append(interfaceClass.getName())
                    .append(".class.getMethod(\"")
                    .append(method.getName())
                    .append("\", new Class[] {");
            // 添加参数类型
            for (int i = 0; i < method.getParameterTypes().length; i++) {
                if (i > 0) {
                    methodBody.append(", ");
                }
                methodBody.append(method.getParameterTypes()[i].getName()).append(".class");
            }
            methodBody.append("});\n");
            
            // 2. 创建参数数组 Object[] args
            methodBody.append("    Object[] args = new Object[").append(paramTypes.length).append("];\n");
            for (int i = 0; i < paramTypes.length; i++) {
                // ($w) 是 Javassist 的特殊语法，用于包装基本类型为其对应的包装类
                methodBody.append("    args[").append(i).append("] = ($w)$").append(i + 1).append(";\n");
            }
            
            // 3. 调用 handler.invoke 方法
            // 'this' 指的是代理对象本身
            // 'method' 是上面获取的被代理方法的 Method 对象
            // 'args' 是包含调用参数的数组
            methodBody.append("    Object result = handler.invoke(this, method, args);\n");
            
            // 4. 处理返回值
            if (!method.getReturnType().equals(void.class)) { // 如果方法有返回值
                if (method.getReturnType().isPrimitive()) { // 如果是基本数据类型
                    // 需要将 Object result 拆箱为对应的基本类型
                    if (method.getReturnType().equals(boolean.class)) {
                        methodBody.append("    return ((Boolean)result).booleanValue();\n");
                    } // ... 其他基本类型的处理 ...
                    else if (method.getReturnType().equals(double.class)) {
                        methodBody.append("    return ((Double)result).doubleValue();\n");
                    }
                } else { // 如果是引用类型
                    // 直接强制类型转换
                    methodBody.append("    return (").append(method.getReturnType().getName()).append(")result;\n");
                }
            }
            methodBody.append("}");
            ```
            生成的代理方法体大致如下（以一个 `String sayHello(String name)` 方法为例）：
            ```java
            {
                // 获取 Method 对象: MyInterface.class.getMethod("sayHello", new Class[] {String.class});
                java.lang.reflect.Method method = com.example.MyInterface.class.getMethod("sayHello", new Class[] {java.lang.String.class});
            
                // 准备参数: args[0] = (String)$1; ($1 是方法的第一个参数 name)
                Object[] args = new Object[1];
                args[0] = ($w)$1;
            
                // 调用 handler: result = handler.invoke(this, method, args);
                Object result = handler.invoke(this, method, args);
            
                // 返回结果: return (String)result;
                return (java.lang.String)result;
            }
            ```
            *   `($w)$1`: Javassist 的特殊语法，`$1` 代表方法的第一个参数，`$2` 代表第二个，以此类推。`($w)` 用于自动装箱/拆箱，确保基本类型参数能正确传递给 `Object[] args`。
    
        *   **设置方法体并添加到代理类**:
            ```java
            System.out.println("method Object:\n"+methodBody); // 打印生成的方法体，用于调试
            ctMethod.setBody(methodBody.toString());
            proxyClass.addMethod(ctMethod);
            ```
            将构建好的方法体字符串设置给 `CtMethod`，然后将该方法添加到代理类 `proxyClass` 中。
    
    *   **写入 `.class` 文件 (可选，用于调试)**:
        ```java
        proxyClass.writeFile("./target/classes");
        System.out.println("\ngenerate proxyClass at ./target/classes/generated/"+proxyClassName+".class\n");
        ```
        这行代码会将动态生成的代理类的 `.class` 文件输出到指定的目录（这里是 `./target/classes`）。这对于调试和理解生成的字节码非常有帮助。生成的类文件会位于 `com/myproxy/generated/` 目录下。
    
    *   **加载代理类**:
        ```java
        Class<?> generatedClass = proxyClass.toClass(loader, null);
        ```
        使用指定的类加载器 `loader` 将 `CtClass` 对象（包含字节码信息）转换为实际的 `java.lang.Class` 对象。此时，这个动态生成的类就被加载到 JVM 中了。
    
    *   **创建代理对象实例**:
        ```java
        Constructor<?> cons = generatedClass.getConstructor(MyInvocationHandler.class);
        return cons.newInstance(handler);
        ```
        通过反射获取代理类的构造函数（该构造函数接收一个 `MyInvocationHandler` 参数），然后使用传入的 `handler` 实例创建代理对象，并返回。

**如何与 `MyInvocationHandler` 配合工作**：

你需要创建一个实现了 `MyInvocationHandler` 接口的类。这个接口通常会有一个 `invoke` 方法，例如：
```java
// MyInvocationHandler.java (假设的接口定义)
package com.myproxy;

import java.lang.reflect.Method;

public interface MyInvocationHandler {
    Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

当你调用代理对象上的任何方法时，`MyProxy` 生成的代理方法内部逻辑会执行，最终会调用到你提供的 `MyInvocationHandler` 实例的 `invoke` 方法。
*   `proxy`: 指的是代理对象本身。
*   `method`: 指的是被调用的实际方法（`java.lang.reflect.Method` 对象）。
*   `args`: 包含传递给方法的参数的数组。

在 `invoke` 方法中，你可以决定如何处理这个方法调用，例如：
*   直接调用原始对象的方法（如果代理的是一个具体实现）。
*   执行一些前置/后置操作。
*   修改参数或返回值。
*   甚至完全不调用原始方法，而是返回一个自定义的结果。

**一个简单的使用示例 (概念性)**:

假设你有如下接口和调用处理器：

```java
// 1. 定义一个接口
package com.example;

public interface GreetingService {
    String greet(String name);
    void sayGoodbye();
}
```

```java
// 2. 实现 MyInvocationHandler
package com.myproxy.handlers;

import com.myproxy.MyInvocationHandler;
import java.lang.reflect.Method;
import java.util.Arrays;

public class LoggingInvocationHandler implements MyInvocationHandler {
    private Object target; // 可选，如果要代理具体对象

    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }

    public LoggingInvocationHandler() {}


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("[MyProxy Log] Calling method: " + method.getName() + " with args: " + Arrays.toString(args));

        Object result = null;
        // 如果有实际的目标对象，可以在这里调用目标对象的方法
        // if (target != null) {
        //     result = method.invoke(target, args);
        // } else {
        //     // 如果只是代理接口，没有具体实现，可以在这里提供默认行为或抛出异常
        //     System.out.println("[MyProxy Log] No target object, method " + method.getName() + " not invoked on a real object.");
        //     if (method.getReturnType() == String.class) {
        //         result = "Default String Response from Proxy";
        //     }
        // }

        // 示例：简单地根据方法名返回不同结果，不调用实际对象
        if ("greet".equals(method.getName())) {
            result = "Hello, " + args[0] + " from MyProxy!";
        } else if ("sayGoodbye".equals(method.getName())) {
            System.out.println("[MyProxy Log] Saying goodbye!");
            // void 方法，result 可以是 null
        }


        System.out.println("[MyProxy Log] Method " + method.getName() + " finished.");
        return result;
    }
}
```

```java
// 3. 使用 MyProxy 创建代理
package com.example.main;

import com.example.GreetingService;
import com.myproxy.MyProxy;
import com.myproxy.handlers.LoggingInvocationHandler;

public class Main {
    public static void main(String[] args) {
        try {
            // 创建一个 InvocationHandler 实例
            LoggingInvocationHandler handler = new LoggingInvocationHandler();

            // 使用 MyProxy 创建代理对象
            GreetingService proxyInstance = (GreetingService) MyProxy.newProxyInstance(
                    GreetingService.class.getClassLoader(),
                    new Class<?>[]{GreetingService.class},
                    handler
            );

            // 调用代理对象的方法
            String greetingMessage = proxyInstance.greet("World");
            System.out.println("Received from proxy: " + greetingMessage);

            proxyInstance.sayGoodbye();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

当运行 `Main` 类时，输出可能如下：
```
method Object: // 这里会打印 MyProxy 中生成的 greet 方法体
{
    java.lang.reflect.Method method = com.example.GreetingService.class.getMethod("greet", new Class[] {java.lang.String.class});
    Object[] args = new Object[1];
    args[0] = ($w)$1;
    Object result = handler.invoke(this, method, args);
    return (java.lang.String)result;
}
method Object: // 这里会打印 MyProxy 中生成的 sayGoodbye 方法体
{
    java.lang.reflect.Method method = com.example.GreetingService.class.getMethod("sayGoodbye", new Class[] {});
    Object[] args = new Object[0];
    Object result = handler.invoke(this, method, args);
}

generate proxyClass at ./target/classes/generated/com.myproxy.generated.$MyProxy0.class

[MyProxy Log] Calling method: greet with args: [World]
[MyProxy Log] Method greet finished.
Received from proxy: Hello, World from MyProxy!
[MyProxy Log] Calling method: sayGoodbye with args: []
[MyProxy Log] Saying goodbye!
[MyProxy Log] Method sayGoodbye finished.
```

**总结**：

`MyProxy` 类通过 Javassist 动态生成实现了指定接口的代理类。这个代理类的每个方法都会将调用委托给用户提供的 `MyInvocationHandler` 的 `invoke` 方法。这提供了一种强大而灵活的方式来实现横切关注点，而无需修改原始业务代码，也不依赖 Java 内置的 `Proxy` 类（后者要求被代理的类必须实现接口，或者使用 CGLIB 等库来代理类）。`MyProxy` 的实现方式更接近于手动编写代理类的字节码，Javassist 使得这个过程相对简单一些。

这种自定义代理的优点在于：
*   **更深层次的控制**：可以直接操作字节码（通过 Javassist 的 API），实现更复杂的代理逻辑。
*   **不依赖 Java `Proxy`**: 在某些特定场景下可能有用，或者用于学习目的。

缺点：
*   **复杂性**：相比 Java 内置 `Proxy`，实现更复杂，需要处理字节码生成、类加载等细节。
*   **Javassist 依赖**：需要引入 Javassist 库。
*   **性能**：动态生成类的过程可能会有一些性能开销，但一旦类生成并加载后，调用性能通常是可以接受的。

​        