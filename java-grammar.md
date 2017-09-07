---
title: Java 语法基础
date: 2017-09-07 13:03:00
categories:
- java
tags:
- java
keywords: Java 语法基础
---

> 
Java 语法基础，基本数据类型、数据类型转换、运算符、流程控制、数组、String、StringBuffer、StringBuilder字符串等

<!-- more -->

## 基本数据类型
Java 中共有 8 种基本数据类型，包括 4 种整型、2 种浮点型、1 种字符型、1 种布尔型：
- `byte`：字节型，长度 1 byte；
- `short`：短整型，长度 2 bytes；
- `int`：整型，长度 4 bytes；
- `long`：长整型，长度 8 bytes；数值最后需要加上后缀`l`（大小写无所谓）；
- `float`：单精度浮点型，长度 4 bytes；数值最后需要加上后缀`f`（大小写无所谓）；
- `double`：双精度浮点型，长度 8 bytes；数值最后可以加上后缀`d`（大小写无所谓）；
- `char`：字符型，长度 2 bytes，16 位 Unicode 字符；字符用单引号包围，字符串用双引号包围；
- `boolean`：布尔型，长度 1 bit，取值：`true`、`false`；


> 
Java 不支持`无符号类型(unsigned)`；C/C++ 有无符号类型；
在 Java 中，整型数据的长度与平台无关，这就解决了软件从一个平台移植到另一个平台时给程序员带来的诸多问题；

二进制有一个前缀`0b`，例如`0b1001`对应十进制中的`9`；（Java 7 起）
八进制有一个前缀`0`，例如`010`对应十进制中的`8`；
十六进制有一个前缀`0x`，例如`0xCAFE`对应十进制中的`51966`；

从 Java 7 开始，可以使用下划线来分隔数字，例如`1_000_000`表示`1,000,000`，也就是一百万；下划线只是为了让代码更加易读，编译器会删除这些下划线；


## 数据类型转换
数据类型的转换，分为`自动转换`和`强制转换`：
- 自动转换是程序在执行过程中“悄然”进行的转换，不需要用户提前声明，一般是**从位数低的类型向位数高的类型转换**，自动转换不会丢失精度；
- 强制类型转换则必须在代码中声明，使用`(dst_type)src_data`语法进行强制转换，可能会丢失精度；


**自动转换按从低到高的顺序转换**，优先级从低到高依次为：`byte/short/char` -> `int` -> `long` -> `float` -> `double`；


## 运算符
Java 中的运算符和 C/C++ 相差无几；

1) 数学运算符：`+`、`-`、`*`、`/`、`%`、`++`、`--`，结果为一个数值；
2) 关系运算符：`>`、`>=`、`<`、`<=`、`==`、`!=`、`&&`、`||`、`!`，结果为一个 bool 值；
3) 位运算符：`&`、`|`、`^`、`~`、`<<`、`>>`，结果为一个数值；
4) 条件运算符：`condition ? x1 : x2`，condition 为一个 bool 值；根据 condition，取 x1 或 x2 的值；


## 流程控制
Java 流程控制的语法与 C/C++ 类似，也有 if...else、while、do...while、for、switch...case 等；

这里说一下 switch...case 的注意事项：
- switch 语句中的变量类型可以是：byte、short、int、char，还有枚举；从 Java SE 7 开始，switch 支持字符串 String 类型了，同时 case 标签必须为字符串常量或字面量；
- switch 语句可以拥有多个 case 语句；每个 case 后面跟一个要比较的值和冒号；
- case 语句中的值的数据类型必须与变量的数据类型相同，而且只能是常量或者字面常量；
- 当变量的值与 case 语句的值相等时，那么 case 语句之后的语句开始执行，直到 break 语句出现才会跳出 switch 语句；
- 当遇到 break 语句时，switch 语句终止；程序跳转到 switch 语句后面的语句执行；case 语句不必须要包含 break 语句；如果没有 break 语句出现，程序会继续执行下一条 case 语句，直到出现 break 语句；
- switch 语句可以包含一个 default 分支，该分支必须是 switch 语句的最后一个分支；default 在没有 case 语句的值和变量值相等的时候执行；default 分支不需要 break 语句；


Java 从 JDK 1.5.0 开始引入 foreach 循环，和 C++11 中的基于范围的 for 循环相似：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        int[] arr1 = {1, 2, 3, 4, 5};

        out.printf("arr1_before: %s\n", Arrays.toString(arr1));
        for (int i : arr1) {
            i += 100;
        }
        out.printf("arr1_after: %s\n", Arrays.toString(arr1));

        out.printf("-------------------------------------\n");

        StringBuffer[] arr2 = {new StringBuffer("a"), new StringBuffer("b"), new StringBuffer("c"), new StringBuffer("d"), new StringBuffer("e")};

        out.printf("arr2_before: %s\n", Arrays.toString(arr2));
        for (StringBuffer i : arr2) {
            i.append("$");
        }
        out.printf("arr2_after: %s\n", Arrays.toString(arr2));
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:39:11]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:39:13]
$ java Main
arr1_before: [1, 2, 3, 4, 5]
arr1_after: [1, 2, 3, 4, 5]
-------------------------------------
arr2_before: [a, b, c, d, e]
arr2_after: [a$, b$, c$, d$, e$]
</script></code></pre>


为什么同样是数组，arr1 就不变，arr2 就变了呢？
要搞清楚这个问题其实很简单：**Java 中只有`按值传递`，没有`引用/指针传递`**
在 C 语言中，有`按值传递`、`指针传递`两种方式；
在 C++ 中，有`按值传递`、`指针传递`、`左值引用`、`右值引用`四种方式；
在 Java 中，只有`按值传递`一种方式；

对于 arr1，它是一个整型数组，for 循环中的变量 i 和数组中的元素毫无关联，因为这是两份不同的内存；
对于 arr2，它是一个对象数组，for 循环中的变量 i 和数组中的元素毫无关联，因为这是两份不同的内存；

那为啥 arr2 就改变了了？
因为 arr2 中的变量 i 保存的值有点特殊，它保存的是一个`对象指针`，也就是内存地址，有了内存地址，就可以进行修改了；

虽然大家都说 Java 中没有指针、引用，但是其实 Java 中除了 8 种基本类型之外，都是指针/引用！
比如上面的`StringBuffer`，创建对象的语句：`StringBuffer str = new StringBuffer("a");`；
和 C++ 中在堆上创建对象的语句（以 Student 类为例）：`Student *stu = new Student("name", 18, 111.5f);`；
不知道聪明的你看出来什么没有，没错，java 创建的对象都是在堆上的，而表面上的数据类型`StringBuffer`实质上是一个指针类型`StringBuffer *`，只不过不需要显示指明罢了；

那么这下就更好理解了，对于 arr2，这不就是一个`指针数组`嘛，数组的每个元素都是`StringBuffer *`指针类型，当然可以在循环体内部调用成员函数进行修改了；


## 数组
和 C/C++ 一样，Java 也支持数组这种非常常用的数据结构；Java 提供的数组是用来存储`固定大小`的`同类型`元素；

> 
Java 中的数组和 C/C++ 中的数组有着本质的不同；
Java 中的数组其实是一个对象，它包装了 C/C++ 类型的数组，提供了一些有用的方法和属性；
比如我们可以直接从一个数组对象中获取数组的长度信息（通过`length`属性），但是在 C/C++ 中就必须利用`sizeof`进行长度的计算；

Java 中**定义数组**的语法有两种：
- `type[] arrayName;`：Java 风格；
- `type arrayName[];`：C/C++ 风格；


type 为 Java 中的任意数据类型，包括基本类型和组合类型，arrayName 为数组名，必须是一个合法的标识符，[]指明该变量是一个数组类型变量；
注意，`[]`符号的作用是声明该变量是一个数组对象，不能在里面填写数组的长度；

但是现在我们只是声明了一个数组对象，还没有进行初始化；也就是还没有分配数组元素的内存（数组对象的内存已经分配了，即8个字节，*64位环境*）

**静态初始化、动态初始化**
在定义的同时进行赋值就叫做`静态初始化`，在定义之后进行赋值就叫做`动态初始化`；
- `静态初始化`：比如初始化一个拥有 5 个元素的 int 数组：`int[] arr = {1, 2, 3, 4, 5};`
- `动态初始化`：定义：`int[] arr = new int[5];`，然后进行一一赋值：`arr[0] = 1; arr[1] = 2; arr[2] = 3; arr[3] = 4; arr[4] = 5;`


例子，计算输入的 5 个数的和：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        int arr[] = new int[5], sum = 0;

        Scanner sc = new Scanner(in);
        out.printf("请输入 5 个整数: ");

        for (int i=0; i<arr.length; i++) {
            arr[i] = sc.nextInt();
            sum += arr[i];
        }

        out.printf("输入的数字为: %s，它们的和为: %d\n", Arrays.toString(arr), sum);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:38:58]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:39:01]
$ java Main
请输入 5 个整数: 1 2 3 4 5 6
输入的数字为: [1, 2, 3, 4, 5]，它们的和为: 15
</script></code></pre>


注意这条语句：`int arr[] = new int[5], sum = 0;`
如果定义数组使用的是后置形式，即`int arr[], sum;`，那么表示`arr`是一个数组，而`sum`是一个 int 类型的变量；
如果定义数组使用的是前置形式，即`int[] arr, sum;`，那么表示`arr`是一个数组，而`sum`也是一个数组；
所以，相比之下，我还是习惯使用 C/C++ 风格的数组定义方式，免得出错；


**二维数组**
二维数组，就是数组的数组，和 C/C++ 不一样的地方是，二维数组的第二维度的长度可以不一样，因为 Java 中的二维数组实际上是一个指针数组，第一维度的数组保存的是一组指针（数组对象的指针），第二维度的数组才是数组对象；

多说无益，看例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        int arr1[][] = {{1, 2}, {3, 4}, {5, 6}}; // 静态初始化

        out.printf("arr1 = { ");
        for (int[] i : arr1) {
            out.printf("%s, ", Arrays.toString(i));
        }
        out.printf("\b\b }\n");

        int arr2[][] = new int[3][];  // 分配第一维度数组的内存
        arr2[0] = new int[1]; arr2[0][0] = 1;
        arr2[1] = new int[2]; arr2[1][0] = 11; arr2[1][1] = 22;
        arr2[2] = new int[3]; arr2[2][0] = 111; arr2[2][1] = 222; arr2[2][2] = 333;

        out.printf("arr2 = { ");
        for (int[] i : arr2) {
            out.printf("%s, ", Arrays.toString(i));
        }
        out.printf("\b\b }\n");
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:17:22]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:17:37]
$ java Main
arr1 = { [1, 2], [3, 4], [5, 6] }
arr2 = { [1], [11, 22], [111, 222, 333] }
</script></code></pre>


关于数组的几点说明：
- 上面讲的是`静态数组`；静态数组一旦被声明，它的容量就固定了，不能改变；所以在声明数组时，一定要考虑数组的最大容量，防止容量不够的现象；
- 如果想在运行程序时改变容量，就需要用到`数组列表`(`ArrayList`，也称`动态数组`)或`向量(Vector)`；


## 字符串类
线程安全：
- `StringBuffer`：线程安全；
- `StringBuilder`：线程不安全；


速度从快到慢为`StringBuilder` > `StringBuffer` > `String`，当然这是相对的，不是绝对的；

使用环境：
- 操作少量的数据使用 String；
- 单线程操作大量数据使用 StringBuilder；
- 多线程操作大量数据使用 StringBuffer；


**String 类**
字符串就是双引号之间的数据，如`"www.zfl9.com"`；
像这样定义的字符串称为`字符串常量`，被硬编码到代码区，如果定义了多个相同的字符串常量，那么 Java 并不会创建多个副本，而是将其他的字符串常量都指向已存在的字符串；

我们来验证一下：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        String s1 = "www.zfl9.com";
        String s2 = "www.zfl9.com";
        String s3 = "www.zfl9.com";

        boolean ret = s1 == s2 && s1 == s3;   // == 比较的是内存地址
        out.printf("s1 == s2 == s3 ? %b\n", ret);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:45:49] C:1
$ javac Main.java

# root @ arch in ~/work on git:master x [16:45:52]
$ java Main
s1 == s2 == s3 ? true
</script></code></pre>


大家可能会感到疑惑，为什么可以直接把一个字符串常量赋值给一个 String 类型的变量呢？
我们以 C++ 的角度看待这个问题，这其实就是`转换构造函数`的作用：
这个构造函数的原型类似：`String(const char *str);`，而调用`String s = "www.zfl9.com";`时，实质是调用函数`String s("www.zfl9.com");`；
这样就让我们产生了一个错觉，可以直接将一个`const char *`类型的字符串常量赋值给一个类`String`的变量；

不过在 Java 中我们并不能显示的定义转换构造函数，所以这个转换操作是由编译器隐式完成的；

字符串也可以通过“+”连接，基本数据类型与字符串进行“+”操作一般也会自动转换为字符串（本质：转换构造函数和运算符重载`+`）

不过有一个问题要注意，因为字符串常量是"硬编码"到代码区的，所以在运行过程中不能修改字符串的值！
对于 String 的修改，其实编译器会创建一个临时字符串，然后将 String 重新指向到这个临时字符串；
所以大量频繁的 String 修改操作，会产生大量的`malloc + free`操作，这个开销是巨大的！

String 是 java.lang 包下的一个类，按照标准的面向对象的语法，其格式应该为：`String s = new String("www.zfl9.com");`

但是由于 String 特别常用，所以 Java 提供了一种简化的语法；
使用简化语法的另外一个原因是，按照标准的面向对象的语法，在内存使用上存在比较大的浪费；
例如`String s = new String("www.zfl9.com");`实际上创建了两个 String 对象，一个是`"www.zfl9.com"`对象，存储在`常量空间`中，一个是使用 new 关键字为对象 s 申请的空间；


**String 主要方法**
String 对象有很多方法，可以方便的操作字符串；

1) `length()`方法，返回字符串的长度；
2) `charAt(i)`方法，返回索引 i 的字符；
3) `contains("sub_str")`方法，用来检测字符串是否包含某个子串；
4) `replace("sub_str", "rep_str")`方法，字符串替换；
5) `split("sep")`方法，字符串分割，返回分割后的字符串数组；


**StringBuffer 类**
String 的值是不可变的，每次对 String 的操作都会生成新的 String 对象，不仅效率低，而且耗费大量内存空间；

StringBuffer 类和 String 类一样，也用来表示字符串，但是 StringBuffer 的内部实现方式和 String 不同，在进行字符串处理时，不生成新的对象，在内存使用上要优于 String；
StringBuffer 默认分配 16 字节长度的缓冲区，当字符串超过该大小时，会自动增加缓冲区长度，而不是生成新的对象；

StringBuffer 不像 String，只能通过 new 来创建对象，不支持简写方式，例如：
<pre><code class="language-java line-numbers"><script type="text/plain">StringBuffer str1 = new StringBuffer();     // 分配 16 个字节长度的缓冲区
StringBuffer str2 = new StringBuffer(512); // 分配 512 个字节长度的缓冲区
StringBuffer str3 = new StringBuffer("www.zfl9.com"); // 在缓冲区中存放字符串，并在后面预留了 16 个字节长度的空缓冲区
</script></code></pre>


**StringBuffer 主要方法**
StringBuffer 类中的方法主要偏重于对于字符串的操作，例如追加、插入和删除等，这个也是 StringBuffer 类和 String 类的主要区别；实际开发中，如果需要对一个字符串进行频繁的修改，建议使用 StringBuffer；

1) `append("new_str")`方法，用于向当前字符串的末尾追加内容，类似于字符串的连接；实参可以为其他的类型，都会自动的转换为字符串再进行拼接；
2) `insert(i, "str")`方法，在指定位置 i 插入字符串 str；
3) `setCharAt(i, 'C')`方法，修改指定位置 i 的字符为 C；
4) `deleteCharAt(i)`方法，删除指定位置 i 的字符；
5) `delete(beg, end)`方法，删除区间`[beg, end)`的字符串；

String 和 StringBuffer 速度对比：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String args[]) {
        String base = "www.zfl9.com";
        int cnt = 10000;

        // String + 操作
        long start1 = currentTimeMillis();
        String s1 = "";
        for (int i=0; i<cnt; i++) {
            s1 += base;
        }
        long end1 = currentTimeMillis();
        out.printf("String: %d ms\n", end1 - start1);

        // StringBuffer append() 方法
        long start2 = currentTimeMillis();
        StringBuffer s2 = new StringBuffer();
        for (int i=0; i<cnt; i++) {
            s2.append(base);
        }
        long end2 = currentTimeMillis();
        out.printf("StringBuffer: %d ms\n", end2 - start2);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:46:47]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:46:49]
$ java Main
String: 2004 ms
StringBuffer: 1 ms
</script></code></pre>


**StringBuilder 类**
StringBuilder 类和 StringBuffer 类功能基本相似，方法也差不多，主要区别在于 StringBuffer 类的方法是`多线程安全`的，而 StringBuilder 不是线程安全的，相比而言，StringBuilder 类会略微快一点；

> 
StringBuffer、StringBuilder、String 中都实现了 CharSequence 接口；
CharSequence 是一个定义字符串操作的接口，它只包括 length()、charAt(int index)、subSequence(int start, int end) 这几个 API；


## 输入与输出
**Scanner 类**
在 C/C++ 中，我们可以使用 scanf() 函数获取用户的输入，但是在 Java 中并没有 scanf() 这样的函数；
`java.util.Scanner`是 Java5 的新特征，我们可以通过 Scanner 类来获取用户的输入；

首先需要创建一个 Scanner 对象用于获取输入：`Scanner sc = new Scanner(System.in);`

然后通过 Scanner 类的 next() 与 nextLine() 方法获取输入的字符串，在读取前我们一般需要使用 hasNext() 与 hasNextLine() 判断是否还有输入的数据；
next() 获取的是一个不含有空白符的字符串，nextLine() 则是获取一行字符串；

同时我们也可以获取其他基本类型的数据，比如获取 int 类型的输入使用函数 nextInt()，并且，一般会先判断下一个数据是否为 int 类型，可以使用函数 hasNextInt() 来判断；

对于其他的基本类型，都是这种通用的格式，获取 xxx 类型的输入：nextXxx()，判断下一个数据是否为 xxx 类型：hasNextXxx()

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String args[]) {
        Scanner sc = new Scanner(in);

        // String
        out.printf("String: ");
        out.printf("get_String: %s\n", sc.next());
        sc.nextLine();

        // Line
        out.printf("Line: ");
        out.printf("get_Line: %s\n", sc.nextLine());

        // boolean
        while (true) {
            out.printf("boolean: ");
            if (sc.hasNextBoolean()) {
                out.printf("get_boolean: %b\n", sc.nextBoolean());
                break;
            } else {
                sc.next();
            }
        }

        // char
        out.printf("char: ");
        out.printf("get_char: %c\n", sc.next().charAt(0));

        // short
        while (true) {
            out.printf("short: ");
            if (sc.hasNextShort()) {
                out.printf("get_short: %d\n", sc.nextShort());
                break;
            } else {
                sc.next();
            }
        }

        // int
        while (true) {
            out.printf("int: ");
            if (sc.hasNextInt()) {
                out.printf("get_int: %d\n", sc.nextInt());
                break;
            } else {
                sc.next();
            }
        }

        // long
        while (true) {
            out.printf("long: ");
            if (sc.hasNextLong()) {
                out.printf("get_long: %d\n", sc.nextLong());
                break;
            } else {
                sc.next();
            }
        }

        // float
        while (true) {
            out.printf("float: ");
            if (sc.hasNextFloat()) {
                out.printf("get_float: %g\n", sc.nextFloat());
                break;
            } else {
                sc.next();
            }
        }

        // double
        while (true) {
            out.printf("double: ");
            if (sc.hasNextDouble()) {
                out.printf("get_double: %g\n", sc.nextDouble());
                break;
            } else {
                sc.next();
            }
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:04:57]
$ javac Main.java

# root @ arch in ~/work on git:master x [19:05:00]
$ java Main
String: www.zfl9.com www.google.com www.baidu.com
get_String: www.zfl9.com
Line: www.zfl9.com www.google.com www.baidu.com
get_Line: www.zfl9.com www.google.com www.baidu.com
boolean: 1
boolean: true
get_boolean: true
char: false
get_char: f
short: test
short: 100
get_short: 100
int: 200
get_int: 200
long: 300
get_long: 300
float: 400
get_float: 400.000
double: 500.23
get_double: 500.230
</script></code></pre>


**printf 格式化输出**
常用格式控制符：
- `'b'`、`'B'`：boolean 值，`'b'`是小写形式的，`'B'`是大写形式的，下同；
- `'h'`、`'H'`：hashCode 值；
- `'s'`、`'S'`：String 字符串；
- `'c'`、`'C'`：Unicode 字符；
- `'d'`：十进制数字；
- `'o'`：八进制数字；
- `'#o'`：八进制数字，带前缀；
- `'x'`、`'X'`：十六进制数字；
- `'#x'`、`'#X'`：十六进制数字，带前缀；
- `'e'`、`'E'`：浮点数，科学记数法；
- `'f'`：浮点数，小数形式；
- `'g'`、`'G'`：浮点数，自动选择小数形式、科学记数法；
- `'n'`：同`\n`，换行符；
- `'%'`：输出`%`符号；


Date/Time 格式及其综合例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String args[]) {
        // 支持编号，%$nX 形式，n 从 1 开始
        out.printf("name = %1$s, age = %3$d, score = %2$.1f\n", "Otokaze", 111.5f, 12);

        // o|#o x/X|#x/#X 区别
        out.printf("oct: %1$o, oct: %1$#o, hex: %2$x, hex: %2$X, hex: %2$#x, hex: %2$#X\n", 012, 0x1af3);

        // %t|%T 日期格式，后面跟特定的字母以输出不同的格式
        // %ty: 年份，比如 1998 显示后两位 98
        // %tY: 年份，比如 1998 显示四位 1998
        // %tm: 月份，%td: 日号
        // %tb: 月份，简称
        // %tB: 月份，全称
        // %tD: "%tm/%td/%ty"格式化日期
        // %tF: "%tY-%tm-%td"格式化日期

        // %t|%T 时间格式，后面跟特定的字母以输出不同的格式
        // %tH: 时，24小时制
        // %tI: 时，12小时制
        // %tM: 分
        // %tS: 秒
        // %tL: 毫秒
        // %tp: 上下午信息
        // %tR: "%tH:%tM"格式化时间
        // %tT: "%tH:%tM:%tS"格式化时间
        // %tr: "%tI:%tM:%tS %Tp"格式化时间

        // %t|%T 星期格式，后面跟特定的字母以输出不同的格式
        // %ta: 星期，简称
        // %tA: 星期，全称

        // %tc: 输出日期时间星期的完整信息

        Date date = new Date();
        long secs = date.getTime(); // 自 1970-01-01 00:00:00 GMT 起的毫秒数

        // 日期信息
        out.printf("date: %1$tY-%1$tm-%1$td\n", date);
        out.printf("date: %1$tY-%1$tm-%1$td\n", secs);
        out.printf("date: %tF\n", date);

        // 时间信息
        out.printf("time: %1$tH:%1$tM:%1$tS.%1$tL\n", date);
        out.printf("time: %1$tI:%1$tM:%1$tS.%1$tL %Tp\n", date);
        out.printf("time: %tT\n", date);
        out.printf("time: %tr\n", date);

        // 日期时间星期
        out.printf("datetime: %1$tF %1$tT %1$ta\n", date);
        out.printf("datetime: %1$tF %1$tT %1$tA\n", date);

        // 日期时间星期(简写)
        out.printf("datetime: %tc\n", date);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:46:50]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:47:07]
$ java Main
name = Otokaze, age = 12, score = 111.5
oct: 12, oct: 012, hex: 1af3, hex: 1AF3, hex: 0x1af3, hex: 0X1AF3
date: 2017-09-07
date: 2017-09-07
date: 2017-09-07
time: 20:47:08.563
time: 08:47:08.563 PM
time: 20:47:08
time: 08:47:08 PM
datetime: 2017-09-07 20:47:08 Thu
datetime: 2017-09-07 20:47:08 Thursday
datetime: Thu Sep 07 20:47:08 CST 2017
</script></code></pre>


其他的格式控制符请查阅官方 API 文档：[Formatter - Java SE 7](https://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html#syntax)
