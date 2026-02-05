[[Java]]

# SE

## I/O

### Stream

#### 概述

Stream 流其实就是一根传送带，元素在上面可以被 Stream 流操作

* 可以解决已有集合类库或者数组 API 的弊端
* Stream 流简化集合和数组的操作
* 链式编程

```java
list.stream().filter(new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.startsWith("张");
            }
        });

list.stream().filter(s -> s.startsWith("张"));
```



***



#### 获取流

集合获取 Stream 流用：`default Stream<E> stream()`

数组：Arrays.stream(数组)   /  Stream.of(数组);

```java
// Collection集合获取Stream流。
Collection<String> c = new ArrayList<>();
Stream<String> listStream = c.stream();

// Map集合获取流
// 先获取键的Stream流。
Stream<String> keysStream = map.keySet().stream();
// 在获取值的Stream流
Stream<Integer> valuesStream = map.values().stream();
// 获取键值对的Stream流（key=value： Map.Entry<String,Integer>）
Stream<Map.Entry<String,Integer>> keyAndValues = map.entrySet().stream();

//数组获取流
String[] arr = new String[]{"Java", "JavaEE" ,"Spring Boot"};
Stream<String> arrStream1 = Arrays.stream(arr);
Stream<String> arrStream2 = Stream.of(arr);
```



****



#### 常用API

| 方法名                                                    | 说明                                                     |
| --------------------------------------------------------- | -------------------------------------------------------- |
| void forEach(Consumer<? super T> action)                  | 逐一处理（遍历）                                         |
| long count                                                | 返回流中的元素数                                         |
| Stream<T> filter(Predicate<? super T> predicate)          | 用于对流中的数据进行过滤                                 |
| Stream<T> limit(long maxSize)                             | 返回此流中的元素组成的流，截取前指定参数个数的数据       |
| Stream<T> skip(long n)                                    | 跳过指定参数个数的数据，返回由该流的剩余元素组成的流     |
| <R> Stream<R> map(Function<? super T,? extends R> mapper) | 加工方法，将当前流中的 T 类型数据转换为另一种 R 类型的流 |
| static <T> Stream<T> concat(Stream a, Stream b)           | 合并 a 和 b 两个流为一个，调用 `Stream.concat(s1,s2)`    |
| Stream<T> distinct()                                      | 返回由该流的不同元素组成的流                             |

```java
public class StreamDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌"); list.add("周芷若"); list.add("赵敏");
        list.add("张三"); list.add("张三丰"); list.add("张");
        //取以张开头并且名字是三位数的
        list.stream().filter(s -> s.startsWith("张")
                .filter(s -> s.length == 3).forEach(System.out::println);
        //统计数量
		long count = list.stream().filter(s -> s.startsWith("张")
                .filter(s -> s.length == 3).count();
		//取前两个
		list.stream().filter(s -> s.length == 3).limit(2).forEach(...);
		//跳过前两个
		list.stream().filter(s -> s.length == 3).skip(2).forEach(...);

		// 需求：把名称都加上“张三的:+xxx”
		list.stream().map(s -> "张三的" + s).forEach(System.out::println);
		// 需求：把名称都加工厂学生对象放上去!!
		// list.stream().map(name -> new Student(name));
		list.stream.map(Student::new).forEach(System.out::println);
                                          	
		//数组流
		Stream<Integer> s1 = Stream.of(10,20,30,40,50);
		//集合流
		Stream<String> s2 = list.stream();
		//合并流
		Stream<Object> s3 = Stream.concat(s1,s2);
		s3.forEach(System.out::println);
    }
}
class Student{
    private String name;
    //......
}                                          
```



***



#### 终结方法

终结方法：Stream 调用了终结方法，流的操作就全部终结，不能继续使用，如 foreach，count 方法等

非终结方法：每次调用完成以后返回一个新的流对象，可以继续使用，支持**链式编程**

```java
// foreach终结方法
list.stream().filter(s -> s.startsWith("张"))
    .filter(s -> s.length() == 3).forEach(System.out::println);
```



***



#### 收集流

收集 Stream：把 Stream 流的数据转回到集合中去

* Stream 流：工具
* 集合：目的

Stream 收集方法：`R collect(Collector collector)` 把结果收集到集合中

Collectors 方法：

* `public static <T> Collector toList()`：把元素收集到 List 集合中
* `public static <T> Collector toSet()`：把元素收集到 Set 集合中
* `public static  Collector toMap(Function keyMapper,Function valueMapper)`：把元素收集到 Map 集合中
* `Object[] toArray()`：把元素收集数组中
* `public static Collector groupingBy(Function<? super T, ? extends K> classifier)`：分组

```java
public static void main(String[] args) {
	List<String> list = new ArrayList<>();
	Stream<String> stream = list.stream().filter(s -> s.startsWith("张"));    
    //把stream流转换成Set集合。
    Set<String> set = stream.collect(Collectors.toSet());
    
    //把stream流转换成List集合。
    //重新定义，因为资源已经被关闭了
    Stream<String> stream1 = list.stream().filter(s -> s.startsWith("张"));
    List<String> list = stream.collect(Collectors.toList());
    
    //把stream流转换成数组。
    Stream<String> stream2 = list.stream().filter(s -> s.startsWith("张"));
    Object[] arr = stream2.toArray();
    // 可以借用构造器引用申明转换成的数组类型！！！
    String[] arr1 = stream2.toArray(String[]::new);
}
```



***



### File

#### 文件类

File 类：代表操作系统的文件对象，是用来操作操作系统的文件对象的，删除文件，获取文件信息，创建文件（文件夹），广义来说操作系统认为文件包含（文件和文件夹）

File 类构造器：

* `public File(String pathname)`：根据路径获取文件对象
* `public File(String parent , String child)`：根据父路径和文件名称获取文件对象

File 类创建文件对象的格式：

* `File f = new File("绝对路径/相对路径");`
  * 绝对路径：从磁盘的的盘符一路走到目的位置的路径
    * 绝对路径依赖具体的环境，一旦脱离环境，代码可能出错
    * 一般是定位某个操作系统中的某个文件对象
  * **相对路径**：不带盘符的（重点）
    * 默认是直接相对到工程目录下寻找文件的。
    * 相对路径只能用于寻找工程下的文件，可以跨平台

* `File f = new File("文件对象/文件夹对象")` 广义来说：文件是包含文件和文件夹的

```java
public class FileDemo{
    public static void main(String[] args) {
        // 1.创建文件对象：使用绝对路径
        // 文件路径分隔符：
        //      -- a.使用正斜杠： /
        //      -- b.使用反斜杠： \\
        //      -- c.使用分隔符API:File.separator
        //File f1 = new File("D:"+File.separator+"it"+File.separator
		//+"图片资源"+File.separator+"beautiful.jpg");
        File f1 = new File("D:\\seazean\\图片资源\\beautiful.jpg");
        System.out.println(f1.length()); // 获取文件的大小，字节大小

        // 2.创建文件对象：使用相对路径
        File f2 = new File("Day09Demo/src/dlei.txt");
        System.out.println(f2.length());

        // 3.创建文件对象：代表文件夹。
        File f3 = new File("D:\\it\\图片资源");
        System.out.println(f3.exists());// 判断路径是否存在！！
    }
}
```



***



#### 常用API

##### 常用方法

| 方法                           | 说明                                   |
| ------------------------------ | -------------------------------------- |
| String getAbsolutePath()       | 返回此 File 的绝对路径名字符串         |
| String getPath()               | 获取创建文件对象的时候用的路径         |
| String getName()               | 返回由此 File 表示的文件或目录的名称   |
| long length()                  | 返回由此 File 表示的文件的长度（大小） |
| long length(FileFilter filter) | 文件过滤器                             |

```java
public class FileDemo {
    public static void main(String[] args) {
        // 1.绝对路径创建一个文件对象
        File f1 = new File("E:/图片/test.jpg");
        // a.获取它的绝对路径。
        System.out.println(f1.getAbsolutePath());
        // b.获取文件定义的时候使用的路径。
        System.out.println(f1.getPath());
        // c.获取文件的名称：带后缀。
        System.out.println(f1.getName());
        // d.获取文件的大小：字节个数。
        System.out.println(f1.length());
        System.out.println("------------------------");

        // 2.相对路径
        File f2 = new File("Demo/src/test.txt");
        // a.获取它的绝对路径。
        System.out.println(f2.getAbsolutePath());
        // b.获取文件定义的时候使用的路径。
        System.out.println(f2.getPath());
        // c.获取文件的名称：带后缀。
        System.out.println(f2.getName());
        // d.获取文件的大小：字节个数。
        System.out.println(f2.length());
    }
}

```



***



##### 判断方法

方法列表：

* `boolean exists()`：此 File 表示的文件或目录是否实际存在
* `boolean isDirectory()`：此 File 表示的是否为目录
* `boolean isFile()`：此 File 表示的是否为文件

```java
File f = new File("Demo/src/test.txt");
// a.判断文件路径是否存在
System.out.println(f.exists()); // true
// b.判断文件对象是否是文件,是文件返回true ,反之
System.out.println(f.isFile()); // true
// c.判断文件对象是否是文件夹,是文件夹返回true ,反之
System.out.println(f.isDirectory()); // false
```



****



##### 创建删除

方法列表：

* `boolean createNewFile()`：当且仅当具有该名称的文件尚不存在时， 创建一个新的空文件
* `boolean delete()`：删除由此 File 表示的文件或目录（只能删除空目录）
* `boolean mkdir()`：创建由此 File 表示的目录（只能创建一级目录）
* `boolean mkdirs()`：可以创建多级目录（建议使用）

```java
public class FileDemo {
    public static void main(String[] args) throws IOException {
        File f = new File("Demo/src/test.txt");
        // a.创建新文件，创建成功返回true ,反之
        System.out.println(f.createNewFile());

        // b.删除文件或者空文件夹
        System.out.println(f.delete());
        // 不能删除非空文件夹，只能删除空文件夹
        File f1 = new File("E:/it/aaaaa");
        System.out.println(f1.delete());

        // c.创建一级目录
        File f2 = new File("E:/bbbb");
        System.out.println(f2.mkdir());

        // d.创建多级目录
        File f3 = new File("D:/it/e/a/d/ds/fas/fas/fas/fas/fas/fas");
        System.out.println(f3.mkdirs());
    }
}
```



***



#### 遍历目录

- `public String[] list()`：获取当前目录下所有的一级文件名称到一个字符串数组中去返回
- `public File[] listFiles()`：获取当前目录下所有的一级文件对象到一个**文件对象数组**中去返回（**重点**）
- `public long lastModified`：返回此抽象路径名表示的文件上次修改的时间

```java
public class FileDemo {
    public static void main(String[] args) {
        File dir = new File("D:\\seazean");
        // a.获取当前目录对象下的全部一级文件名称到一个字符串数组返回。
        String[] names = dir.list();
        for (String name : names) {
            System.out.println(name);
        }
        // b.获取当前目录对象下的全部一级文件对象到一个File类型的数组返回。
        File[] files = dir.listFiles();
        for (File file : files) {
            System.out.println(file.getAbsolutePath());
        }

        // c
        File f1 = new File("D:\\图片资源\\beautiful.jpg");
        long time = f1.lastModified(); // 最后修改时间！
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(time));
    }
}
```



***



#### 文件搜索

递归实现文件搜索（非规律递归）

* 定义一个方法用于做搜索
* 进入方法中进行业务搜索分析

```java
/**
 * 去某个目录下搜索某个文件
 * @param dir 搜索文件的目录。
 * @param fileName 搜索文件的名称。
 */
public static void searchFiles(File dir , String fileName){
    // 1.判断是否存在该路径，是否是文件夹
    if(dir.exists() && dir.isDirectory()){
        // 2.提取当前目录下的全部一级文件对象
        File files = dir.listFiles();// 可能是null/也可能是空集合[]
        // 3.判断是否存在一级文件对象,判断是否不为空目录
        if(files != null && files.length > 0){
            // 4.判断一级文件对象
            for(File file : files){
                // 5.判断file是文件还是文件夹
                if(file.isFile()){
                    // 6.判断该文件是否为我要找的文件对象
                    if(f.getName().contains(fileName)){//模糊查找
                        sout(f.getAbsolutePath());
                        try {
                            // 启动它（拓展）
                            Runtime r = Runtime.getRuntime();
                            r.exec(f.getAbsolutePath());
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                } else {
                    // 7.该文件是文件夹，文件夹要递归进入继续寻找
                    searchFiles(file,fileName)
                }
            }
        }
    }
}
```



***



### Character

字符集：为字符编制的一套编号规则

计算机的底层是不能直接存储字符的，只能存储二进制 010101

ASCII 编码：8 个开关一组就可以编码字符，1 个字节 2^8 = 256， 一个字节存储一个字符完全够用，英文和数字在底层存储都是采用 1 个字节存储的

```
a  97
b  98

A  65
B  66

0  48
1  49
```

中国人：中国人有 9 万左右字符，2 个字节编码一个中文字符，1 个字节编码一个英文字符，这套编码叫：GBK 编码，兼容 ASCII 编码表

美国人：收集全球所有的字符，统一编号，这套编码叫 Unicode 编码（万国码），一个英文等于两个字节，一个中文（含繁体）等于两个字节，中文标点占两个字节，英文标点占两个字节

* UTF-8 是变种形式，也必须兼容 ASCII 编码表
* UTF-8 一个中文一般占 3 个字节，中文标点占 3 个，英文字母和数字 1 个字节

编码前与编码后的编码集必须一致才不会乱码



***



### IOStream

#### 概述

IO 输入输出流：输入/输出流

* Input：输入
* Output：输出

引入：File 类只能操作文件对象本身，不能读写文件对象的内容，读写数据内容，应该使用 IO 流

IO 流是一个水流模型：IO 理解成水管，把数据理解成水流

IO 流的分类：

* 按照流的方向分为：输入流，输出流。
  * 输出流：以内存为基准，把内存中的数据**写出到磁盘文件**或者网络介质中去的流称为输出流
  * 输入流：以内存为基准，把磁盘文件中的数据或者网络中的数据**读入到内存**中的流称为输入流
* 按照流的内容分为：字节流，字符流
  * 字节流：流中的数据的最小单位是一个一个的字节，这个流就是字节流
  * 字符流：流中的数据的最小单位是一个一个的字符，这个流就是字符流（**针对于文本内容**）

流大体分为四大类：字节输入流、字节输出流、字符输入流、字符输出流

```java
IO 流的体系：
        字节流                                   字符流
  字节输入流              字节输出流            字符输入流         字符输出流
InputStream           OutputStream          Reader            Writer   (抽象类)
FileInputStream       FileOutputStream      FileReader        FileWriter(实现类)
BufferedInputStream  BufferedOutputStream  BufferedReader   BufferedWriter(实现类缓冲流)
                                           InputStreamReader OutputStreamWriter
ObjectInputStream     ObjectOutputStream
```



****



#### 字节流

##### 字节输入

FileInputStream 文件字节输入流：以内存为基准，把磁盘文件中的数据按照字节的形式读入到内存中的流

构造方法：

* `public FileInputStream(File path)`：创建一个字节输入流管道与源文件对象接通
* `public FileInputStream(String pathName)`：创建一个字节输入流管道与文件路径对接，底层实质上创建 File 对象 

方法：

* `public int read()`：每次读取一个字节返回，读取完毕会返回 -1
* `public int read(byte[] buffer)`：从字节输入流中读取字节到字节数组中去，返回读取的字节数量，没有字节可读返回 -1，**byte 中新读取的数据默认是覆盖原数据**，构造 String 需要设定长度
* `public String(byte[] bytes,int offset,int length)`：构造新的 String
* `public long transferTo(OutputStream out) `：从输入流中读取所有字节，并按读取的顺序，将字节写入给定的输出流

```java
public class FileInputStreamDemo01 {
    public static void main(String[] args) throws Exception {
        // 1.创建文件对象定位dlei01.txt
        File file = new File("Demo/src/dlei01.txt");
        // 2.创建一个字节输入流管道与源文件接通
        InputStream is = new FileInputStream(file);
        // 3.读取一个字节的编号返回，读取完毕返回-1
		//int code1 = is.read(); // 读取一滴水，一个字节
		//System.out.println((char)code1);

        // 4.使用while读取字节数
        // 定义一个整数变量存储字节
        int ch = 0 ;
        while((ch = is.read())!= -1){
            System.out.print((char) ch);
        }
    }
}
```

一个一个字节读取英文和数字没有问题，但是读取中文输出无法避免乱码，因为会截断中文的字节。一个一个字节的读取数据，性能也较差，所以**禁止使用上面的方案**

采取下面的方案：

```java
public static void main(String[] args) throws Exception {
    //简化写法，底层实质上创建了File对象
    InputStream is = new FileInputStream("Demo/src/test.txt");
    byte[] buffer = new byte[3];//开发中使用byte[1024]
    int len;
    while((len = is.read(buffer)) !=-1){
        // 读取了多少就倒出多少！
        String rs = new String(buffer, 0, len);
        System.out.print(rs);
    }
}
```

```java
File f = new File("Demo/src/test.txt");
InputStream is = new FileInputStream(f);
// 读取全部的
byte[] buffer = is.readAllBytes();
String rs = new String(buffer);
System.out.println(rs);
```



****



##### 字节输出

FileOutputStream 文件字节输出流：以内存为基准，把内存中的数据，按照字节的形式写出到磁盘文件中去

构造方法：

* `public FileOutputStream(File file)`：创建一个字节输出流管道通向目标文件对象
* `public FileOutputStream(String file) `：创建一个字节输出流管道通向目标文件路径
* `public FileOutputStream(File file, boolean append)` : 创建一个追加数据的字节输出流管道到目标文件对象
* `public FileOutputStream(String file, boolean append)` : 创建一个追加数据的字节输出流管道通向目标文件路径

API：

* `public void write(int a)`：写一个字节出去 
* `public void write(byte[] buffer)`：写一个字节数组出去
* `public void write(byte[] buffer , int pos , int len)`：写一个字节数组的一部分出去，从 pos 位置，写出 len 长度

* FileOutputStream 字节输出流每次启动写数据的时候都会先清空之前的全部数据，重新写入：
  * `OutputStream os = new FileOutputStream("Demo/out05")`：覆盖数据管道
  * `OutputStream os = new FileOutputStream("Demo/out05" , true)`：追加数据的管道 

说明：

* 字节输出流只能写字节出去，字节输出流默认是**覆盖**数据管道
* 换行用：**os.write("\r\n".getBytes())**
* 关闭和刷新：刷新流可以继续使用，关闭包含刷新数据但是流就不能使用了

```java
OutputStream os = new FileOutputStream("Demo/out05");
os.write(97);//a
os.write('b');
os.write("\r\n".getBytes());
os.write("我爱Java".getBytes());
os.close();
```



##### 文件复制

字节是计算机中一切文件的组成，所以字节流适合做一切文件的复制

```java
public class CopyDemo01 {
    public static void main(String[] args) {
        InputStream is = null ;
        OutputStream os = null ;
        try{
            //（1）创建一个字节输入流管道与源文件接通。
            is = new FileInputStream("D:\\seazean\\图片资源\\test.jpg");
            //（2）创建一个字节输出流与目标文件接通。
            os = new FileOutputStream("D:\\seazean\\test.jpg");
            //（3）创建一个字节数组作为桶
            byte buffer = new byte[1024];
            //（4）从字节输入流管道中读取数据，写出到字节输出流管道即可
            int len = 0;
            while((len = is.read(buffer)) != -1){
                os.write(buffer,0,len);
            }
            System.out.println("复制完成！");
        }catch (Exception e){
            e.printStackTrace();
        } finally {
            /**（5）关闭资源！ */
            try{
                if(os!=null)os.close();
                if(is!=null)is.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```



***



#### 字符流

##### 字符输入

FileReader：文件字符输入流，以内存为基准，把磁盘文件的数据以字符的形式读入到内存，读取文本文件内容到内存中去

构造器：

* `public FileReader(File file)`：创建一个字符输入流与源文件对象接通。
*  `public FileReader(String filePath)`：创建一个字符输入流与源文件路径接通。

方法：

* `public int read()`：读取一个字符的编号返回，读取完毕返回 -1
* `public int read(char[] buffer)`：读取一个字符数组，读取多少个就返回多少个，读取完毕返回 -1

结论：

* 字符流一个一个字符的读取文本内容输出，可以解决中文读取输出乱码的问题，适合操作文本文件，但是一个一个字符的读取文本内容性能较差
* 字符流按照**字符数组循环读取数据**，可以解决中文读取输出乱码的问题，而且性能也较好

**字符流不能复制图片，视频等类型的文件**。字符流在读取完了字节数据后并没有直接往目的地写，而是先查编码表，查到对应的数据就将该数据写入目的地。如果查不到，则码表会将一些未知区域中的数据去 map 这些字节数据，然后写到目的地，这样的话就造成了源数据和目的数据的不一致。

```java
public class FileReaderDemo01{//字符
    public static void main(String[] args) throws Exception {
        // 创建一个字符输入流管道与源文件路径接通
        Reader fr = new FileReader("Demo/src/test.txt");
        int ch;
        while((ch = fr.read()) != -1){
            System.out.print((char)ch);
        }
    }
}
public class FileReaderDemo02 {//字符数组
    public static void main(String[] args) throws Exception {
        Reader fr = new FileReader("Demo/src/test.txt");
        
        char[] buffer = new char[1024];
        int len;
        while((len = fr.read(buffer)) != -1) {
            System.out.print(new String(buffer, 0 , len));
        }
    }
}
```



***



##### 字符输出

FileWriter：文件字符输出流，以内存为基准，把内存中的数据按照字符的形式写出到磁盘文件中去

构造器：

* `public FileWriter(File file)`：创建一个字符输出流管道通向目标文件对象（覆盖数据管道）
* `public FileWriter(String filePath)`：创建一个字符输出流管道通向目标文件路径
* `public FileWriter(File file, boolean append)`：创建一个追加数据的字符输出流管道通向文件对象（追加数据管道）
* `public FileWriter(String filePath, boolean append)`：创建一个追加数据的字符输出流管道通向目标文件路径

方法：

* `public void write(int c)`：写一个字符出去
* `public void write(char[] buffer)`：写一个字符数组出去
* `public void write(String c, int pos, int len)`：写字符串的一部分出去
* `public void write(char[] buffer, int pos, int len)`：写字符数组的一部分出去
* `fw.write("\r\n")`：换行

读写字符文件数据建议使用字符流

```java
Writer fw = new FileWriter("Demo/src/test.txt");
fw.write(97);   // 字符a
fw.write('b');  // 字符b
fw.write("Java是最优美的语言！");
fw.write("\r\n");
fw.close;
```



****



#### 缓冲流

##### 基本介绍

缓冲流可以提高字节流和字符流的读写数据的性能

缓冲流分为四类：

* BufferedInputStream：字节缓冲输入流，可以提高字节输入流读数据的性能
* BufferedOutStream：字节缓冲输出流，可以提高字节输出流写数据的性能
* BufferedReader：字符缓冲输入流，可以提高字符输入流读数据的性能
* BufferedWriter：字符缓冲输出流，可以提高字符输出流写数据的性能



***



##### 字节缓冲输入

字节缓冲输入流：BufferedInputStream

作用：可以把低级的字节输入流包装成一个高级的缓冲字节输入流管道，提高字节输入流读数据的性能

构造器：`public BufferedInputStream(InputStream in)`

原理：缓冲字节输入流管道自带了一个 8KB 的缓冲池，每次可以直接借用操作系统的功能最多提取 8KB 的数据到缓冲池中去，以后我们直接从缓冲池读取数据，所以性能较好

```java
public class BufferedInputStreamDemo01 {
    public static void main(String[] args) throws Exception {
        // 1.定义一个低级的字节输入流与源文件接通
        InputStream is = new FileInputStream("Demo/src/test.txt");
        // 2.把低级的字节输入流包装成一个高级的缓冲字节输入流。
        BufferInputStream bis = new BufferInputStream(is);
        // 3.定义一个字节数组按照循环读取。
        byte[] buffer = new byte[1024];
        int len;
        while((len = bis.read(buffer)) != -1){
            String rs = new String(buffer, 0 , len);
            System.out.print(rs);
        }
    }
}
```



***



##### 字节缓冲输出

字节缓冲输出流：BufferedOutputStream

作用：可以把低级的字节输出流包装成一个高级的缓冲字节输出流，从而提高写数据的性能

构造器：`public BufferedOutputStream(OutputStream os)`

原理：缓冲字节输出流自带了 8KB 缓冲池,数据就直接写入到缓冲池中去，性能提高了

```java
public class BufferedOutputStreamDemo02 {
    public static void main(String[] args) throws Exception {
        // 1.写一个原始的字节输出流
        OutputStream os = new FileOutputStream("Demo/src/test.txt");
        // 2.把低级的字节输出流包装成一个高级的缓冲字节输出流
        BufferedOutputStream bos =  new BufferedOutputStream(os);
        // 3.写数据出去
        bos.write('a');
        bos.write(100);
        bos.write("我爱中国".getBytes());
        bos.close();
    }
}

```



##### 字节流性能

利用字节流的复制统计各种写法形式下缓冲流的性能执行情况

复制流：

* 使用低级的字节流按照一个一个字节的形式复制文件
* 使用低级的字节流按照一个一个字节数组的形式复制文件
* 使用高级的缓冲字节流按照一个一个字节的形式复制文件
* 使用高级的缓冲字节流按照一个一个字节数组的形式复制文件

高级的缓冲字节流按照一个一个字节数组的形式复制文件，性能最高，建议使用



****



##### 字符缓冲输入

字符缓冲输入流：BufferedReader

作用：字符缓冲输入流把字符输入流包装成高级的缓冲字符输入流，可以提高字符输入流读数据的性能。

构造器：`public BufferedReader(Reader reader)`

原理：缓冲字符输入流默认会有一个 8K 的字符缓冲池,可以提高读字符的性能

按照行读取数据的功能：`public String readLine()`  读取一行数据返回，读取完毕返回 null

```java
public static void main(String[] args) throws Exception {
    // 1.定义一个原始的字符输入流读取源文件
    Reader fr = new FileReader("Demo/src/test.txt");
    // 2.把低级的字符输入流管道包装成一个高级的缓冲字符输入流管道
    BufferedReader br = new BufferedReader(fr);
    // 定义一个字符串变量存储每行数据
    String line;
    while((line = br.readLine()) != null){
        System.out.println(line);
    }
    br.close();
    //淘汰数组循环读取
    //char[] buffer = new char[1024];
    //int len;
    //while((len = br.read(buffer)) != -1){
    //System.out.println(new String(buffer , 0 , len));
}
```



***



##### 字符缓冲输出

符缓冲输出流：BufferedWriter

作用：把低级的字符输出流包装成一个高级的缓冲字符输出流，提高写字符数据的性能。

构造器：`public BufferedWriter(Writer writer)`

 原理：高级的字符缓冲输出流多了一个 8K 的字符缓冲池，写数据性能极大提高了

字符缓冲输出流多了一个换行的特有功能：`public void newLine()`  **新建一行**

```java
public static void main(String[] args) throws Exception {
    Writer fw = new FileWriter("Demo/src/test.txt",true);//追加
    BufferedWriter bw = new BufferedWriter(fw);
    
    bw.write("我爱学习Java");
    bw.newLine();//换行
    bw.close();
}
```



***



##### 高效原因

字符型缓冲流高效的原因：（空间换时间）

* BufferedReader：每次调用 read 方法，只有第一次从磁盘中读取了 8192（**8k**）个字符，存储到该类型对象的缓冲区数组中，将其中一个返回给调用者，再次调用 read 方法时，就不需要访问磁盘，直接从缓冲区中拿出一个数据即可，提升了效率
* BufferedWriter：每次调用 write 方法，不会直接将字符刷新到文件中，而是存储到字符数组中，等字符数组写满了，才一次性刷新到文件中，减少了和磁盘交互的次数，提升了效率

字节型缓冲流高效的原因：

* BufferedInputStream：在该类型中准备了一个数组，存储字节信息，当外界调用 read() 方法想获取一个字节的时候，该对象从文件中一次性读取了 8192 个字节到数组中，只返回了第一个字节给调用者。将来调用者再次调用 read 方法时，当前对象就不需要再次访问磁盘，只需要从数组中取出一个字节返回给调用者即可，由于读取的是数组，所以速度非常快。当 8192 个字节全都读取完成之后，再需要读取一个字节，就得让该对象到文件中读取下一个 8192 个字节
* BufferedOutputStream：在该类型中准备了一个数组，存储字节信息，当外界调用 write 方法想写出一个字节的时候，该对象直接将这个字节存储到了自己的数组中，而不刷新到文件中。一直到该数组所有 8192 个位置全都占满，该对象才把这个数组中的所有数据一次性写出到目标文件中。如果最后一次循环没有将数组写满，最终在关闭流对象的时候，也会将该数组中的数据刷新到文件中。 



注意：**字节流和字符流，都是装满时自动写出，或者没满时手动 flush 写出，或 close 时刷新写出**



***



#### 转换流

##### 乱码问题

字符流读取：

```
代码编码            文件编码         中文情况。
UTF-8              UTF-8           不乱码!
GBK                GBK             不乱码!
UTF-8              GBK             乱码!
```

* 如果代码编码和读取的文件编码一致，字符流读取的时候不会乱码
* 如果代码编码和读取的文件编码不一致，字符流读取的时候会乱码



***



##### 字符输入

字符输入转换流：InputStreamReader

作用：解决字符流读取不同编码的乱码问题，把原始的**字节流**按照默认的编码或指定的编码**转换成字符输入流**

构造器：

* `public InputStreamReader(InputStream is)`：使用当前代码默认编码 UTF-8 转换成字符流
* `public InputStreamReader(InputStream is, String charset)`：指定编码把字节流转换成字符流

```java
public class InputStreamReaderDemo{
    public static void main(String[] args) throws Exception {
        // 1.提取GBK文件的原始字节流
        InputStream is = new FileInputStream("D:\\seazean\\Netty.txt");
        // 2.把原始字节输入流通过转换流，转换成 字符输入转换流InputStreamReader
        InputStreamReader isr = new InputStreamReader(is, "GBK"); 
        // 3.包装成缓冲流
        BufferedReader br = new BufferedReader(isr);
        //循环读取
        String line;
        while((line = br.readLine()) != null){
            System.out.println(line);
        }
    }
}
```



***



##### 字符输出

字符输出转换流：OutputStreamWriter

作用：可以指定编码**把字节输出流转换成字符输出流**，可以指定写出去的字符的编码

构造器：

* `public OutputStreamWriter(OutputStream os)`：用默认编码 UTF-8 把字节输出流转换成字符输出流
* `public OutputStreamWriter(OutputStream os, String charset)`：指定编码把字节输出流转换成

```Java
OutputStream os = new FileOutputStream("Demo/src/test.txt");
OutputStreamWriter osw = new OutputStreamWriter(os,"GBK");
osw.write("我在学习Java");   
osw.close();
```



****



#### 序列化

##### 基本介绍

对象序列化：把 Java 对象转换成字节序列的过程，将对象写入到 IO 流中，对象 => 文件中

对象反序列化：把字节序列恢复为 Java 对象的过程，从 IO 流中恢复对象，文件中 => 对象

transient 关键字修饰的成员变量，将不参与序列化



***



##### 序列化

对象序列化流（对象字节输出流）：ObjectOutputStream

作用：把内存中的 Java 对象数据保存到文件中去

构造器：`public ObjectOutputStream(OutputStream out)`

序列化方法：`public final void writeObject(Object obj)`

注意：对象如果想参与序列化，对象必须实现序列化接口 **implements Serializable** ，否则序列化失败

```java
public class SerializeDemo01 {
    public static void main(String[] args) throws Exception {
        // 1.创建User用户对象
        User user = new User("seazean","980823","七十一");
        // 2.创建低级的字节输出流通向目标文件
        OutputStream os = new FileOutputStream("Demo/src/obj.dat");
        // 3.把低级的字节输出流包装成高级的对象字节输出流 ObjectOutputStream
        ObjectOutputStream oos = new ObjectOutputStream(os);
        // 4.通过对象字节输出流序列化对象：
        oos.writeObject(user);
        // 5.释放资源
        oos.close();
        System.out.println("序列化对象成功~~~~");
    }
}

class User implements Serializable {
    // 加入序列版本号
    private static final long serialVersionUID = 1L;

    private String loginName;
    private transient String passWord;
    private String userName;
    // get+set
}
```

```java
// 序列化为二进制数据
ByteArrayOutputStream bos = new ByteArrayOutputStream();
ObjectOutputStream oos = new ObjectOutputStream(bos);
oos.writeObject(obj);	// 将该对象序列化为二进制数据
oos.flush();
byte[] bytes = bos.toByteArray();
```





****



##### 反序列

对象反序列化（对象字节输入流）：ObjectInputStream

作用：读取序列化的对象文件恢复到 Java 对象中

构造器：`public ObjectInputStream(InputStream is)`

方法：`public final Object readObject()`

序列化版本号：`private static final long serialVersionUID = 2L`

注意：序列化使用的版本号和反序列化使用的版本号一致才可以正常反序列化，否则报错

```java
public class SerializeDemo02 {
    public static void main(String[] args) throws Exception {
        InputStream is = new FileInputStream("Demo/src/obj.dat");
        ObjectInputStream ois = new ObjectInputStream(is);
        User user = (User)ois.readObject();//反序列化
        System.out.println(user);
        System.out.println("反序列化完成！");
    }
}
class User implements Serializable {
    // 加入序列版本号
    private static final long serialVersionUID = 1L;
    //........
}
```



****



#### 打印流

打印流 PrintStream / PrintWriter

打印流的作用：

* 可以方便，快速的写数据出去，可以实现打印什么类型，就是什么类型
* PrintStream/PrintWriter 不光可以打印数据，还可以写字节数据和字符数据出去
* **System.out.print() 底层基于打印流实现的** 

构造器：

* `public PrintStream(OutputStream os)`
* `public PrintStream(String filepath)`

System 类：

* `public static void setOut(PrintStream out)`：让系统的输出流向打印流

```java
public class PrintStreamDemo01 {
    public static void main(String[] args) throws Exception {
        PrintStream ps = new  PrintStream("Demo/src/test.txt");
        ps.println(任何类型的数据);
        ps.print(不换行);
        ps.write("我爱你".getBytes());
        ps.close();
    }
}
public class PrintStreamDemo02 {
    public static void main(String[] args) throws Exception {
        System.out.println("==seazean0==");
        PrintStream ps = new PrintStream("Demo/src/log.txt");
        System.setOut(ps); // 让系统的输出流向打印流
		//不输出在控制台，输出到文件里
        System.out.println("==seazean1==");
        System.out.println("==seazean2==");
    }
}
```



***



### Close

try-with-resources：

```java
try(
    // 这里只能放置资源对象，用完会自动调用close()关闭
){

}catch(Exception e){
 	e.printStackTrace();
}
```

资源类一定是实现了 Closeable 接口，实现这个接口的类就是资源

有 close() 方法，try-with-resources 会自动调用它的 close() 关闭资源

```java
try(
	/** （1）创建一个字节输入流管道与源文件接通。 */
	InputStream is  = new FileInputStream("D:\\seazean\\图片资源\\meinv.jpg");
	/** （2）创建一个字节输出流与目标文件接通。*/
	OutputStream os = new FileOutputStream("D:\\seazean\\meimei.jpg");
	/** （5）关闭资源！是自动进行的 */
){
	byte[] buffer = new byte[1024];
	int len = 0;
	while((len = is.read(buffer)) != -1){
		os.write(buffer, 0 , len);
	}
	System.out.println("复制完成！");
}catch (Exception e){
	e.printStackTrace();
}
```



***



### Properties

Properties：属性集对象。就是一个 Map 集合，一个键值对集合

核心作用：Properties 代表的是一个属性文件，可以把键值对数据存入到一个属性文件

属性文件：后缀是 `.properties` 结尾的文件，里面的内容都是 key=value

Properties 方法：

| 方法名                                       | 说明                                          |
| -------------------------------------------- | --------------------------------------------- |
| Object setProperty(String key, String value) | 设置集合的键和值，底层调用 Hashtable 方法 put |
| String getProperty(String key)               | 使用此属性列表中指定的键搜索属性              |
| Set<String>   stringPropertyNames()          | 所有键的名称的集合                            |
| synchronized void load(Reader r)             | 从输入字符流读取属性列表（键和元素对）        |
| synchronized void load(InputStream in)       | 加载属性文件的数据到属性集对象中去            |
| void store(Writer w, String comments)        | 将此属性列表(键和元素对)写入 Properties 表    |
| void store(OutputStream os, String comments) | 保存数据到属性文件中去                        |

````java
public class PropertiesDemo01 {
    public static void main(String[] args) throws Exception {
        // a.创建一个属性集对象：Properties的对象。
        Properties properties = new Properties();//{}
        properties.setProperty("admin" , "123456");
        // b.把属性集对象的数据存入到属性文件中去（重点）
        OutputStream os = new FileOutputStream("Demo/src/users.properties");
        properties.store(os,"i am very happy!!我保存了用户数据!");
        //参数一：被保存数据的输出管道
        //参数二：保存心得。就是对象保存的数据进行解释说明！
    }
}
````

````java
public class PropertiesDemo02 {
    public static void main(String[] args) throws Exception {
        Properties properties = new Properties();//底层基于map集合
        properties.load(new FileInputStream("Demo/src/users.properties"));
        System.out.println(properties);
        System.out.println(properties.getProperty("admin"));
        
		Set<String> set = properties.stringPropertyNames();
        for (String s : set) {
            String value = properties.getProperty(s);
            System.out.println(s + value);
        }
    }
}
````



***



### RandomIO

RandomAccessFile 类：该类的实例支持读取和写入随机访问文件

构造器：

* `RandomAccessFile(File file, String mode)`：创建随机访问文件流，从 File 参数指定的文件读取，可选择写入
* `RandomAccessFile(String name, String mode)`：创建随机访问文件流，从指定名称文件读取，可选择写入文件

常用方法：

* `public void seek(long pos)`：设置文件指针偏移，从该文件开头测量，发生下一次读取或写入(插入+覆盖)
* `public void write(byte[] b)`：从指定的字节数组写入 b.length 个字节到该文件
* `public int read(byte[] b)`：从该文件读取最多 b.length 个字节的数据到字节数组

```java
public static void main(String[] args) throws Exception {
    RandomAccessFile rf = new RandomAccessFile(new File(),"rw");
    rf.write("hello world".getBytes());
    rf.seek(5);//helloxxxxld
    rf.write("xxxx".getBytes());
    rf.close();
}
```



***



### Commons

commons-io 是 apache 提供的一组有关 IO 操作的类库，可以提高 IO 功能开发的效率

commons-io 工具包提供了很多有关 IO 操作的类：

| 包                                  | 功能描述                                        |
| ----------------------------------- | :---------------------------------------------- |
| org.apache.commons.io               | 有关 Streams、Readers、Writers、Files 的工具类  |
| org.apache.commons.io.input         | 输入流相关的实现类，包含 Reader 和 InputStream  |
| org.apache.commons.io.output        | 输出流相关的实现类，包含 Writer 和 OutputStream |
| org.apache.commons.io.serialization | 序列化相关的类                                  |

IOUtils 和 FileUtils 可以方便的复制文件和文件夹

```java
public class CommonsIODemo01 {
    public static void main(String[] args) throws Exception {
        // 1.完成文件复制！
        IOUtils.copy(new FileInputStream("Demo/src/books.xml"), 
                     new FileOutputStream("Demo/new.xml"));
        // 2.完成文件复制到某个文件夹下！
        FileUtils.copyFileToDirectory(new File("Demo/src/books.xml"),
                                      new File("D:/it"));
        // 3.完成文件夹复制到某个文件夹下！
        FileUtils.copyDirectoryToDirectory(new File("D:\\it\\图片服务器") ,
                                           new File("D:\\"));

        //  Java从1.7开始提供了一些nio, 自己也有一行代码完成复制的技术。
        Files.copy(Paths.get("Demo/src/books.xml")
                , new FileOutputStream("Demo/new11.txt"));
    }
}
```





***





