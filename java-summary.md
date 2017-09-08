---
title: Java 概述
date: 2017-09-07 08:12:00
categories:
- java
tags:
- java
keywords: java 概述 环境搭建 jdk jre jvm import package
---

> 
Java 概述，C/C++ 和 Java 的区别、安装并配置 JDK、JVM 虚拟机简述、Java 类和对象的概念、package 和 import 用法等

<!-- more -->

## Java语言
Java 是典型的`面向对象`的语言，晚于 C++ 发布，部分语法和思想也参考了 C++；

> 
尽管人们发现 C++ 的语法太复杂，有很多冗余，但是 Java 在设计的时候还是尽可能的接近 C++，降低人们的学习成本；
Java 语法是 C++ 语法的一个“纯净”版，没有头文件、指针运算（也没有指针语法）、结构、联合、运算符重载、虚基类等；

Java 的应用非常广泛：
- Web 开发：Java 非常适合开发大型的企业网站，例如人人网、去哪儿网的后台都是 Java；
- Android 开发：Android 手机上 APP 几乎都是用 Java 开发的，例如 QQ、微信、UC 浏览器；
- 客户端开发：Java 也可以用来开发电脑上的软件，例如 Elicpse、Netbeans；
- 嵌入式应用：嵌入式应用就是在小型电子产品中运行的软件，例如老式手机上的软件、MP3 上的软件；


但是，Java 目前的主要应用方向是`Web 开发`和`Android 开发`，大部分 IT 公司招聘的 Java 程序员也是从事这两方面的工作；

Java 的 GUI 库称不上出色，很多用户不习惯它的界面；Java 编写的客户端资源消耗也比较多；更重要的是，Java 程序必须借助 jvm 才能运行，操作系统默认没有安装 jvm；直接投放市场的面向普通用户的客户端程序，用 Java 开发的很少；

在嵌入式方面，Java 很难操作底层硬件，灵活性较小，而且需要 jvm 支持，占用资源较多，对于配置很低的单片机系统来说有些吃力；所以，在力求高效、小型化、执行关键任务的应用中，最好采用汇编和 C 语言；

Java 虽然是一门功能完善的语言，但是它有自己擅长的方面，也有不擅长的方面，大家在具体项目中要学会取舍；

## JVM虚拟机
Java 具有跨平台的特性，可以“一次编译，到处运行”，在 Windows 下编写的程序，无需任何修改就可以在 Linux 下运行，这是 C 和 C++ 很难做到的（这话有点偏激，说实话现代 C++ 越来越统一了，也能做到“一次编写，到处编译”）；

那么，跨平台是怎样实现的呢？这就要谈及`Java虚拟机（Java Virtual Machine，简称 JVM）`了；

JVM 也是一个软件，不同的平台有不同的版本；我们编写的 Java 源码，编译后会生成一种`.class`文件，称为`字节码文件`；JVM 就是负责将字节码文件翻译成特定平台下的机器码然后运行；
也就是说，只要在不同平台上安装对应的 JVM，就可以运行字节码文件，运行我们编写的 Java 程序；

而这个过程中，我们编写的 Java 程序没有做任何改变，仅仅是通过 JVM 这一中间层，就能在不同平台上运行，真正实现了”一次编译，到处运行“的目的；

JVM 是一个”桥梁“，是一个”中间件“，是实现跨平台的关键，Java 代码首先被编译成平台无关的字节码文件，再由 JVM 将字节码文件翻译成机器语言，从而达到运行 Java 程序的目的；

注意：编译的结果不是生成机器码，而是生成字节码，字节码不能直接运行，必须通过 JVM 翻译成机器码才能运行；不同平台下编译生成的字节码是一样的，但是由 JVM 翻译成的机器码却不一样；

所以，运行 Java 程序必须有 JVM 的支持，因为编译的结果不是机器码，必须要经过 JVM 的再次翻译才能执行；

> 
跨平台的是 Java 程序，不是 JVM；JVM 是用 C/C++ 开发的，是编译后的机器码，不能跨平台，不同平台下需要安装不同版本的 JVM；
JVM 只关心由 JAVA 源文件编译而成的字节码文件，不关心 JAVA 源码文件，运行 JAVA 程序也只需要提供编译后的`.class`文件

**JVM 的执行效率**
Java 推出的前几年，人们有不同的看法，解释字节码肯定比全速运行机器码慢很多，牺牲性能换来跨平台的优势是否值得？

然而，JVM 有一个选项，可以将使用最频繁的字节码翻译成机器码并保存，这一过程被称为即时编译；这种方式确实很有效，致使微软的 .NET 平台也使用了虚拟机；

现在的及时编译器已经相当出色，甚至成了传统编译器的竞争对手，某些情况下甚至超过了传统编译器，原因是 JVM 可以监控运行时信息；例如，即时编译器可以监控使用频率高的代码并进行优化，可以消除函数调用（即“内嵌”）；

但是，Java 毕竟有一些 C/C++ 没有的额外的开销，关键应用程序速度较慢；比如 Java 采用了与平台无关的绘图方式，GUI 程序（客户端程序）执行要慢；虚拟机启动也需要时间；

**JAVA 的客户端市场**
Java 的 GUI 库称不上出色，界面不算友好，大部分用户不太习惯；Java 的客户端资源消耗也比较大，大数据量的应用和功能复杂的应用性能堪忧；

更加不能接受的是，微软因自身利益和 SUN 分家后，Windows 便不再预装 JVM 了，用户安装你的程序之前，必须要安装 JVM 并正确设置，你可以要求普通用户安装你的软件，但是你能期望他了解 JVM 的有关知识并正确安装设置吗？

虽然你可以将 JVM 集成在你的程序中，自动安装并设置，不让用户干预，但是你希望附带一个比你的程序还要大好多的 JVM 吗？一个软件这样做或许可以接受，成千上万个软件都这样做，那用户要安装多少个 JVM？磁盘空间要浪费多少？

所以，直接投放市场的面向普通用户的客户端程序，用 Java 开发的很少，大部分 Java 开发的客户端是给企业内部员工使用；如果你希望从事客户端开发，建议学习 C/C++ 和 .NET，它们在 Window 客户端开发方面有较大的优势；

种种原因，注定了 Java 客户端不利于推向市场，让普通用户接受；不过话又说回来，客户端开发也不是 Java 的初衷，Java 最初是面向嵌入式的，却随着互联网的兴起而快速成长，在 Web 开发上大显身手；

## J2SE、J2EE、J2ME
1998 年 12 月，SUN 公司发布了 Java 1.2，开始使用“Java 2” 这一名称，目前我们已经很少使用 1.2 之前的版本，所以通常所说的 Java 都是指 Java2；

Java 有三个版本，分别为 J2SE、J2EE 和 J2ME：

**`J2SE(Java 2 Platform Standard Edition)`标准版**
J2SE 是 Java 的标准版，主要用于开发客户端（桌面应用软件），例如常用的文本编辑器、下载软件、即时通讯工具等，都可以通过 J2SE 实现；

J2SE 包含了 Java 的核心类库，例如数据库连接、接口定义、输入/输出、网络编程等；学习 Java 编程就是从 J2SE 入手；

**`J2EE(Java 2 Platform Enterprise Edition)`企业版**
J2EE 是功能最丰富的一个版本，主要用于开发高访问量、大数据量、高并发量的网站，例如美团、去哪儿网的后台都是 J2EE；通常所说的 JSP 开发就是 J2EE 的一部分；

J2EE 包含 J2SE 中的类，还包含用于开发企业级应用的类，例如 EJB、servlet、JSP、XML、事务控制等；
J2EE 也可以用来开发技术比较庞杂的管理软件，例如`ERP系统（Enterprise Resource Planning，企业资源计划系统）`；

**`J2ME(Java 2 Platform Micro Edition)`微型版**
J2ME 只包含 J2SE 中的一部分类，受平台影响比较大，主要用于嵌入式系统和移动平台的开发，例如呼机、智能卡、手机（功能机）、机顶盒等；

在智能手机还没有进入公众视野的时候，你是否还记得你的摩托罗拉、诺基亚手机上有很多 Java 小游戏吗？这就是用 J2ME 开发的；Java 的初衷就是做这一块的开发；
> 
注意：Android 有自己的开发组件，不使用 J2ME 进行开发；

Java5.0 版本后，J2SE、J2EE、J2ME 分别更名为 Java SE、Java EE、Java ME，由于习惯的原因，我们依然称之为 J2SE、J2EE、J2ME；

## 配置Java环境
> 
笔者主要使用 Linux + Vim 进行 Java SE 的学习，因此这里的安装环境也是指 Linux 环境；

下载页面：[Java SE Development Kit 8u144](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
选择 Linux 平台的`*.tar.gz`文件；可以直接解压使用，纯绿色软件；

解压之后将 jdk 文件夹拷贝到某个目录，这里选择`/opt/jdk1.8.0`，然后设置环境变量`JAVA_HOME`、`PATH`即可，具体步骤如下：
<pre><code class="language-bash line-numbers"><script type="text/plain"># root @ arch in /opt/jdk1.8.0 [8:56:37]
$ pwd
/opt/jdk1.8.0

# root @ arch in /opt/jdk1.8.0 [8:56:38]
$ ll
total 26M
drwxr-xr-x 2 root root 4.0K Jul 22 13:08 bin
-r--r--r-- 1 root root 3.2K Jul 22 13:07 COPYRIGHT
drwxr-xr-x 4 root root 4.0K Jul 22 13:07 db
drwxr-xr-x 3 root root 4.0K Jul 22 13:07 include
-rwxr-xr-x 1 root root 4.9M Jun 27 04:26 javafx-src.zip
drwxr-xr-x 5 root root 4.0K Jul 22 13:07 jre
drwxr-xr-x 5 root root 4.0K Jul 22 13:08 lib
-r--r--r-- 1 root root   40 Jul 22 13:07 LICENSE
drwxr-xr-x 4 root root 4.0K Jul 22 13:07 man
-r--r--r-- 1 root root  159 Jul 22 13:07 README.html
-rw-r--r-- 1 root root  526 Jul 22 13:07 release
-rw-r--r-- 1 root root  21M Jul 22 13:07 src.zip
-rwxr-xr-x 1 root root  63K Jun 27 04:26 THIRDPARTYLICENSEREADME-JAVAFX.txt
-r--r--r-- 1 root root 142K Jul 22 13:07 THIRDPARTYLICENSEREADME.txt

# root @ arch in /opt/jdk1.8.0 [8:56:39]
$ vim /etc/profile.d/jdk.sh

# root @ arch in /opt/jdk1.8.0 [8:56:49]
$ cat /etc/profile.d/jdk.sh
export JAVA_HOME=/opt/jdk1.8.0
export PATH=$JAVA_HOME/bin:$PATH

# root @ arch in /opt/jdk1.8.0 [8:56:52]
$ source /etc/profile

# root @ arch in /opt/jdk1.8.0 [8:57:03]
$ env | grep -E 'JAVA_HOME|PATH'
PATH=/opt/jdk1.8.0/bin:/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl
JAVA_HOME=/opt/jdk1.8.0

# root @ arch in /opt/jdk1.8.0 [8:57:22]
$ javac -version
javac 1.8.0_144

# root @ arch in /opt/jdk1.8.0 [8:57:44]
$ java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
</script></code></pre>


**Hello World程序**
为了验证 java 环境是否正确配置，我们按照惯例先来一个 Hello World 程序：
<pre><code class="language-java line-numbers"><script type="text/plain">public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:02:27]
$ javac HelloWorld.java

# root @ arch in ~/work on git:master x [9:02:32]
$ java HelloWorld
Hello, World!
</script></code></pre>


Java 和 C/C++ 一样，都是从 main() 函数开始运行，并且 main() 函数的属性必须是`public static`；
一个 Java 源文件中只能有一个 public 属性的 class，并且必须与文件名同名，当然一个源文件中也可以没有 public 属性的 class，并且不强制与文件名相同；

## Java类和对象
Java 中的类可以看作是 C 语言中结构体 struct 的升级版，结构体只能包含成员变量，而类除了可以包含成员变量，还可以包含成员函数；

我们用三个例子感受一下 C、C++、Java 中如何实现一个简单的 Student 类；

C语言：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define NAME_LEN 128

typedef struct Student {
    char name[NAME_LEN];
    int age;
    float score;
} Student;

void print(const Student *stu) {
    printf("name: %s, age: %d, score: %.1f\n", stu->name, stu->age, stu->score);
}

int main(void) {
    Student stu = {"Otokaze", 18, 87.5f};
    print(&stu);
    return 0;
}
</script></code></pre>


C++：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <string>
using namespace std;

class Student {
public:
    Student(string name, int age, float score) : m_name(name), m_age(age), m_score(score) {}
public:
    string getname() const { return m_name; }
    int getage() const { return m_age; }
    float getscore() const { return m_score; }
    void setname(string name) { m_name = name; }
    void setage(int age) { m_age = age; }
    void setscore(float score) { m_score = score; }
public:
    friend ostream & operator<<(ostream &out, const Student &stu);
private:
    string m_name;
    int m_age;
    float m_score;
};

ostream & operator<<(ostream &out, const Student &stu) {
    return out << "name: " << stu.m_name << ", age: " << stu.m_age << ", score: " << stu.m_score << endl;
}

int main() {
    Student stu("Otokaze", 18, 87.5f);  // 栈上创建对象
    cout << stu;

    Student *pstu = new Student("Otokaze", 18, 87.5f);  // 堆上创建对象
    cout << *pstu;
    delete pstu;

    return 0;
}
</script></code></pre>


Java：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Student {
    public static void main(String[] args) {
        Student stu = new Student("Otokaze", 18, 87.5f);    // 堆上创建对象
        stu.print();
    }

    public Student(String name, int age, float score) {
        m_name = name;
        m_age = age;
        m_score = score;
    }

    public String getName() { return m_name; }
    public int getAge() { return m_age; }
    public float getScore() { return m_score; }
    public void setName(String name) { m_name = name; }
    public void setAge(int age) { m_age = age; }
    public void setScore(float score) { m_score = score; }

    public void print() {
        out.printf("name: %s, age: %d, score: %.1f\n", m_name, m_age, m_score);
    }

    private String m_name;
    private int m_age;
    private float m_score;
}
</script></code></pre>


## Java类库结构
Java 类库中有很多包：
- 以`java.*`开头的是 Java 的核心包，所有程序都会使用这些包中的类；
- 以`javax.*`开头的是扩展包，x 是 extension 的意思，也就是扩展；虽然`javax.*`是对`java.*`的优化和扩展，但是由于`javax.*`使用的越来越多，很多程序都依赖于`javax.*`，所以`javax.*`也是核心的一部分了，也随 JDK 一起发布；
- 以`org.*`开头的是各个机构或组织发布的包，因为这些组织很有影响力，它们的代码质量很高，所以也将它们开发的部分常用的类随 JDK 一起发布；


> 
在包的命名方面，为了防止重名，有一个惯例：大家都以自己域名的倒写形式作为开头来为自己开发的包命名，例如百度发布的包会以`com.baidu.*`开头，w3c 组织发布的包会以`org.w3c.*`开头等等；

java 中常用的几个包介绍：
- `java.lang`：该包提供了 Java 编程的基础类，例如 Object、Math、String、StringBuffer、System、Thread 等；
- `java.util`：该包提供了包含集合框架、遗留的集合类、事件模型、日期和时间实施、国际化和各种实用工具类等；
- `java.io`：该包通过文件系统、数据流和序列化提供系统的输入与输出；
- `java.text`：提供了与自然语言无关的方式来处理文本、日期、数字和消息的类和接口；
- `java.net`：该包提供实现网络应用与开发的类；
- `java.sql`：该包提供了使用 Java 语言访问并处理存储在数据源中的数据 API；
- `java.awt`、`javax.swing`：这两个包提供了 GUI 设计与开发的类；java.awt 包提供了创建界面和绘制图形图像的所有类，而 javax.swing 包提供了一组“轻量级”的组件，尽量让这些组件在所有平台上的工作方式相同；


> 
其中，`java.lang.*`由 JAVA 编译器自动 import 到每个源文件中，因此不需要我们再次显示 import；


## package和import
**package 为解决类的命名冲突问题而引入的机制**，和 C++ 中的命名空间是差不多的概念；
> 
**package 语句作为 Java 源文件的第一条语句（若缺省该语句，则指定为无名包）**；

通常 package 的名字都是公司域名的倒写，比如对于域名 google.com，那么包名就是`com.google`，实际文件夹的组织路径为`$CLASSPATH/com/google/*.java|class`（CLASSPATH为类的搜索路径）；

**import 导入**
如果需要使用一个包中的类，那么有两种方式：
- 使用类时带上包名：比如对于包`java.util`中的类`Arrays`，可以这样使用`System.out.println(java.util.Arrays.toString(arr));`
- 使用 import 导入：先导入包`import java.util.*;`，然后`System.out.println(Arrays.toString(arr));`
`import`语句需要在`package`声明语句后面，如果没有 package 语句，那么就是文件首行，可以有多个 import 语句；


这里再说明一下`CLASSPATH`环境变量：
在 jdk1.5 之后，配置 java 环境的时候并不需要显示的设置 CLASSPATH 环境变量了，java 默认会在`.`（当前目录）和`$JAVA_HOME/jre/lib/rt.jar`中搜寻；
如果自己使用的包不在这些路径下面，那么有两种方法：
- 使用`-classpath`参数进行指定：`java -classpath .:/path/to/classpath`，多个可用`:`冒号分隔；
- 使用`CLASSPATH`环境变量指定：`export CLASSPATH=.:/path/to/classpath`，多个可用`:`冒号分隔；


`import static`可以导入 static 成员至当前命名空间：
比如包`com.zfl9`中的类`Demo`中有两个静态成员`static_var`、`static_func()`，前者是一个静态成员变量，后者是一个静态成员函数；
那么可以使用静态导入`import static com.zfl9.Demo.*;`将他们导入到当前命名空间，然后就能直接使用了；

## javac和java
`javac`命令位于`$JAVA_HOME/bin/javac`，是 Java 的编译器，用于将`*.java`源文件编译为`*.class`字节码文件；
`java`命令位于`$JAVA_HOME/bin/java`或`$JAVA_HOME/jre/bin/java`，是 Java 的解释器，用于将`*.class`字节码解释为特定平台的机器码；

`javac`命令可以一次只编译一个源文件：`javac Test.java`，也可以一次编译多个文件：`javac *.java`；

对于 javac，每个类编译后都会生成一个和类名一样的 class 文件，不管这些类在不同文件中还是都在一个文件中；
例如，对于一个源文件`Main.java`，定义了类：`Main`、`A`、`B`、`C`，那么编译后生成的 class 文件为：`Main.class`、`A.class`、`B.class`、`C.class`；

对于 javac、java 命令，都有一个`-classpath`选项，用于指定 class 的搜索路径，但是有些细微的区别：
- `javac`：`-classpath`选项用于指定当前编译的`*.java`源文件的依赖包的搜索路径（即`import`语句的搜索路径）；
默认的搜索路径：当前工作路径`.`和`$JAVA_HOME/jre/lib/rt.jar`
- `java`：`-classpath`选项用于指定需要运行的 class 文件及其依赖包的路径；
默认的搜索路径：当前工作路径`.`和`$JAVA_HOME/jre/lib/rt.jar`
