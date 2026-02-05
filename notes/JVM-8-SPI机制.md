[[JVM]]

SPI（Service Provider Interface）是一种服务发现机制，它允许应用程序在运行时动态地发现和加载实现某个接口的服务提供者。

### SPI原理

#### 双亲委派模型失效

据双亲委派模型的规定，当一个接口的实现类位于父类加载器加载的类路径下时，子类记载器无法加载到该实现类。

SPI 接口通常由 Java 核心库提供，而 SPI 的实现则由应用依赖的第三方 jar 引入。所以SPI 的接口是由 Bootstrap ClassLoader 加载的，而 SPI 的实现类一般是通过 Application ClassLoader 加载的。根据双亲委派模型，我们发现Bootstrap ClassLoader是无法反向找到 SPI 的实现类的，因为它只能加载 Java 核心库。

#### 线程上下文加载器

类 java.lang.Thread 中的 getContextClassLoader()和 setContextClassLoader(ClassLoader cl) 用来获取和设置线程的上下文类加载器。

Java 应用运行的初始线程的上下文类加载器是 Application ClassLoader。在线程中运行的代码可以通过这个类加载器来加载类和资源。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。通过线程上下文加载器，实现了可以通过父类加载器请求子类加载器去完成类加载器的动作，也就是**打破了双亲委派模型的限制**。

### SPI的约定

1. SPI的实现类实现一个不带参数的构造函数。
2. 在jar包的META-INF/services目录里创建一个文件，文件以接口的全限定名进行命名，文件内容是实现类的全限定名，方便SPI自动查找和加载实现类。
3. 把包含这个接口实现类的jar包放到主程序的classpath里面。

![图片](JVM-8-SPI%E6%9C%BA%E5%88%B6.assets/937b7a9c0ba1dc6bc83b3baa01584fda.png)

### SPI 背后设计思想

#### 开放封闭原则（Open-Closed Principle）

通过添加新的代码来扩展功能，而不是修改已有的代码。

#### 控制反转（Inversion of Control，简称IoC）

原本由开发人员控制的对象的创建和依赖关系的管理交给容器来控制。

### SPI实践

```java
public class ExtensionLoader<T> {
    private final Class<T> type;
    private final Map<String, Class<? extends T>> extensions = new ConcurrentHashMap<>();
    private final Map<String, T> instances = new ConcurrentHashMap<>();
    private String defaultName;

    private ExtensionLoader(Class<T> type) {
        this.type = type;
        loadExtensions();
    }

    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        return new ExtensionLoader<>(type);
    }

    // 加载扩展实现
    private void loadExtensions() {
        String fileName = "META-INF/services/" + type.getName();
        try (InputStream in = type.getClassLoader().getResourceAsStream(fileName)) {
            if (in == null) {
                return;
            }
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(in))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    line = line.trim();
                    if (line.isEmpty() || line.startsWith("#")) {
                        continue;
                    }
                    Class<?> clazz = Class.forName(line);//反射获取类
                    if (!type.isAssignableFrom(clazz)) {
                        throw new IllegalArgumentException(clazz.getName() + " is not a subtype of " + type.getName());
                    }
                    String name = getName(clazz);
                    extensions.put(name, (Class<? extends T>) clazz);//缓存扩展类
                }
            }
        } catch (Exception e) {
            throw new RuntimeException("Failed to load extensions for " + type.getName(), e);
        }

        // 设置默认实现
        SPI spi = type.getAnnotation(SPI.class);
        if (spi != null && !spi.value().isEmpty()) {
            defaultName = spi.value();
        }
    }

    // 获取实现类的名称（使用@Extension或默认规则）
    private String getName(Class<?> clazz) {
        Extension extension = clazz.getAnnotation(Extension.class);
        if (extension != null) {
            return extension.value();
        }
        // 默认名称：类名去掉接口名后小写
        String className = clazz.getSimpleName();
        String interfaceName = type.getSimpleName();
        if (className.endsWith(interfaceName)) {
            className = className.substring(0, className.length() - interfaceName.length());
        }
        return className.toLowerCase();
    }

    // 获取指定名称的扩展实例
    public T getExtension(String name) {
        T instance = instances.get(name);
        if (instance == null) {
            synchronized (this) {
                instance = instances.get(name);
                if (instance == null) {
                    Class<? extends T> clazz = extensions.get(name);
                    if (clazz == null) {
                        throw new RuntimeException("No extension found for name: " + name + " in " + type.getName());
                    }
                    try {
                        instance = clazz.getDeclaredConstructor().newInstance();
                        instances.put(name, instance);
                    } catch (Exception e) {
                        throw new RuntimeException("Failed to instantiate extension: " + name, e);
                    }
                }
            }
        }
        return instance;
    }

    // 获取默认扩展实例
    public T getDefaultExtension() {
        if (defaultName == null) {
            throw new RuntimeException("No default extension defined for " + type.getName());
        }
        return getExtension(defaultName);
    }
}

```

扫描资源文件
权限定类名
