---
title: Java IO流
date: 2017-09-20 16:56:00
categories:
- java
tags:
- java
keywords: Java IO流 阻塞IO 面向流的IO
---

> 
Java IO流，主要有四大类：`File`文件、`InputStream/OutputStream`字节流、`Reader/Writer`字符流、`RandomAccessFile`随机存取文件类

<!-- more -->

## IO 流
IO 流类图结构：
![IO 流类图结构](/images/java-io.jpg)

**Stream 流的概念**
流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象；
即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作；

**IO 流的分类**
1) 根据处理`数据类型`的不同分为：`字符流`和`字节流`；
2) 根据数据`流向`不同分为：`输入流`和`输出流`。

**字符流和字节流**
字符流的由来：因为数据编码的不同，从而有了对字符进行高效操作的流对象；本质就是基于字节流读取时，去查了指定的码表。

字节流和字符流的区别：
1) 读写单位不同：`字节流`以`字节（8bit）`为单位，`字符流`以`字符`为单位，根据码表映射字符，一次可能读多个字节（`2字节或者4字节`）；
2) 处理对象不同：字节流能处理所有类型的数据（如图片、视频、文字等），而字符流只能处理字符类型的数据；

结论：只要是处理纯文本数据，就优先考虑使用字符流，除此之外都使用字节流；

**输入流和输出流**
对输入流只能进行读操作，对输出流只能进行写操作，程序中需要根据待传输数据的不同特性而使用不同的流。

**字节流类图**
![字节流类图](/images/java-io-byte-stream.png)
> 
图中蓝色的为主要的对应部分，红色的部分就是不对应部分；紫色的虚线部分代表这些流一般要搭配使用。

抽象类：
`InputStream`：所有输入字节流的父类，它是一个抽象类；
`OutputStream`：所有输出字节流的父类，它是一个抽象类。

基本的类：
1) `ByteArrayInputStream/ByteArrayOutputStream`：从字节数组中获取数据/往字节数组中写入数据（在内存中操作）
2) `FileInputStream/FileOutputStream`：从磁盘文件中获取数据/往磁盘文件中写入数据（在外存中操作）
3) `PipedInputStream/PipedOutputStream`：从管道中获取数据/往管道中写入数据（线程间通信）

缓冲 IO 流：
`BufferedInputStream/BufferedOutputStream`：自带缓冲区的包装流，默认建立一个 8192 字节的缓冲区，可以显著提高 IO 效率

序列化相关：
1) `DataInputStream/DataOutputStream`：用于基本类型的序列化、反序列化操作，包括 8 大基本类型、String字符串；
2) `ObjectInputStream/ObjectOutputStream`：用于引用类型的序列化、反序列化操作，需实现 java.io.Serializable 接口（标记接口）；

“存在即合理”，简单分析一下"不对称"的流类：
1) `LineNumberInputStream`：可以获取/设置数据的行号信息，已被标记为`Deprecated`，不建议使用；
2) `PushbackInputStream`：可以用来“窥探”输入流的内容而不对它们进行破坏；比较少用；
3) `StringBufferInputStream`：已被标记为`Deprecated`，本身就不应该出现在 java.io 包中；
4) `SequenceInputStream`：一个工具类，将两个或者多个输入流当成一个输入流依次读取；完全可以从 IO 包中去除；
5) `PrintStream`：用于格式化输出，提供 print、println、printf、format 系列方法；`System.out`、`System.err`就是`PrintStream`的实例；


**字符流类图**
![字符流类图](/images/java-io-char-stream.png)
> 
图中蓝色的为主要的对应部分，红色的部分就是不对应部分；紫色的虚线部分代表这些流一般要搭配使用。

抽象类：
`Reader`：所有输入字符流的父类，它是一个抽象类；
`Writer`：所有输出字符流的父类，它是一个抽象类。

基本的类：
1) `CharArrayReader/CharArrayWriter`：从字符数组中获取数据/往字符数组中写入数据（在内存中操作）
2) `StringReader/StringWriter`：从字符串中获取数据/往字符串中写入数据（在内存中操作）
3) `InputStreamReader/OutputStreamWriter`：从字节流中获取数据（解码`decoding`）/往字节流中写入数据（编码`encoding`）（字节流和字符流之间的桥梁）
4) `FileReader/FileWriter`：针对磁盘文件的`InputStreamReader/OutputStreamWriter`特化版本
5) `PipedReader/PipedWriter`：从管道中获取数据/往管道中写入数据（线程间通信）

缓冲 IO 流：
`BufferedReader/BufferedWriter`：自带缓冲区的包装流，默认建立一个 8192 字符（即 16 KB）的缓冲区，可以显著提高 IO 效率；
装饰 IO 流：
1) `LineNumberReader`：可以获取/设置数据的行号信息；
2) `PrintWriter`：支持格式化输出，提供 print、println、printf、format 方法，在 jdk1.4 之后，`PrintStream`、`PrintWriter`基本无区别，除了 autoFlush 的不同；


**File 文件类**
File 类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹；

File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法；

注意，实例化一个 File 对象并不会检查对应的文件（夹）的真实性，只有在进行真正的 IO 操作时才会检查（如创建文件、删除文件等）；


**RandomAccessFile**
RandomAccessFile 并不是上述流体系中的一员，其封装了字节流，同时还封装了一个缓冲区，通过内部的文件指针来操作文件；
RandomAccessFile 的特点：既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模式（`r`、`rw`、`rws`、`rwd`）；
RandomAccessFile 可以用于多线程下载或多个线程同时写数据到文件。


**STDIN、STDOUT、STDERR**
Java 和 C/C++ 一样，默认为每个进程打开了 3 个文件，即`标准输入文件`、`标准输出文件`、`标准错误文件`；

这 3 个标准文件定义在 java.lang.System 类中，均为静态成员：
1) `System.in`：`InputStream`类的实例，文件描述符 FileDescriptor 为 FileDescriptor.in，fd = 0；
2) `System.out`：`PrintStream`类的实例，文件描述符 FileDescriptor 为 FileDescriptor.out，fd = 1；
3) `System.err`：`PrintStream`类的实例，文件描述符 FileDescriptor 为 FileDescriptor.err，fd = 2；


**File 和 FileDescriptor 的区别**
一个 File 对象代表一个抽象的"磁盘文件"，仅仅是一个记录而已，和真正的文件之间没有关联；
一个 FileDescriptor 表示一个已打开的文件，fd 是程序在内核注册的一个文件"链接"，就像建立的网络连接一样；


## File 类
**构造函数**
`public File(String pathname);`：文件名（包含路径），可以为相对路径、绝对路径；
`public File(String parent, String child);`：父路径 + 子路径；
`public File(File parent, String child);`：父路径（File 对象） + 子路径；
`public File(URI uri);`：java.net.URI 形式，如`file:///etc/sysctl.d/default.conf`；

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public String getName(); // 获取文件名
public String getParent(); // 获取路径名
public File getParentFile(); // 获取 parent 的 File 对象
public String getPath(); // 获取完整路径名

public boolean isAbsolute(); // 是否为绝对路径
public String getAbsolutePath(); // 获取绝对路径名
public File getAbsoluteFile();  // 获取绝对路径的 File 对象

public URL toURL() throws MalformedURLException; // toURL
public URI toURI(); // toURI

public boolean canRead(); // 可读
public boolean canWrite(); // 可写
public boolean canExecute(); // 可执行

public boolean exists(); // 是否存在
public boolean isDirectory(); // 是否为目录
public boolean isFile(); // 是否为文件
public boolean isHidden(); // 是否为隐藏文件

public long lastModified(); // 获取 modify 时间
public long length(); // 获取文件长度

public boolean createNewFile() throws IOException; // 创建新文件
public boolean mkdir(); // 新建文件夹
public boolean mkdirs(); // 递归创建文件夹，如 mkdir -p /a/b/c

public boolean delete(); // 删除文件、空文件夹（立即删除）
public void deleteOnExit(); // 在程序结束时删除（一般用于临时文件的删除）

public String[] list(); // 返回当前路径下的文件列表
public File[] listFiles(); // 返回当前路径下的 File 对象列表

public boolean renameTo(File dest); // 重命名
public boolean setLastModified(long time); // 设置 modify 时间

public boolean setReadOnly(); // 设置只读权限

public boolean setReadable(boolean readable, boolean ownerOnly); // 设置读权限，ownerOnly 表示只作用在文件所有者上
public boolean setReadable(boolean readable); // ownerOnly = true
public boolean setWritable(boolean writable, boolean ownerOnly); // 设置写权限，ownerOnly 表示只作用在文件所有者上
public boolean setWritable(boolean writable); // ownerOnly = true
public boolean setExecutable(boolean executable, boolean ownerOnly); // 设置执行权限，ownerOnly 表示只作用在文件所有者上
public boolean setExecutable(boolean executable); // ownerOnly = true

public static File[] listRoots(); // 列出根文件系统 '/'

public long getTotalSpace(); // 获取分区总大小
public long getFreeSpace(); // 获取分区剩余空间大小
public long getUsableSpace(); // 获取分区可用空间大小

/**
 * 创建 temp 临时文件.
 * @param prefix 文件名前缀
 * @param suffix 文件名后缀
 * @param directory 在指定的文件夹下创建
 * @throws IOException 在操作失败时抛出 IOException 异常
 */
public static File createTempFile(String prefix, String suffix, File directory) throws IOException;
public static File createTempFile(String prefix, String suffix) throws IOException; // directory = null，默认在 /tmp/ 下创建

public int compareTo(File pathname); // 比较
public boolean equals(Object obj); // 判等
public int hashCode(); // HashCode值

public String toString(); // 等同于 getPath()
</script></code></pre>



**File.separator**
在 Windows 下的路径分隔符和 Linux 下的路径分隔符是不一样的，当直接使用绝对路径时，跨平台会暴出“No such file or diretory”错误；

比如要在 tmp 目录下建立一个临时文件 test.tmp：
Windows 下应写为：`File f = new File("C:\\tmp\\test.tmp");`，也可以为`File f = new File("C:/tmp/test.tmp");`
Linux 下应写为：`File f = new File("/tmp/test.tmp");`
如果考虑跨平台，最好这么写：`File f = new File("C:" + File.separator + "tmp" + File.separator + "test.tmp");`

File 类的几个 static 分隔符：
1) `public static final char separatorChar`：char 形式，文件路径分隔符，Windows下为`\`，Linux下为`/`；
2) `public static final String separator`：String 形式，同上；
3) `public static final char pathSeparatorChar`：char 形式，路径列表分隔符，Windows下为`;`，Linux下为`:`；
4) `public static final String pathSeparator`：String 形式，同上；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.io.*;
import java.net.URI;
import java.net.URISyntaxException;

public class Main {
    public static void main(String[] args) {
        // 创建 File 对象
        File f1 = new File("/etc/sysctl.d/default.conf");
        File f2 = new File("/etc/sysctl.d", "default.conf");
        File f3 = new File(new File("/etc/sysctl.d"), "default.conf");
        File f4 = null;
        try {
            f4 = new File(new URI("file:///etc/sysctl.d/default.conf"));
        } catch (URISyntaxException e) {
            System.err.println(e.getMessage());
        }

        System.out.printf("getName(): %s\n", f1.getName()); // default.conf
        System.out.printf("getParent(): %s\n", f2.getParent()); // /etc/sysctl.d
        System.out.printf("getParentFile().getPath(): %s\n", f3.getParentFile().getPath()); // /etc/sysctl.d
        System.out.printf("getPath(): %s\n", f4.getPath()); // /etc/sysctl.d/default.conf

        System.out.printf("isAbsolute(): %b\n", f1.isAbsolute()); // true

        System.out.printf("exists(): %b\n", f2.exists());
        System.out.printf("canRead(): %b\n", f2.canRead());
        System.out.printf("canWrite(): %b\n", f2.canWrite());
        System.out.printf("canExecute(): %b\n", f2.canExecute());
        System.out.printf("isDirectory(): %b\n", f2.isDirectory());
        System.out.printf("isFile(): %b\n", f2.isFile());
        System.out.printf("isHidden(): %b\n", f2.isHidden());

        System.out.printf("lastModified(): %1$tF %1$tT\n", f2.lastModified());
        System.out.printf("length(): %d\n", f2.length());

        System.out.printf("getTotalSpace(): %d GB\n", f3.getTotalSpace() / 1024 / 1024 / 1024);
        System.out.printf("getFreeSpace(): %d GB\n", f3.getFreeSpace() / 1024 / 1024 / 1024);
        System.out.printf("getUsableSpace(): %d GB\n", f3.getUsableSpace() / 1024 / 1024 / 1024);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:20:25]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:20:37]
$ java Main
getName(): default.conf
getParent(): /etc/sysctl.d
getParentFile().getPath(): /etc/sysctl.d
getPath(): /etc/sysctl.d/default.conf
isAbsolute(): true
exists(): true
canRead(): true
canWrite(): true
canExecute(): false
isDirectory(): false
isFile(): true
isHidden(): false
lastModified(): 2017-08-13 08:31:55
length(): 803
getTotalSpace(): 48 GB
getFreeSpace(): 46 GB
getUsableSpace(): 43 GB
</script></code></pre>


## InputStream、OutputStream
**InputStream 抽象类**
<pre><code class="language-java line-numbers"><script type="text/plain">// 读取 1 个字节 byte，EOF 返回 -1
public abstract int read() throws IOException;
// 调用 read(b, 0, b.length) 方法
public int read(byte b[]) throws IOException;
// 读取一个自 offset 起 length 长度的字节数组
public int read(byte b[], int off, int len) throws IOException;

// 跳过 n 个字节
public long skip(long n) throws IOException;
// 返回可读取的字节数
public int available() throws IOException;
// 关闭输入流
public void close() throws IOException;

/**
 * 标记当前位置 pos.
 * @param readlimit 在标记位置无效之前可以读取的最大字节数限制
 */
public synchronized void mark(int readlimit);
// 将此流重新定位到最近的调用 mark 方法时的位置
public synchronized void reset() throws IOException;
public boolean markSupported(); // 是否支持 mark 标记
</script></code></pre>



**OutputStream 抽象类**
<pre><code class="language-java line-numbers"><script type="text/plain">// 写入 1 个字节
public abstract void write(int b) throws IOException;
// 写入整个字节数组
public void write(byte b[]) throws IOException;
// 写入指定长度的字节
public void write(byte b[], int off, int len) throws IOException;

// flush 缓冲区(对 Buffered 流有效)
public void flush() throws IOException;
public void close() throws IOException; // 关闭输出流
</script></code></pre>



### ByteArray 字节数组
`ByteArrayInputStream`
**构造函数**
`public ByteArrayInputStream(byte buf[]);`
`public ByteArrayInputStream(byte buf[], int offset, int length);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public synchronized int read();
public synchronized int read(byte b[], int off, int len);

public synchronized long skip(long n);
public synchronized int available();

public boolean markSupported(); // true
public void mark(int readAheadLimit); // 在标记位置无效之前可以读取的最大字节数限制
public synchronized void reset();

public void close() throws IOException;
</script></code></pre>



`ByteArrayOutputStream`
**构造函数**
`public ByteArrayOutputStream();`：default 大小为 32（数组长度）
`public ByteArrayOutputStream(int size);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public synchronized void write(int b);
public synchronized void write(byte b[], int off, int len);

// 写入到指定的 OutputStream 流
public synchronized void writeTo(OutputStream out) throws IOException;

public synchronized int size(); // 返回元素长度
public synchronized void reset();

public synchronized byte toByteArray()[]; // 导出字节数组
public synchronized String toString();
public synchronized String toString(String charsetName) throws UnsupportedEncodingException; // 指定字符编码，如"UTF-8"
public void close() throws IOException;
</script></code></pre>


### File 文件
`FileInputStream`
**构造函数**
`public FileInputStream(String name) throws FileNotFoundException;`
`public FileInputStream(File file) throws FileNotFoundException;`
`public FileInputStream(FileDescriptor fdObj);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public int read() throws IOException;
public int read(byte b[]) throws IOException;
public int read(byte b[], int off, int len) throws IOException;

public native long skip(long n) throws IOException;
public native int available() throws IOException;
public void close() throws IOException;

public final FileDescriptor getFD() throws IOException;
public FileChannel getChannel();
</script></code></pre>



`FileOutputStream`
**构造函数**
`public FileOutputStream(String name) throws FileNotFoundException;`
`public FileOutputStream(String name, boolean append) throws FileNotFoundException;`
`public FileOutputStream(File file) throws FileNotFoundException;`
`public FileOutputStream(File file, boolean append) throws FileNotFoundException;`
`public FileOutputStream(FileDescriptor fdObj);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public void write(int b) throws IOException;
public void write(byte b[]) throws IOException;
public void write(byte b[], int off, int len) throws IOException;

public void close() throws IOException;

public final FileDescriptor getFD()  throws IOException;
public FileChannel getChannel();
</script></code></pre>


### Buffered 缓冲流
`BufferedInputStream`
**构造函数**
`public BufferedInputStream(InputStream in);`：default 大小 8192 字节
`public BufferedInputStream(InputStream in, int size);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public synchronized int read() throws IOException;
public synchronized int read(byte b[], int off, int len) throws IOException;

public synchronized long skip(long n) throws IOException;
public synchronized int available() throws IOException;

public synchronized void mark(int readlimit);
public synchronized void reset() throws IOException;
public boolean markSupported();

public void close() throws IOException;
</script></code></pre>



`BufferedOutputStream`
**构造函数**
`public BufferedOutputStream(OutputStream out);`：default 大小 8192 字节
`public BufferedOutputStream(OutputStream out, int size);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public synchronized void write(int b) throws IOException;
public synchronized void write(byte b[], int off, int len) throws IOException;

public synchronized void flush() throws IOException; // 刷新缓冲区
</script></code></pre>


### Data 基本类型
`DataInputStream`
**构造函数**
`public DataInputStream(InputStream in);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public final int read(byte b[]) throws IOException;
public final int read(byte b[], int off, int len) throws IOException;

// 尽量读满
public final void readFully(byte b[]) throws IOException;
public final void readFully(byte b[], int off, int len) throws IOException;

public final int skipBytes(int n) throws IOException;

public final boolean readBoolean() throws IOException;
public final byte readByte() throws IOException;
public final int readUnsignedByte() throws IOException;
public final short readShort() throws IOException;
public final int readUnsignedShort() throws IOException;
public final char readChar() throws IOException;
public final int readInt() throws IOException;
public final long readLong() throws IOException;
public final float readFloat() throws IOException;
public final double readDouble() throws IOException;

public final String readUTF() throws IOException; // modified_UTF-8 变种UTF-8，专用于序列化
</script></code></pre>



`DataOutputStream`
**构造函数**
`public DataOutputStream(OutputStream out);`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public synchronized void write(int b) throws IOException;
public synchronized void write(byte b[], int off, int len) throws IOException;

public void flush() throws IOException;
public final int size(); // 返回已写入的字节数

public final void writeBoolean(boolean v) throws IOException;
public final void writeByte(int v) throws IOException;
public final void writeShort(int v) throws IOException;
public final void writeChar(int v) throws IOException;
public final void writeInt(int v) throws IOException;
public final void writeLong(long v) throws IOException;
public final void writeFloat(float v) throws IOException;
public final void writeDouble(double v) throws IOException;

public final void writeUTF(String str) throws IOException; // 编码为 modified_UTF-8 字节流
</script></code></pre>


### Object 引用类型
`ObjectInputStream`
**构造函数**
`public ObjectInputStream(InputStream in) throws IOException;`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public final Object readObject() throws IOException, ClassNotFoundException; // 反序列化对象
public void defaultReadObject() throws IOException, ClassNotFoundException; // 默认的反序列化操作

public int read() throws IOException;
public int read(byte[] buf, int off, int len) throws IOException;

public void readFully(byte[] buf) throws IOException;
public void readFully(byte[] buf, int off, int len) throws IOException;

public int skipBytes(int len) throws IOException;
public int available() throws IOException;
public void close() throws IOException;

public boolean readBoolean() throws IOException;
public byte readByte() throws IOException;
public int readUnsignedByte()  throws IOException;
public char readChar()  throws IOException;
public short readShort()  throws IOException;
public int readUnsignedShort() throws IOException;
public int readInt()  throws IOException;
public long readLong()  throws IOException;
public float readFloat() throws IOException;
public double readDouble() throws IOException;

public String readUTF() throws IOException;
</script></code></pre>



`ObjectOutputStream`
**构造函数**
`public ObjectOutputStream(OutputStream out) throws IOException;`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public final void writeObject(Object obj) throws IOException; // 序列化对象
public void defaultWriteObject() throws IOException; // 默认的序列化操作

public void write(int val) throws IOException;
public void write(byte[] buf) throws IOException;
public void write(byte[] buf, int off, int len) throws IOException;

public void reset() throws IOException;
public void flush() throws IOException;
public void close() throws IOException;

public void writeBoolean(boolean val) throws IOException;
public void writeByte(int val) throws IOException;
public void writeShort(int val)  throws IOException;
public void writeChar(int val)  throws IOException;
public void writeInt(int val)  throws IOException;
public void writeLong(long val)  throws IOException;
public void writeFloat(float val) throws IOException;
public void writeDouble(double val) throws IOException;

public void writeUTF(String str) throws IOException;
</script></code></pre>



**自定义序列化**
readObject()、writeObject() 方法先检查序列化对象的类是否已定义 readObject()、writeObject() 方法，如果有那么就执行自定义的序列化操作，否则执行 defaultReadObject()、defaultWriteObject() 默认操作；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.io.*;

public class Main {
    public static void main(String[] args) {
        Student s = new Student("Otokaze", 18, 113.5f);

        try {
            ObjectOutputStream objOut = new ObjectOutputStream(new FileOutputStream("Student.ser"));
            objOut.writeObject(s);
            objOut.close();
        } catch (IOException e) {
            System.err.println(e.getMessage());
            System.exit(1);
        }

        s = null;
        try {
            ObjectInputStream objIn = new ObjectInputStream(new FileInputStream("Student.ser"));
            s = (Student) objIn.readObject();
            System.out.println(s);
            objIn.close();
        } catch (IOException | ClassNotFoundException e) {
            System.err.println(e.getMessage());
            System.exit(1);
        }
    }
}

class Student implements Serializable {
    private static final long serialVersionUID = 1L;

    private String name;
    private int age;
    private float score;

    public Student(String name, int age, float score) {
        this.name = name;
        this.age = age;
        this.score = score;
    }

    @Override
    public String toString() {
        return String.format("name: %s, age: %d, score: %.1f", name, age, score);
    }

    // 自定义 writeObject()
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject(); // 先调用默认的 writeObject() 方法
        out.writeLong(System.currentTimeMillis()); // 记录序列化的时间
    }

    // 自定义 readObject()
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject(); // 先调用默认的 readObject() 方法
        System.out.printf("lastModified: %1$tF %1$tT\n", in.readLong()); // 打印序列化的时间
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [7:59:07]
$ javac Main.java

# root @ arch in ~/work on git:master x [7:59:23]
$ java Main
lastModified: 2017-09-21 07:59:25
name: Otokaze, age: 18, score: 113.5
</script></code></pre>



**对象序列化**
序列化是一种对象持久化的手段，普遍应用在网络传输、RMI（远程方法调用）等场景中；

Java 平台允许我们在内存中创建可复用的 Java 对象，但一般情况下，只有当 JVM 处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比 JVM 的生命周期更长；但在现实应用中，就可能要求在 JVM 停止运行之后能够保存(持久化)指定的对象，并在将来重新读取被保存的对象；Java 对象序列化就能够帮助我们实现该功能；

使用 Java 对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象；必须注意地是，对象序列化保存的是对象的”状态”，即它的成员变量；由此可知，对象序列化不会关注类中的静态变量；

除了在持久化对象时会用到对象序列化之外，当使用 RMI(远程方法调用)，或在网络中传递对象时，都会用到对象序列化；Java 序列化 API 为处理对象序列化提供了一个标准机制，该 API 简单易用；

> 在 Java 中，只要一个类实现了`java.io.Serializable`接口，那么它就可以被序列化；

**序列化及反序列化相关知识**
1、在 Java 中，只要一个类实现了 java.io.Serializable 接口，那么它就可以被序列化；
2、通过 ObjectOutputStream 和 ObjectInputStream 对对象进行序列化及反序列化；
3、虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（`serialVersionUID`）
4、序列化并不保存静态变量；
5、要想将父类对象也序列化，就需要让父类也实现 Serializable 接口；
6、`transient`（短暂的）关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 整型的是 0，引用类型的是 null；
7、Java 允许我们自己定义对象的序列化及反序列化操作，只需要在类中定义两个方法：
`private void writeObject(ObjectOutputStream out) throws IOException`
`private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException`
8、`serialVersionUID`可以由 IDE 生成，也可以通过 jdk 自带的`serialver`命令生成；

如果一个类想被序列化，需要实现 Serializable 接口。否则将抛出 NotSerializableException 异常；
这是因为，在序列化操作过程中会对类型进行检查，要求被序列化的类必须属于`String`、`Array`、`Enum`和`Serializable`类型其中的任何一种。


### Print 格式化
**构造函数**
`public PrintStream(OutputStream out);`：autoFlush = false；
`public PrintStream(OutputStream out, boolean autoFlush);`
`public PrintStream(OutputStream out, boolean autoFlush, String encoding) throws UnsupportedEncodingException;`

`public PrintStream(String fileName) throws FileNotFoundException;`
`public PrintStream(String fileName, String csn) throws FileNotFoundException, UnsupportedEncodingException;`

`public PrintStream(File file) throws FileNotFoundException;`
`public PrintStream(File file, String csn) throws FileNotFoundException, UnsupportedEncodingException;`

> 
`autoFlush`，即自动刷新缓冲区，对于没有缓冲区的输出字节流来说，该参数没有意义；
如果输出流有 buffer 缓冲区，并且 autoFlush 的值为 true，那么 PrintStream 在这几种情况下会进行 flush() 调用：
1) 写入一个`byte[]`字节数组时，自动刷新缓冲区；
2) 写入一个'\n'换行符时，自动刷新缓冲区；
3) 调用 println() 时，自动刷新缓冲区；

到现在为止，我们可以创建自定义的 stdin、stdout、stderr 标准输入输出文件了：
1) `InputStream stdin = new FileInputStream(FileDescriptor.in);`
2) `PrintStream stdout = new PrintStream(new FileOutputStream(FileDescriptor.out));`
3) `PrintStream stderr = new PrintStream(new FileOutputStream(FileDescriptor.err));`

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public void flush();
public void close();
public boolean checkError(); // 检查是否发生 Error

public void write(int b);
public void write(byte buf[], int off, int len);

public void print(boolean b);
public void print(char c);
public void print(int i);
public void print(long l);
public void print(float f);
public void print(double d);
public void print(char s[]);
public void print(String s);
public void print(Object obj);

public void println();
public void println(boolean x);
public void println(char x);
public void println(int x);
public void println(long x);
public void println(float x);
public void println(double x);
public void println(char x[]);
public void println(String x);
public void println(Object x);

// printf() 内部调用 format()，是 format() 的一个别名
public PrintStream printf(String format, Object ... args);
public PrintStream printf(Locale l, String format, Object ... args);
public PrintStream format(String format, Object ... args);
public PrintStream format(Locale l, String format, Object ... args);

public PrintStream append(char c);
public PrintStream append(CharSequence csq); // CharSequence接口：String、StringBuffer、StringBuilder
public PrintStream append(CharSequence csq, int start, int end);
</script></code></pre>
