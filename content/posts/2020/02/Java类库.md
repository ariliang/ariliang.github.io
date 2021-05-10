---
title: "Java类库"
date: 2020-02-20T23:47:27+08:00
summary: "核心类与扩展类"
draft: false
tags: [java]
---

# Java类库

## JAVA核心类

### 字符串和编码

1. 实际上字符串内部是用`char[]`来表示的

   > 比如`String s = new String(new char[] {'H', 'e', 'l', 'l', 'o', '!'});`
   >
   > 它的内部的字段定义是`private final char[]`，所以字符串对象是恒定不变的

1. 字符串比较应该利用`equals()`而不是`==`，因为字符串是引用类型，`==`比较的是引用值

```
// 比较内容，利用==比较也为true，是因为编译器会自动把相同的字符串当作一个对象放入常量池，自然引用"hello"和引用s的引用值是相等的
String s = "hello";
boolean isEqual1 = "hello".equals(s) ? true : false;	// true
boolean isEqual2 = "hello" == s ? true : false;			// true

// 比较内容，s与"hello".toUpperCase()的引用不相同
String s = "HELLO";
boolean isEqual = "hello".toUpperCase() == "s" ? true : false;	// false
```

1. 常用方法

```
"Hello".equalsIgnoreCase("Hello");	// true，忽略大小写比较
"Hello".contains("llo");			// true，是否包含某个子串，方法参数是CharSequence

"Hello".indexOf("e");				// 1，子串的索引，由0开始
"Hello".lastIndexOf("l");			// 3，最后一个字串的索引，0开始

"Hello".startsWith("He");			// true，由某个字串开始
"Hello".endsWith("lo");				// true，由某个字串结束

" \nHello\r\t".trim();				// "Hello"，去除首尾空白字符
"\u3000Hello\u3000".strip();		// "Hello"，去除首尾空白字符，包括其他语言的，'\u3000'是中文的空格
" Hello ".stripLeading();			// "Hello "，去除头部空白字符
" Hello ".stripEnding();			// " Hello"，去除尾部空白字符

"".isEmpty();		// true
"  \n".isBlank();	// true

"Hello".subString(2);				// "llo"，提取2到末尾的字串，索引从0开始
"Hello".subString(2, 4);			// "ll"，2到3字串，4之前

"Hello".replace('l', 'w');			// "hewwo"，替换字符
"Hello".replace("ll", "~~");		// "he~~o"，所有"ll"替换为"~~"
// 还有正则表达式的替换，待添加

String[] arr = "A,B,C,D".split("\\,");	// ["A", "B", "C", "D"]，正则表达式来分割
String s = String.join("***", arr);		// "A***B***C***D"，拼接字符串

String.valueOf(123);	// 将其他类型转换为字符串
String.valueOf(35.43);
String.valueOf(true);	// "true"
String.valueOf(new Object());	//	类似java.lang.Object@636be97c

// String可以和char[]相互转换
char[] cs = "Hello".toCharArray();	// String --> char[]
String s = new String(cs);			// char[] --> String
char[0] = 'X';
System.out.println(s);		// "Hello"，修改了char[]后s并不会改变，因为两者是不同的对象。new String(char[]);是复制，若传入的对象有可能改变，那么是复制内容
```

1. 字符编码

   > 1. ASCII只有1个字节编码的范围是0~~127(0000 0000~~0111 1111)，高位始终为0
   > 2. GB2312采用2字节，第一个高位始终为1以便和ASCII码区分(1000 0000 0000 0000 ~ 1111 1111 1111)
   > 3. Unicode编码使用两个或更多字节，英文字符就是简单的在前面加00，因为这样会浪费空间，就出现了UTF-8，把固定长度的Unicode编码编程1~4字节的变长编码
   > 4. UTF-8依靠高字节来确定一个字符究竟是几个字节，传输中容错能强，常用作传输编码
   > 5. Java的String和char在内存中总是以Unicode表示

   ```
      // char类型实际上是两个字节的Unicode编码，可以手动把字符串转换为其他编码，转换后不再是char类型，而是byte型数组
      // 转换为byte[]类型的时候优先考虑UTF-8
      byte[] b1 = "Hello".getBytes();		// 按ISO8859-1编码转换，不推荐
      byte[] b2 = "Hello".getBytes("UTF-8");		// 按UTF-8转换
      byte[] b3 = "Hello".getBytes("GBK");		// 按GBK转换
      byte[] b4 = "Hello".getBytes(StandardCharsets.UTF_8);	// 按UTF-8转换
      
      // 将已知的byte型数组转化为String
      String s1 = new String(b3, "GBK");
   String s2 = new String(b4, StandardCharsets.UTF_8);
      
      // 早期的String内部由char[]实现，而现在内部是byte[]，因为许多情况下只有ASCII内容，而使用char[]会浪费内存，利用byte[]能够存储一个字节，更节约
      private final char[] value;  -->  private final byte[] value;
   ```

### StringBuilder

1. java对String做了特殊处理，可以直接使用`+`拼接字符串
2. 构造复杂的字符串利用`StringBuilder`

## Object

方法列表

```
// toString();	将内容输出为字符串
// equals();	判断是否逻辑相等
// hashCode();	计算instance的hash值
```

## 容器类

### Arrays

```
// 将数值类型转化为字符串
Arrays.toString();
Arrays.deepToString();	// 输出多维数组为字符串

// 排序，若是对基本类型数组排序，则改变了数组元素的的位置。
// 若是对引用类型排序，如字符串，则改变了索引的引用值
Arrays.sort();

```

## IO

1. IO是指内存和外部硬盘，网络等的数据交换，以内存为中心；实际上在内存中以某种数据结构表示，如`byte[]`，`String`

2. IO分为同步和异步，分别在包`java.io`和`java.nio`中；同步指必须在IO结束后才能继续后续步骤，异步指只是发出IO请求，立刻执行后续步骤

   ```
   // 同步IO
   import java.io;
   
   /* 
    * 抽象类
    * 	InputStream/OutputStream, Reader/Writer
    * 实现类
    * 	FileInputStream/FileOutputStream, FileReader/FileWriter
    */
   
   // 异步IO
   import java.nio
   
   /*
    *	Path/Paths
    *
    */
   ```

### File/Path对象

利用`File`来操作文件和目录，`File`既可以表示文件，也可以表示目录；构造`File`时只是构造对象，不会进行任何IO操作，所以不会报错。而在调用的时候，才会有磁盘操作

```
// 要构造一个File对象需要传入文件/目录路径，可以是绝对路径，也可以是相对路径
File f1 = new File("D:\\Downloads");	// Win下，绝对路径
File f2 = new File("/home/user");		// unix下
System.out.println(File.separator)		// 返回当前OS分隔符
File f3 = new File("./dir");			// 相对路径
File f4 = new Fifle("../test");

// 利用File对象获取三种路径
f.getPath();			// (String)返回构造方法传入的路径：./Downloads
f.getAbsoluttePath/File();	// (String/File)返回绝对路径：类似 D:\Downloads\..\Downloa
ds
f.getCanonicalPath/File();	// (String/File)返回绝对规范路径：类似 D:\Downloads
f.getName();				// String，返回文件或目录的名字

// 文件或目录操作，在调用这些方法的时候才会进行实际的磁盘访问
f.exists();
f.isFile();			// 是否是文件
f.diDirectory();	// 是否是目录

f.canRead();		// boolean
f.canWrite();		// boolean
f.canExecute();		// boolean，目录的话代表是否可以列出其子目录和文件
f.lastModified();	// long，返回上次修改时间
f.setReadable(true);	// 设置可读，同样还有可写，可执行，修改上次修改时间
f.length();			// long，文件字节大小
f.getFreeSpace/TotalSpace/UsableSpace();	// long, 返回空间大小

f.createNewFile();	// 创建新文件
File f = File.createTempFile("tmp-", ".txt");	// 创建临时文件，指定前缀和后缀，类似 temp-342556345.txt
f.delete();			// 删除文件或目录，删除目录时必须为空
f.deleteOnExit();	// JVM退出时自动删除
f.mkdir();
f.mkdirs();			// 级联创建目录

// 遍历文件和目录
String[] fl = f.list();		// String[]，同上，以String数组返回
File[] fl = f.listFiles();	// File[]，列出所有文件和目录，以File数组形式返回
File[] fl = f.listFiles(
    new FilenameFilter() {
        public boolean accept(File dir, String name) {
            return name.endWith(".exe");  // 仅列出该类型文件
        }
    }
);		// 传入一个过滤对象，过滤不必要的文件或目录
```

可以利用Path类来更轻松地创建路径`java.nio.file.Path/Paths`

```
// 构造一个路径
Path p = Paths.get(".", "Downloads", "temp");	// Path，返回一个Path对象，其路径为 ./Downloads/temp
p = p.toAbsolutePath();		// Path，返回绝对路径
p = p.normalize();			// Path，返回标准路径（不是绝对的）；要先 toAbsolutePath()，再 normalize()，才能得到标准的绝对路径

// 与File对象相互转换
File f = p.toFile();		// File，返回File对象
p = f.toPath();				// Path，返回Path对象

// 遍历
for (Path p : Paths.get("..").toAbsolutePath()) {
    System.out.print(p + ", ");
}	// 构造成 */*/.. 路径，再把其变为绝对路径拆开输出：*, *, ..
```

### InputStream/OutputStream

`InputSteam/OutputStream`：IO流以*字节\*为单位(byte)，因此也称为字节流；`InputStream`表示输入流，`OutputStream`表示输出流，是两种基本的流；\*它们的特点是像水流一样单向流动*
它们并不是接口，而是抽象类，是一切输入输出流的超类

**InputStream**

定义了一个重要的抽象方法 `pubilc abstract int read() throws IOException;`这个抽象方法会读出下一个字节，以int返回(0~255)，读到末尾-1表示

在调用read()时是阻塞(Blocking)的，即IO未完成之前不会执行后续步骤

> 实现类：
>
> `FileInputStream`文件输入流
>
> `ByteArrayInputStream`在内存中模拟一个输入流，实际上是把`byte[]`当作`InputStream`，可以用于测试

```
// 实现类 FileInputStream 文件输入流
// 打开资源并使用后一定要记得关闭，下面展示了一个最简单的方法，但不是最常用的
public void readFile() throws IOException {
   InputStream input = new FileInputStream("./text.txt");   // 传入文件路径
   for (;;) {
      int n = input.read();   // 读出下一个byte值
      if (n != -1) {
         break;
      }
      System.out.println(n);  // 打印byte的值
   }
   input.close();    // 关闭流
}
// 文件输入输出流不管是否成功或失败，都需要及时关闭资源
public void readFile() throws IOException {
   InputStream input = null;
   try {
      input = new FileInputStream("./text.txt");
      for(int n; (n = input.read()) != -1;) {
         System.out.println(n);
      }
   } finally {
      if (input != null) { input.close(); }
   }
}
// 利用try(resources)来自动关闭资源，常用(JAVA 7引入)
public void readFile() IOException {
   try (InputStream input = new FileInputStream("./text.txt")) {
      for (int n;(n = input.read()) != -1) {
         System.out.println(n);
      }
   }  // 自动关闭，决定于try()里边的对象是否实现了java.lang.AutoClosable接口，里边有finally，并调用close()
}


// 一次读取一个字节不是高效的办法，利用缓冲可以一次读取多个字节；每次会读满buffer.length个字节，然后清空之前读的字节，记录读到的位置，直到整个文件读完
byte[] buffer = new byte[1000];        // 定义一个buffer，每次读取字节数不会超过这个数组的大小
int read(byte[] b);                    // 读取若干字节填充到byte[]，返回读取的字节数，返回-1表示没有更多的字节了
int read(byte[] b, int off, int len);  // 同时指定偏移量和最大填充数
public void readFile(){		// 缓冲读取完整步骤
    // 利用缓冲读取字节流
    byte[] btBuffer = new byte[1000];
    try (InputStream fs = new FileInputStream("./text.txt")) {
        // read()返回的是读取的长度
        for (int n; (n = fs.read(btBuffer)) != -1;){
            // byte数组输出为字符串的时候必须指定(offset, len, charset)
            System.out.println(new String(btBuffer, 0, n, StandardCharsets.UTF_8));
        }
    } catch (Exception e) {}
    System.out.println();
}

// ByteArrayInputStream，模拟输入流
byte[] data = { 72, 101, 108, 108, 111, 33 };		// 构造byte[]
// 从byte[]中输入
try (InputStream input = new ByteArrayInputStream(data)) {
    int n;
    while ((n = input.read()) != -1) {
        System.out.println((char)n);
    }
}
```

**OutputStream**

这个抽象类定义了一个重要的方法`public abstract void write(int b) throws IOException`每次写入一个字节byte

因为它先把写入的字节放到缓冲区而不是真正的写入，当缓冲区写满时才会把缓冲区写入，它还提供了另外一个方法`flush()`，利用它可以立即把缓冲区写入到文件，网络等

在调用`close()`的时候会自动调用`flush()`

OutputStream也是阻塞的

> 实现类：
>
> `FileOutputStream`文件输出流
>
> `ByteArrayOutputStream`内存模拟输出流

```
// write()每次写入一个字节，或一个byte[],写入后会覆盖原文件内容
byte[] b = new byte[] {'e', 'l', 'l', 'o'};
String s = "world!";
OutputStream output = new FileOutputStream("./text.txt");
output.write('h');				// char/int
output.write(43);				// 0~255
output.write(b, 0, b.length);	// (byte[], offset, len)
output.write(s.getBytes(StandardCharsets.UTF_8));
output.close();					// 关闭输出流

// 同样的，应该把文件的输出流放在try(resources)中，会自动关闭文件
try (OutputStream output = new FileOutputStream("./text.txt")) {
    output.write('h');
} catch (Exception e) {}
```

### Filter模式

通过基础组件再叠加各种附加功能组件的模式，叫做Filter模式，或者装饰器(Decorator)模式；如图，各种`*InputStream`都可以用`InputStream`来使用，而且仅仅是在基础的数据录入源的基础上增添功能，若每个附加的功能都写一个类，那样会使类的数量大大增

![Filer-mod](images/Java类库/Filter-mod.png)

```
// 编写 FilterInputStream 扩展类
public class Main {
    public static void main(String[] args) throws IOException {
        byte[] data = "hello, world!".getBytes("UTF-8");
        try (CountInputStream input = new CountInputStream(new ByteArrayInputStream(data))) {
            int n;
            while ((n = input.read()) != -1) {
                System.out.println((char)n);
            }
            System.out.println("Total read " + input.getBytesRead() + " bytes");
        }
    }
}

class CountInputStream extends FilterInputStream {
    private int count = 0;

    CountInputStream(InputStream in) {
        super(in);
    }

    public int getBytesRead() {
        return this.count;
    }

    public int read() throws IOException {
        int n = in.read();
        if (n != -1) {
            this.count ++;
        }
        return n;
    }

    public int read(byte[] b, int off, int len) throws IOException {
        int n = in.read(b, off, len);
        this.count += n;
        return n;
    }
}
```

操作Zip，它是一种 `FilterInputStream`；另外`JarInputStream`派生自 `ZipInputStream`，是在zip的基础上增加对`MANIFEST.MF`文件的读写

![filter-zip](images/Java类库/filter-zip.png)



```
// 读取压缩包内容
try (ZipInputStream zip = new ZipInputStream(new FileInputStream("./zip.zip"))) {
    for (ZipEntry entry; (entry = zip.getNextEntry()) != null; ) {
        System.out.print(entry.getName());		// 获取压缩包内部文件或目录的名字
        System.out.print(", ");
        System.out.print(entry.getTime());		// 获取内容的时间
        System.out.print(", ");
        System.out.println(entry.getSize());	// 获取压缩前内容的大小
    }
} catch (Exception e) {
    //TODO: handle exception
}

// 写入压缩包
try (ZipOutputStream zip = new ZipOutputStream(new FileOutputStream(...))) {
    File[] files = ...
    for (File file : files) {
        zip.putNextEntry(new ZipEntry(file.getName()));
        zip.write(getFileDataAsBytes(file));
        zip.closeEntry();
    }
}
```

### 读取`classpath`

经常读取配置文件，且每次写磁盘路径很麻烦，可以利用`classpath`而不用关心具体的路径，这样就解决了文件路径依赖。它的路径从`/`开始，可以包含任意其他类型格式

```
// 先获取到当前Class对象，getResourceAsStream()当对象不存在时返回null
try (InputStream input = getClass().getResourceAsStream("/default.properties")) {
    if (input != null) {
        // TODO
    }
}

// 默认的配置在jar包中，若有用户配置，则读取用户配置
Properties props = new Properties();
props.load(inputStreamFromClassPath("/default.properties"));
props.load(inputStreamFromFile("./conf.properties"));
```

### 序列化

序列化就是把java对象编程二进制内容，本质就是一个`byte[]`，因为序列化后可以把对象保存到文件中，便于传输

反序列化时由`byte[]`直接生成对象，不会调用构造方法，但是这个过程中会产生安全问题；而且Java的序列化机制仅适用于Java，要和其他语言交换数据，要使用通用的序列化方法：如 Json，只输出基本类型和String

```
// 要实现对象的序列化，必须实现一个特殊接口 java.io.Serializable；它没有定义任何方法，是一个空接口，称这种接口叫做标记接口
public interface Serializable {}

// 序列化需要使用 ObjectOutputStream，它可以写入基本类型如int/boolean，也可以写入String等；还可以写入实现了 Sreierializeable 接口的 Object，写入 Object 需要大量信息，所以内容很大
ByteArrayOutputStream buffer = new ByteArrayOutputStream();
try (ObjectOutputStream output = new ObjectOutputStream(buffer)) {
    output.writeInt(12345);			// 写入int:
    output.writeUTF("Hello");		// 写入String:
    output.writeObject(Double.valueOf(123.456));	// 写入Object:
}
System.out.println(Arrays.toString(buffer.toByteArray()));

// 反序列化，调用 readObject() 可以直接返回一个对象，必须强制转型；反序列化时是直接构造出 Object，不会调用其构造方法，导致了安全性问题
try (ObjectInputStream input = new ObjectInputStream(...)) {
    int n = input.readInt();
    String s = input.readUTF();
    Double d = (Double) input.readObject();
}

// 序列化和反序列化过程中很可能发生一些错误
ClassNotFoundException;		// 反序列化时没有对应的类
InvalidClassException;		// Class不匹配，比如 int 和 long 不匹配
// 所以序列化时允许 Class 定义一个特殊的 ID；静态变量 serialVersionUID(非必须)，用于标识序列化的版本，通常由IDE生成；如果增加或修改了字段，就可以改变其值
public class Person implements Serializable {
    private static final long serialVersionUID = 2709425275741743919L;
}
```

### Reader/Writer

若要读写的是字符(char)，且字符不全是单字节表示的ASCII，那么按照`char`来读写更方便，这种流称为字符流；其本质是一个能自动编解码的`InputStream/OutputStream`，会自动地在byte和char之间转换，*需指定编码方式*

**Reader**

操作大概与`InputStream`类似

![comparasion-between-inputstream-and-reader](images/Java类库/comparasion-between-inputstream-and-reader.png)

```
// 同样的，可以利用缓冲，一次读取多个字符
public void readFile(){
    // 利用缓冲读取字符流
    char[] chBuffer = new char[1000];
    // 构造时传入文件路径和编码方式
    try (Reader rd = new FileReader("./text.txt", StandardCharsets.UTF_8)) {
        // read()返回的是读取字符个数
        for (int n;(n = rd.read(chBuffer)) != -1;) {
            System.out.println(n);
            // 字符数组offset, len)
            System.out.println(new String(chBuffer, 0, n));
        }
    } catch (Exception e) {}
    System.out.println();
}

// 其他实现类
// CharArrayReader，内存中模拟Reader流，实际上是一个char[]
try (Reader reader = new CharArrayReader("Hello".toCharArray())) {
}
// StringReader， 可以直接把String当作输入流，和CharArrayReader几乎一样
try (Reader reader = new StringReader("Hello")) {
}

// InputStream和Reader：Reader内部有一个InputStream，它先把数据以byte读入InputStream，再以特定的编码转换为char；可以手动转换，利用InputStreamReader
InputStream input = new FileInputStream("src/readme.txt");
Reader reader = new InputStreamReader(input, "UTF-8");
// 简化代码，构造Reader时传入InputStream，这实际上时FileReader的一种实现方式
try (Reader reader = new InputStreamReader(new FileInputStream("src/readme.txt"), "UTF-8")) {
    // TODO:
}
```

### PrintStream/PrintWriter

**PrintStream**

```
PrintStream`是一种`FilterOutputStream`，总是以byte数据输出；额外提供了一些写入各种数据(int, boolean, String, Object…)的方法`print()/println()`，除了提供了额外的方法外还有个特点就是不会抛出`IOException
```

> `System.out.print`是系统默认提供的`PrintStream`， 表示标准输出
>
> `System.err`是系统默认提供的标准错误输出

**PrintWriter**

`PrintWriter`扩展了`Writer`接口，输出的是char数据，使用方法与`PrintStream`几乎一样

```
StringWriter buffer = new StringWriter();
try (PrintWriter pw = new PrintWriter(buffer)) {
    pw.println("Hello");
    pw.println(12345);
    pw.println(true);
}
System.out.println(buffer.toString());
```

[参考自廖雪峰的教程](https://www.liaoxuefeng.com/)