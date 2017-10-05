---
title: Java Random、regex正则、Math类、String字符串
date: 2017-09-21 11:02:00
categories:
- java
tags:
- java
keywords: Java Random随机数 regex正则 Math类 String StringBuffer StringBuilder类 Date日期时间
---

> 
Java Random、regex正则、Math类、String字符串

<!-- more -->

## Random 随机数
实际上，计算机只能为我们提供`伪随机数`，所谓伪随机数就是按照一定`算法`模拟产生的，其结果是`确定的`，是`可见的`；

计算机产生随机数的过程，是根据一个`种子seed`为基准，以某个`递推公式`推算出来的`一系列数`，当递推的**范围足够大**、**往复性足够强**、又**符合正态分布或平均分布**时，我们就可以认为这是一个近似的真随机数；

在 java.util 包中，Random 类就是一个**伪随机数生成器**。
Random 类的默认种子在 JDK1.5 版本以前是`System.currentTimeMillis()`的返回值；
由于 System.currentTimeMillis() 的粒度太大，随机性不是很好，所以改为了`System.nanoTime()`的返回值。

> 
`System.currentTimeMillis()`：返回自 1970-01-01 00:00:00 GMT 到现在经过的毫秒(ms)数；
`System.nanoTime()`：基于 cpu 核心的`时钟周期`来计时，它的开始时间是不确定的，单位是 nano 纳秒；

大家可能已经发现了，在 java.lang.Math 类中也有一个 random() 静态方法，和 java.util.Random 有什么区别？

看一下 Math.random() 的源码就明白了：
<pre><code class="language-java line-numbers"><script type="text/plain">public static double random() {
    return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();
}

private static final class RandomNumberGeneratorHolder {
    static final Random randomNumberGenerator = new Random();
}
</script></code></pre>



也就是说，Math.random() 就是调用的 java.util.Random 类的 nextDouble() 方法，返回区间`[0.0d, 1.0d)`的 double 随机数；

java.util.Random 的实例是`线程安全`的；但是，跨线程同时使用相同的 java.util.Random 实例可能会导致同步锁的竞争，从而导致性能下降。在多线程设计中考虑使用`ThreadLocalRandom`；

java.util.Random 的实例在`加密`方面不安全；考虑使用`SecureRandom`获得一个`加密安全`的伪随机数生成器，供安全敏感应用程序使用；

**构造函数**
`public Random()`：无参构造函数，`System.nanoTime()`为 seed；
`public Random(long seed)`：使用指定的 seed，long 长整型；

我们使用一样的种子，看一下是不是都是一样的数字序列：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Random;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        long seed = 0L;
        Random r1 = new Random(seed);
        Random r2 = new Random(seed);

        int[] arr1 = new int[10];
        int[] arr2 = new int[10];

        for (int i = 0; i < arr1.length; i++) {
            arr1[i] = r1.nextInt();
        }

        for (int i = 0; i < arr2.length; i++) {
            arr2[i] = r2.nextInt();
        }

        System.out.println(Arrays.toString(arr1));
        System.out.println(Arrays.toString(arr2));
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:03:33] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [12:03:35]
$ java Main
[-1155484576, -723955400, 1033096058, -1690734402, -1557280266, 1327362106, -1930858313, 502539523, -1728529858, -938301587]
[-1155484576, -723955400, 1033096058, -1690734402, -1557280266, 1327362106, -1930858313, 502539523, -1728529858, -938301587]

# root @ arch in ~/work on git:master x [12:03:36]
$ java Main
[-1155484576, -723955400, 1033096058, -1690734402, -1557280266, 1327362106, -1930858313, 502539523, -1728529858, -938301587]
[-1155484576, -723955400, 1033096058, -1690734402, -1557280266, 1327362106, -1930858313, 502539523, -1728529858, -938301587]

# root @ arch in ~/work on git:master x [12:03:41]
$ java Main
[-1155484576, -723955400, 1033096058, -1690734402, -1557280266, 1327362106, -1930858313, 502539523, -1728529858, -938301587]
[-1155484576, -723955400, 1033096058, -1690734402, -1557280266, 1327362106, -1930858313, 502539523, -1728529858, -938301587]
</script></code></pre>



从运行结果看得出，一个 seed 种子就代表了属于它的一系列数字，也说明了 Random 生成的数字是伪随机数；

常用方法：
`synchronized public void setSeed(long seed)`：设置新种子 seed
`public void nextBytes(byte[] bytes)`：随机字节数组
`public int nextInt()`：随机 int 值，该值介于 int 的区间`[-2^31, 2^31 - 1]`
`public int nextInt(int bound)`：随机 int 值，介于区间`[0, bound)`
`public long nextLong()`：随机 long 值，该值介于 long 的区间`[-2^63, 2^63 - 1]`
`public boolean nextBoolean()`：随机 boolean 值，true 或 false
`public float nextFloat()`：随机 float 值，介于区间`[0.0f, 1.0f)`
`public double nextDouble()`：随机 double 值，介于区间`[0.0d, 1.0d)`


生成指定区间`[m, n]`，`m < n`的 int 随机数的方法：`nextInt(n - m + 1) + m;`

观察 Random 对象的 nextInt(int bound) 方法，发现这个方法将生成 0 ~ bound 之间随机取值的整数；包括 0，排除 bound；
比如`rand.nextInt(100);`，生成区间`[0, 100)`之间的整数；

那么如果要获得区间`[1, 100]`的随机数，该怎么办呢？
稍微动动脑筋就可以想到：区间 [0, 100) 内的整数，实际上就是区间 [0, 99]；
因为最大边界为 100，可惜不能等于 100，因此最大可能产生的“整数”就是 99；

既然 rand.nextInt(100) 获得的值是区间 [0, 99]，那么在这个区间左右各加 1，就得到了区间 [1, 100]；
因此，代码写成：`rand.nextInt(100) + 1;`即可；运行下面的代码，将获得 [1, 100] 的 10 个取值：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Random;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        Random rand = new Random();
        int[] arr = new int[10];

        for (int i = 0; i < arr.length; i++) {
            arr[i] = rand.nextInt(100) + 1;
        }

        System.out.println(Arrays.toString(arr));
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:23:12]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:23:20]
$ java Main
[91, 45, 82, 54, 20, 49, 63, 22, 2, 5]

# root @ arch in ~/work on git:master x [13:23:23]
$ java Main
[55, 74, 68, 28, 82, 19, 27, 42, 61, 35]

# root @ arch in ~/work on git:master x [13:23:24]
$ java Main
[99, 40, 83, 12, 23, 8, 14, 99, 98, 27]
</script></code></pre>



同理，很容易知道如果要获得随机的两位整数，代码写成：`rand.nextInt(90) + 10;`
在 nextInt() 方法中作为参数的数字 90 表示要生成的随机数的**个数**（两位整数有 90 个），加上后面的数字 10 ，表示区间的最小取值为 10（含）；

生成随机三位数的代码为：`rand.nextInt(900) + 100;`（[100, 999]）
生成区间 [64, 128] 中随机值的代码为：`rand.nextInt(65) + 64;`

取值可能性的数量是如何计算出来的呢？当然是`最大取值 - 最小取值 + 1`

因此，获取在区间`[min, max]`的随机整数的公式为：`nextInt(max - min + 1) + min;`

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Random;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        Random rand = new Random();
        int[] arr = new int[10];

        // 随机的两位数 [10, 99]
        for (int i = 0; i < arr.length; i++) {
            arr[i] = rand.nextInt(90) + 10;
        }
        System.out.println(Arrays.toString(arr));

        // 随机月份 [1, 12]
        for (int i = 0; i < arr.length; i++) {
            arr[i] = rand.nextInt(12) + 1;
        }
        System.out.println(Arrays.toString(arr));

        // 随机秒数 [0, 59]
        for (int i = 0; i < arr.length; i++) {
            arr[i] = rand.nextInt(60);
        }
        System.out.println(Arrays.toString(arr));
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:44:27]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:44:29]
$ java Main
[42, 51, 11, 83, 94, 20, 42, 52, 60, 20]
[10, 1, 2, 5, 7, 3, 9, 9, 8, 11]
[8, 41, 58, 28, 30, 46, 33, 20, 40, 9]

# root @ arch in ~/work on git:master x [13:44:30]
$ java Main
[40, 64, 59, 29, 19, 53, 97, 89, 76, 58]
[10, 2, 6, 2, 5, 7, 11, 8, 8, 9]
[19, 41, 57, 1, 38, 20, 26, 39, 39, 33]

# root @ arch in ~/work on git:master x [13:44:32]
$ java Main
[35, 52, 79, 78, 22, 26, 33, 66, 64, 79]
[8, 7, 12, 3, 7, 12, 9, 11, 2, 6]
[32, 15, 28, 15, 21, 33, 58, 5, 48, 51]
</script></code></pre>


## Date 日期时间
java.util 包提供了 Date 类来封装当前的日期和时间。Date 类提供两个构造函数来实例化 Date 对象：
1) `Date()`：使用当前日期和时间来初始化对象；
2) `Date(long millisec)`：millisec 参数是从 1970 年 1 月 1 日起的**毫秒数**；

常用方法：
`public long getTime()`：返回从 1970-01-01 00:00:00 GMT 起的**毫秒数**
`public void setTime(long time)`：设置时间，参数同上

`public boolean before(Date when)`：如果 this 对象在 when 对象之前，返回 true，否则返回 false
`public boolean after(Date when)`：如果 this 对象在 when 对象之后，返回 true，否则返回 false

`public boolean equals(Object obj)`：判等
`public int hashCode()`：获取 hashCode 值
`public int compareTo(Date anotherDate)`：比较
`public Object clone()`：拷贝 Date 对象


[时间格式化 System.out.printf()](https://www.zfl9.com/java-grammar.html#输入与输出)

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Date;

public class Main {
    public static void main(String[] args) {
        System.out.printf("current datetime: %1$tF %1$tT\n", System.currentTimeMillis());
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:07:39]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:07:49]
$ java Main
current datetime: 2017-09-21 14:07:51

# root @ arch in ~/work on git:master x [14:07:51]
$ java Main
current datetime: 2017-09-21 14:07:52

# root @ arch in ~/work on git:master x [14:07:52]
$ java Main
current datetime: 2017-09-21 14:07:53
</script></code></pre>


**使用 SimpleDateFormat 格式化日期**
java.text.SimpleDateFormat 是一个以语言环境敏感的方式来格式化和分析日期的类，SimpleDateFormat 允许你选择任何用户自定义日期时间格式来运行；
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Date;
import java.text.SimpleDateFormat;

public class Main {
    public static void main(String[] args) {
        SimpleDateFormat datef = new SimpleDateFormat("E yyyy-MM-dd hh:mm:ss a");
        System.out.println("current datetime: " + datef.format(new Date()));
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:15:59] C:1
$ javac Main.java

# root @ arch in ~/work on git:master x [14:16:01]
$ java Main
current datetime: Thu 2017-09-21 02:16:02 PM
</script></code></pre>



SimpleDateFormat 支持的格式化字符：
`y`年份、`M`月份、`d`日期、`h`12小时制、`H`24小时制、`m`分钟、`s`秒、`S`毫秒、`E`星期几、`a`上午下午标记等

同时，SimpleDateFormat 的 parse(String s) 方法可以从字符串中解析指定格式的 datetime，并返回 Date 实例：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Date;
import java.text.SimpleDateFormat;
import java.text.ParseException;

public class Main {
    public static void main(String[] args) {
        SimpleDateFormat datef = new SimpleDateFormat("yyyy-MM-dd");
        String input = args.length == 0 ? "1999-12-12" : args[0];
        try {
            System.out.println("datetime: " + datef.parse(input));
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:30:14]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:30:17]
$ java Main
datetime: Sun Dec 12 00:00:00 CST 1999

# root @ arch in ~/work on git:master x [14:30:18]
$ java Main 2017-10-1
datetime: Sun Oct 01 00:00:00 CST 2017
</script></code></pre>


## regex 正则
java.util.regex 包主要包括以下三个类：
- Pattern 类：Pattern 对象是一个正则表达式的编译表示；
Pattern 类没有公共构造方法；要创建一个 Pattern 对象，需要调用 Pattern.compile(String regex) 静态方法，它返回一个 Pattern 对象；
- Matcher 类：Matcher 对象是对输入字符串进行解释和匹配操作的引擎；
与 Pattern 类一样，Matcher 也没有公共构造方法；需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象；
- PatternSyntaxException：一个 RuntimeException 异常类，它表示一个正则表达式模式中的语法错误；



Pattern 类：
<pre><code class="language-java line-numbers"><script type="text/plain">public static Pattern compile(String regex); // 编译regex模式
public static Pattern compile(String regex, int flags); // 编译regex模式，指定flag，多个flag用按位或"|"连接

/**
 * 当使用内嵌的 flag 时，也可以存在多个，内嵌方式需要放在开头.
 * 例如: String regex = "(?ix)java"; 忽略大小写|启用注释
 */

// (?i) 忽略大小写
public static final int CASE_INSENSITIVE;
// (?m) 多行模式
public static final int MULTILINE;
// (?s) 单行模式
public static final int DOTALL;
// (?x) 启用注释，空格和#号行将被忽略
public static final int COMMENTS;
// (?d) Unix行模式，行结束符为'\n'
public static final int UNIX_LINES;
// 文字解析，元字符、转义符将没有特殊意义
public static final int LITERAL;
// (?u) UNICODE字符感知，不区分大小写
public static final int UNICODE_CASE;
// 启用规范分解
public static final int CANON_EQ;
// (?U) 启用Unicode预定义字符类、POSIX字符类
public static final int UNICODE_CHARACTER_CLASS;

public String pattern(); // 返回传入的regex模式
public String toString(); // 同 pattern()
public int flags(); // 返回传入的flag标志位

/**
 * 匹配正则表达式.
 * @param input 字符序列, 实现的类有String, StringBuffer, StringBuilder
 * @return Matcher 返回 Matcher 类的实例，匹配的结果
 */
public Matcher matcher(CharSequence input); // 获取 Matcher 匹配器
public static boolean matches(String regex, CharSequence input); // 测试是否完全匹配

/**
 * 分割字符串.
 * @param input 字符序列
 * @param limit 匹配的次数:
 *              1. n < 0
 *                 不限制分割次数，末尾的空串被保留.
 *              2. n == 0
 *                 不限制分割次数，末尾的空串被丢弃.
 *              3. n > 0
 *                 最多分割 n-1 次（String[]长度为n）.
 * @return String[] 返回分割后的字符串数组
 */
public String[] split(CharSequence input, int limit);
public String[] split(CharSequence input); // limit = 0
</script></code></pre>



Matcher 类：
<pre><code class="language-java line-numbers"><script type="text/plain">public Pattern pattern(); // 返回当前 Pattern 模式
public Matcher usePattern(Pattern newPattern); // 设置新 Pattern 模式
public Matcher reset(); // 重置 Matcher 匹配器
public Matcher reset(CharSequence input); // 匹配新的字符序列

public int start(); // 返回首次匹配的索引值
public int end(); // 返回匹配的最后一个字符的索引值 + 1
public int start(int group); // 同上，应用于捕获组
public int end(int group); // 同上，应用于捕获组

public String group(); // 返回group0捕获组的字符串, 即匹配的整个字符串
public String group(int group); // 返回group号的子捕获组的字符串, 从 1 开始
public int groupCount(); // 子捕获组的数量, 不包括group(0)

public boolean matches(); // 要求模式与整个字符序列相匹配
public boolean lookingAt(); // 只要求模式与字符序列的开头相匹配

public boolean find(); // 尝试匹配当前位置的下一个子序列
public boolean find(int start); // 重置到start位置，尝试匹配其下一个子序列

public String replaceFirst(String replacement); // 替换首次匹配
public String replaceAll(String replacement); // 替换所有匹配

public Matcher appendReplacement(StringBuffer sb, String replacement); // 渐进式替换
public StringBuffer appendTail(StringBuffer sb); // 将末尾的字符串append上
</script></code></pre>



1) start()、end() 方法，例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class Main {
    public static void main(String[] args) {
        String str = "java1 java2 java3 java4";
        String regex = "\\bjava\\d\\b";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(str);

        int cnt = 0;
        while (matcher.find()) {
            cnt++;
            System.out.printf("[%d] -> \"%s\" at [%d, %d)\n", cnt, matcher.group(), matcher.start(), matcher.end());
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:01:52] C:130
$ javac Main.java

# root @ arch in ~/work on git:master x [17:01:54]
$ java Main
[1] -> "java1" at [0, 5)
[2] -> "java2" at [6, 11)
[3] -> "java3" at [12, 17)
[4] -> "java4" at [18, 23)
</script></code></pre>



2) matches() 和 lookingAt() 方法，例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class Main {
    public static void main(String[] args) {
        String regex = "foo";
        String str1 = "foooooooooo";
        String str2 = "ofooooooooo";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher1 = pattern.matcher(str1);
        Matcher matcher2 = pattern.matcher(str2);

        System.out.println("regex: " + regex);
        System.out.println("str1: " + str1);
        System.out.println("str2: " + str2);

        System.out.println("matcher1.matches(): " + matcher1.matches()); // false
        System.out.println("matcher1.lookingAt(): " + matcher1.lookingAt()); // true
        System.out.println("matcher2.lookingAt(): " + matcher2.lookingAt()); // false
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:14:27]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:14:37]
$ java Main
regex: foo
str1: foooooooooo
str2: ofooooooooo
matcher1.matches(): false
matcher1.lookingAt(): true
matcher2.lookingAt(): false
</script></code></pre>



3) replaceFirst() 和 replaceAll() 方法，例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class Main {
    public static void main(String[] args) {
        String regex = "java";
        String input = "java java java java java";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(input);

        String result1 = matcher.replaceAll("Java"); // 替换全部
        System.out.println(result1);

        String result2 = matcher.replaceFirst("Java"); // 替换第一个
        System.out.println(result2);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:21:20]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:21:23]
$ java Main
Java Java Java Java Java
Java java java java java
</script></code></pre>



4) appendReplacement() 和 appendTail() 方法，例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class Main {
    public static void main(String[] args) {
        String regex = "java";
        String input = "java java java java java";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(input);

        StringBuffer sb = new StringBuffer();
        while (matcher.find()) {
            matcher.appendReplacement(sb, matcher.group().toUpperCase());
            System.out.println(sb);
        }
        matcher.appendTail(sb);
        System.out.println(sb);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:40:55]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:41:00]
$ java Main
JAVA
JAVA JAVA
JAVA JAVA JAVA
JAVA JAVA JAVA JAVA
JAVA JAVA JAVA JAVA JAVA
JAVA JAVA JAVA JAVA JAVA
</script></code></pre>


## Math 类
java.lang.Math 包含了用于执行基本数学运算的属性和方法，如初等指数、对数、平方根和三角函数。
Math 类的方法都被定义为 static 形式，不需要也不能创建 Math 类的对象；

常用方法：
<pre><code class="language-java line-numbers"><script type="text/plain">public static final double E = 2.7182818284590452354; // 常数E
public static final double PI = 3.14159265358979323846; // 圆周率PI

public static double sin(double a); // sin正弦
public static double cos(double a); // cos余弦
public static double tan(double a); // tan正切
public static double asin(double a); // 反正弦
public static double acos(double a); // 反余弦
public static double atan(double a); // 反正切

public static double toRadians(double angdeg); // 将角度转换为弧度
public static double toDegrees(double angrad); // 将参数转化为角度

public static double exp(double a); // 计算 Math.E ^ a

public static double log(double a); // 计算以 Math.E 为底 a 的对数
public static double log10(double a); // 计算以 10 为底 a 的对数

public static double sqrt(double a); // 计算 a 的算术平方根
public static double cbrt(double a); // 计算 a 的立方根

public static double ceil(double a); // 上舍入运算，如 1.1 -> 2.0
public static double floor(double a); // 下舍入运算，如 1.1 -> 1.0
public static double rint(double a); // 返回与 a 最接近的整数
public static int round(float a); // 返回最接近 a 的 int 型值
public static long round(double a); // 返回最接近 a 的 long 型值

public static double atan2(double y, double x); // 将笛卡尔坐标转换为极坐标，并返回极坐标的角度值

public static double pow(double a, double b); // 计算 a ^ b 的值

public static double random(); // new Random().nextDouble(), 区间[0.0d, 1.0d)的随机浮点数

public static int abs(int a); // 绝对值
public static long abs(long a);
public static float abs(float a);
public static double abs(double a);

public static int max(int a, int b); // maximum值
public static long max(long a, long b);
public static float max(float a, float b);
public static double max(double a, double b);

public static int min(int a, int b); // minimum值
public static long min(long a, long b);
public static float min(float a, float b);
public static double min(double a, double b);
</script></code></pre>


## Arrays 类
java.util.Arrays 类能方便地操作数组，它提供的所有方法都是静态的。

Arrays 类具有以下功能：
1) 给数组赋值：通过 fill 方法；
2) 对数组排序：通过 sort 方法，升序排序
3) 比较数组：通过 equals 方法比较数组中元素值是否相等；
4) 查找数组元素：通过 binarySearch 方法能对排序好的数组进行二分查找法操作。

常用方法：
<pre><code class="language-java line-numbers"><script type="text/plain">// 默认按照升序排列
public static void sort(byte[] a);
public static void sort(char[] a);
public static void sort(short[] a);
public static void sort(int[] a);
public static void sort(long[] a);
public static void sort(float[] a);
public static void sort(double[] a);
public static void sort(Object[] a);
// 指定区间 [fromIndex, toIndex)
public static void sort(byte[] a, int fromIndex, int toIndex);
public static void sort(char[] a, int fromIndex, int toIndex);
public static void sort(short[] a, int fromIndex, int toIndex);
public static void sort(int[] a, int fromIndex, int toIndex);
public static void sort(long[] a, int fromIndex, int toIndex);
public static void sort(float[] a, int fromIndex, int toIndex);
public static void sort(double[] a, int fromIndex, int toIndex);
public static void sort(Object[] a, int fromIndex, int toIndex);
// 自定义排序，传入谓词 c
public static <T> void sort(T[] a, Comparator<? super T> c);
public static <T> void sort(T[] a, int fromIndex, int toIndex, Comparator<? super T> c);

// 二分查找
public static int binarySearch(byte[] a, byte key);
public static int binarySearch(byte[] a, int fromIndex, int toIndex, byte key);
public static int binarySearch(char[] a, char key);
public static int binarySearch(char[] a, int fromIndex, int toIndex, char key);
public static int binarySearch(short[] a, short key);
public static int binarySearch(short[] a, int fromIndex, int toIndex, short key);
public static int binarySearch(int[] a, int key);
public static int binarySearch(int[] a, int fromIndex, int toIndex, int key);
public static int binarySearch(long[] a, long key);
public static int binarySearch(long[] a, int fromIndex, int toIndex, long key);
public static int binarySearch(float[] a, float key);
public static int binarySearch(float[] a, int fromIndex, int toIndex, float key);
public static int binarySearch(double[] a, double key);
public static int binarySearch(double[] a, int fromIndex, int toIndex, double key);
public static int binarySearch(Object[] a, Object key);
public static int binarySearch(Object[] a, int fromIndex, int toIndex, Object key);
public static <T> int binarySearch(T[] a, T key, Comparator<? super T> c);
public static <T> int binarySearch(T[] a, int fromIndex, int toIndex, T key, Comparator<? super T> c);

// 判等
public static boolean equals(boolean[] a, boolean[] a2);
public static boolean equals(byte[] a, byte[] a2);
public static boolean equals(char[] a, char[] a2);
public static boolean equals(short[] a, short a2[]);
public static boolean equals(int[] a, int[] a2);
public static boolean equals(long[] a, long[] a2);
public static boolean equals(float[] a, float[] a2);
public static boolean equals(double[] a, double[] a2);
public static boolean equals(Object[] a, Object[] a2);

// 填充元素
public static void fill(boolean[] a, boolean val);
public static void fill(boolean[] a, int fromIndex, int toIndex, boolean val);
public static void fill(byte[] a, byte val);
public static void fill(byte[] a, int fromIndex, int toIndex, byte val);
public static void fill(char[] a, char val);
public static void fill(char[] a, int fromIndex, int toIndex, char val);
public static void fill(short[] a, short val);
public static void fill(short[] a, int fromIndex, int toIndex, short val);
public static void fill(int[] a, int val);
public static void fill(int[] a, int fromIndex, int toIndex, int val);
public static void fill(long[] a, long val);
public static void fill(long[] a, int fromIndex, int toIndex, long val);
public static void fill(float[] a, float val);
public static void fill(float[] a, int fromIndex, int toIndex, float val);
public static void fill(double[] a, double val);
public static void fill(double[] a, int fromIndex, int toIndex,double val);
public static void fill(Object[] a, Object val);
public static void fill(Object[] a, int fromIndex, int toIndex, Object val);

// 拷贝数组
public static boolean[] copyOf(boolean[] original, int newLength);
public static byte[] copyOf(byte[] original, int newLength);
public static char[] copyOf(char[] original, int newLength);
public static short[] copyOf(short[] original, int newLength);
public static int[] copyOf(int[] original, int newLength);
public static long[] copyOf(long[] original, int newLength);
public static float[] copyOf(float[] original, int newLength);
public static double[] copyOf(double[] original, int newLength);
public static <T> T[] copyOf(T[] original, int newLength);
public static <T, U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType);
// 提取指定区间的元素
public static boolean[] copyOfRange(boolean[] original, int from, int to);
public static byte[] copyOfRange(byte[] original, int from, int to);
public static char[] copyOfRange(char[] original, int from, int to);
public static short[] copyOfRange(short[] original, int from, int to);
public static int[] copyOfRange(int[] original, int from, int to);
public static long[] copyOfRange(long[] original, int from, int to);
public static float[] copyOfRange(float[] original, int from, int to);
public static double[] copyOfRange(double[] original, int from, int to);
public static <T> T[] copyOfRange(T[] original, int from, int to);
public static <T, U> T[] copyOfRange(U[] original, int from, int to, Class<? extends T[]> newType);

// Array -> List<E>集合
public static <T> List<T> asList(T... a);

// 计算 HashCode 值
public static int hashCode(boolean a[]);
public static int hashCode(byte a[]);
public static int hashCode(char a[]);
public static int hashCode(short a[]);
public static int hashCode(int a[]);
public static int hashCode(long a[]);
public static int hashCode(float a[]);
public static int hashCode(double a[]);
public static int hashCode(Object a[]);

// toString 转换为字符串形式
public static String toString(long[] a);
public static String toString(int[] a);
public static String toString(short[] a);
public static String toString(char[] a);
public static String toString(byte[] a);
public static String toString(boolean[] a);
public static String toString(float[] a);
public static String toString(double[] a);
public static String toString(Object[] a);
</script></code></pre>



**Comparable 和 Comparator 接口的区别**
Comparable 接口在 java.lang 包，Comparator 接口在 java.util 包；
Comparable 是**可比较**的意思，Comparator 是**比较器**的意思。

Comparable 和 Comparator 都是用来实现集合中元素的比较、排序的；
Comparable 使用集合内部定义的方法实现排序，Comparator 使用集合外部定义的方法实现排序。

所以，如想实现排序，有两种方式：
1) 在类的内部实现 java.lang.Comparable 接口，重写`public int compareTo(T o);`方法；
2) 在类的外部提供一个实现了 java.util.Comparator 接口的类，重写`int compare(T o1, T o2);`方法。

在类外部提供一个专门的比较器的方式更灵活一些，可以根据需求，自定义排序方式，可以随意的实现升序、降序排列；

例1，使用 Comparable + Collections.reverseOrder() 进行升序，降序排列：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Arrays;
import java.util.Comparator;
import java.util.Collections;

public class Main {
    public static void main(String[] args) {
        Student[] infos = {
            new Student("A", 12, 67),
            new Student("B", 12, 79),
            new Student("C", 11, 59),
            new Student("D", 12, 98),
            new Student("E", 13, 72),
        };

        System.out.println("original >>> " + Arrays.toString(infos));

        Arrays.sort(infos); // 升序
        System.out.println("ASC >>> " + Arrays.toString(infos));

        Arrays.sort(infos, Collections.reverseOrder()); // 降序
        System.out.println("DESC >>> " + Arrays.toString(infos));
    }
}

class Student implements Comparable<Student> {
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
        return String.format("@%s: %.1f", name, score);
    }

    @Override
    public int compareTo(Student s) {
        if (score > s.score) {
            return 1;
        } else if (score < s.score) {
            return -1;
        } else {
            return 0;
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:56:46] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [9:56:48]
$ java Main
original >>> [@A: 67.0, @B: 79.0, @C: 59.0, @D: 98.0, @E: 72.0]
ASC >>> [@C: 59.0, @A: 67.0, @E: 72.0, @B: 79.0, @D: 98.0]
DESC >>> [@D: 98.0, @B: 79.0, @E: 72.0, @A: 67.0, @C: 59.0]
</script></code></pre>



例2，使用 Comparator 进行升序，降序排列：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Arrays;
import java.util.Comparator;

public class Main {
    public static void main(String[] args) {
        Student[] infos = {
            new Student("A", 12, 67),
            new Student("B", 12, 79),
            new Student("C", 11, 59),
            new Student("D", 12, 98),
            new Student("E", 13, 72),
        };

        System.out.println("original >>> " + Arrays.toString(infos));

        Arrays.sort(infos, Student.Cmp.ASC); // 升序
        System.out.println("ASC >>> " + Arrays.toString(infos));

        Arrays.sort(infos, Student.Cmp.DESC); // 降序
        System.out.println("DESC >>> " + Arrays.toString(infos));
    }
}

class Student {
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
        return String.format("@%s: %.1f", name, score);
    }

    public static class Cmp implements Comparator<Student> {
        public static final Cmp ASC = new Cmp(true);
        public static final Cmp DESC = new Cmp(false);

        private boolean flag;
        public Cmp(boolean flag) {
            this.flag = flag;
        }

        @Override
        public int compare(Student s1, Student s2) {
            if (s1.score > s2.score) {
                return flag ? 1 : -1;
            } else if (s1.score < s2.score) {
                return flag ? -1 : 1;
            } else {
                return 0;
            }
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:13:40]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:13:56]
$ java Main
original >>> [@A: 67.0, @B: 79.0, @C: 59.0, @D: 98.0, @E: 72.0]
ASC >>> [@C: 59.0, @A: 67.0, @E: 72.0, @B: 79.0, @D: 98.0]
DESC >>> [@D: 98.0, @B: 79.0, @E: 72.0, @A: 67.0, @C: 59.0]
</script></code></pre>


## String 字符串
String 是 Java 中的字符串，String 实现了 CharSequence 字符序列接口；
除了 String 实现了 CharSequence 之外，StringBuffer 和 StringBuilder 也实现了 CharSequence 接口；

StringBuilder 和 StringBuffer 都是`可变`的字符序列，它们都继承于 AbstractStringBuilder，实现了 CharSequence 接口。
但是，StringBuilder 是非线程安全的，而 StringBuffer 是线程安全的。

String 内部封装了一个`char[]`数组，private final 属性，表示该数组对象不能指向其他数组对象，即指向不能变；
但是并不是说数组的内容不可变，因此，String 的不可变是规定出来的，我们依然可以通过反射修改数组的元素。

StringBuffer、StringBuilder 内部也是封装一个`char[]`数组，在实例化的时候可以指定数组的初始长度，默认为 16 字符；
当数组空间不足时，将重新申请一块新的内存，并将原来的数组的内容依次拷贝到新数组，因此，如果可以估算出需要的存储空间，最好在实例化时指明长度，避免不必要的内存拷贝开销！

CharSequence、String、StringBuffer、StringBuilder 之间的关系图如下：
![CharSequence、String、StringBuffer、StringBuilder](/images/java-string.jpg)

**CharSequence 接口**
<pre><code class="language-java line-numbers"><script type="text/plain">public interface CharSequence {
    int length(); // 返回 char[] 数组的长度，即"code unit"的数量

    /**
     * 返回索引 index 上的字符, 即 [] 下标运算符.
     * @param index 索引值
     * @return 返回 index 上的字符 char
     * @throws IndexOutOfBoundsException 越界抛出异常
     */
    char charAt(int index);

    /**
     * 提取子串, [start, end) 左闭右开区间.
     * @param   start   起始索引（包含）
     * @param   end     结束索引（排除）
     * @return  返回指定的子串
     * @throws  IndexOutOfBoundsException
     */
    CharSequence subSequence(int start, int end);

    public String toString();
}
</script></code></pre>



**String 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[]; // char[] 字符数组
    private int hash; // Default to 0

    public String();
    public String(String original);

    public String(char value[]);
    public String(char value[], int offset, int count);

    public String(int[] codePoints, int offset, int count);

    public String(byte bytes[]); // 平台默认编码
    public String(byte bytes[], int offset, int length);
    public String(byte bytes[], String charsetName) throws UnsupportedEncodingException;
    public String(byte bytes[], int offset, int length, String charsetName) throws UnsupportedEncodingException;
    public String(byte bytes[], Charset charset);
    public String(byte bytes[], int offset, int length, Charset charset);

    public String(StringBuffer buffer);
    public String(StringBuilder builder);

    public int length(); // 统计 char 的数目
    public int codePointCount(int beginIndex, int endIndex); // 统计 code point 的数目
    public boolean isEmpty();

    public char charAt(int index);
    public int codePointAt(int index);

    public byte[] getBytes();
    public byte[] getBytes(Charset charset);
    public byte[] getBytes(String charsetName) throws UnsupportedEncodingException;
    public char[] toCharArray(); // 内部创建char[]，并将其返回
    public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin); // 需要调用者new char[]

    public String toString();
    public int hashCode();
    public boolean equals(Object anObject);
    public boolean contentEquals(StringBuffer sb); // 比较字符串内容
    public boolean contentEquals(CharSequence cs);
    public boolean equalsIgnoreCase(String anotherString); // 忽略大小写

    public int compareTo(String anotherString);
    public int compareToIgnoreCase(String str); // 忽略大小写compare比较

    /**
     * 测试子串是否相匹配.
     * @param toffset "this.offset"
     * @param other 要比较的字符串
     * @param ooffset "other.offset"
     * @param len 子串的长度
     * @return true: 匹配; false: 不匹配
     */
    public boolean regionMatches(int toffset, String other, int ooffset, int len);
    // 忽略大小写
    public boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len);

    /**
     * 测试当前字符串的前缀.
     * @param prefix 前缀
     * @param toffset "this.offset"
     * @return true: 匹配; false: 不匹配
     */
    public boolean startsWith(String prefix, int toffset);
    public boolean startsWith(String prefix); // toffset = 0;
    public boolean endsWith(String suffix); // 后缀, 同上

    public int indexOf(int ch); // 查找指定字符的首次匹配index
    public int indexOf(int ch, int fromIndex); // 指定起始位置
    public int lastIndexOf(int ch); // 查找指定字符的最后一次匹配的index
    public int lastIndexOf(int ch, int fromIndex);

    public int indexOf(String str);
    public int indexOf(String str, int fromIndex);
    public int lastIndexOf(String str);
    public int lastIndexOf(String str, int fromIndex);

    public String substring(int beginIndex); // 提取子串
    public String substring(int beginIndex, int endIndex);
    public CharSequence subSequence(int beginIndex, int endIndex);

    public String concat(String str); // 拼接字符串

    public boolean matches(String regex); // Pattern.matches(regex, this);
    public boolean contains(CharSequence s); // 是否包含指定的子串

    public String replace(char oldChar, char newChar); // 字符替换
    public String replace(CharSequence target, CharSequence replacement); // 非regex替换
    public String replaceFirst(String regex, String replacement); // regex替换，首次
    public String replaceAll(String regex, String replacement); // regex替换，全部

    public String[] split(String regex, int limit); // regex分割
    public String[] split(String regex); // limit = 0;

    public String toLowerCase(Locale locale); // 转换为小写
    public String toLowerCase();
    public String toUpperCase(Locale locale);
    public String toUpperCase();
    public String trim(); // 删除任何前导和尾随空格

    public static String format(String format, Object... args); // 格式化字符串
    public static String format(Locale l, String format, Object... args);

    public static String valueOf(Object obj); // obj.toString()
    public static String valueOf(char data[]);
    public static String valueOf(char data[], int offset, int count);
    public static String copyValueOf(char data[]);
    public static String copyValueOf(char data[], int offset, int count);
    public static String valueOf(boolean b);
    public static String valueOf(char c);
    public static String valueOf(int i);
    public static String valueOf(long l);
    public static String valueOf(float f);
    public static String valueOf(double d);

    public native String intern(); // 返回字符串常量池中相同字符串的引用
}
</script></code></pre>



**AbstractStringBuilder 抽象类**
<pre><code class="language-java line-numbers"><script type="text/plain">public int length(); // 元素的数目
public int capacity(); // 内部数组的长度（容量）
public int codePointCount(int beginIndex, int endIndex);

public void ensureCapacity(int minimumCapacity); // 调整容量，仅当 minimumCapacity > this.capacity() 有效
public void trimToSize(); // 试图回收没有使用的容量（内存）
public void setLength(int newLength); // 调整length，按照需要进行填充'\0'或者截断

public char charAt(int index);
public int codePointAt(int index);

public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin);
public void setCharAt(int index, char ch);

public AbstractStringBuilder append(Object obj);
public AbstractStringBuilder append(String str);
public AbstractStringBuilder append(StringBuffer sb);
public AbstractStringBuilder append(CharSequence s);
public AbstractStringBuilder append(CharSequence s, int start, int end);
public AbstractStringBuilder append(char[] str);
public AbstractStringBuilder append(char str[], int offset, int len);
public AbstractStringBuilder append(char c);
public AbstractStringBuilder append(boolean b);
public AbstractStringBuilder append(int i);
public AbstractStringBuilder append(long l);
public AbstractStringBuilder append(float f);
public AbstractStringBuilder append(double d);
public AbstractStringBuilder appendCodePoint(int codePoint);

public AbstractStringBuilder delete(int start, int end);
public AbstractStringBuilder deleteCharAt(int index)

public AbstractStringBuilder replace(int start, int end, String str);

public String substring(int start);
public CharSequence subSequence(int start, int end);
public String substring(int start, int end);

public AbstractStringBuilder insert(int index, char[] str, int offset, int len);
public AbstractStringBuilder insert(int offset, Object obj);
public AbstractStringBuilder insert(int offset, String str);
public AbstractStringBuilder insert(int offset, char[] str);
public AbstractStringBuilder insert(int dstOffset, CharSequence s);
public AbstractStringBuilder insert(int dstOffset, CharSequence s, int start, int end);
public AbstractStringBuilder insert(int offset, boolean b);
public AbstractStringBuilder insert(int offset, char c);
public AbstractStringBuilder insert(int offset, int i);
public AbstractStringBuilder insert(int offset, long l);
public AbstractStringBuilder insert(int offset, float f);
public AbstractStringBuilder insert(int offset, double d);

public int indexOf(String str);
public int indexOf(String str, int fromIndex);
public int lastIndexOf(String str);
public int lastIndexOf(String str, int fromIndex);

public AbstractStringBuilder reverse(); // 翻转

public abstract String toString();
</script></code></pre>



**StringBuffer 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public StringBuffer(); // 16
public StringBuffer(int capacity);
public StringBuffer(String str); // 尾部预留 16
public StringBuffer(CharSequence seq); // 尾部预留 16

public synchronized int length();
public synchronized int capacity();
public synchronized int codePointCount(int beginIndex, int endIndex);
public synchronized void ensureCapacity(int minimumCapacity);
public synchronized void trimToSize();
public synchronized void setLength(int newLength);

public synchronized char charAt(int index);
public synchronized int codePointAt(int index);

public synchronized void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin);
public synchronized void setCharAt(int index, char ch);

public synchronized StringBuffer append(Object obj);
public synchronized StringBuffer append(String str);
public synchronized StringBuffer append(StringBuffer sb);
public synchronized StringBuffer append(CharSequence s);
public synchronized StringBuffer append(CharSequence s, int start, int end);
public synchronized StringBuffer append(char[] str);
public synchronized StringBuffer append(char[] str, int offset, int len);
public synchronized StringBuffer append(boolean b);
public synchronized StringBuffer append(char c);
public synchronized StringBuffer append(int i);
public synchronized StringBuffer append(long lng);
public synchronized StringBuffer append(float f);
public synchronized StringBuffer append(double d);
public synchronized StringBuffer appendCodePoint(int codePoint);

public synchronized StringBuffer delete(int start, int end);
public synchronized StringBuffer deleteCharAt(int index);

public synchronized StringBuffer replace(int start, int end, String str);

public synchronized String substring(int start);
public synchronized String substring(int start, int end);
public synchronized CharSequence subSequence(int start, int end);

public StringBuffer insert(int offset, boolean b);
public StringBuffer insert(int offset, int i);
public StringBuffer insert(int offset, long l);
public StringBuffer insert(int offset, float f);
public StringBuffer insert(int offset, double d);
public synchronized StringBuffer insert(int offset, Object obj);
public synchronized StringBuffer insert(int offset, String str);
public synchronized StringBuffer insert(int offset, char c);
public synchronized StringBuffer insert(int offset, char[] str);
public synchronized StringBuffer insert(int index, char[] str, int offset, int len);
public synchronized StringBuffer insert(int dstOffset, CharSequence s);
public synchronized StringBuffer insert(int dstOffset, CharSequence s, int start, int end);

public int indexOf(String str);
public synchronized int indexOf(String str, int fromIndex);
public int lastIndexOf(String str);
public synchronized int lastIndexOf(String str, int fromIndex);

public synchronized StringBuffer reverse();

public synchronized String toString();
</script></code></pre>



**StringBuilder 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public StringBuilder(); // 16
public StringBuilder(int capacity);
public StringBuilder(String str);
public StringBuilder(CharSequence seq);

public StringBuilder append(Object obj);
public StringBuilder append(String str);
public StringBuilder append(StringBuffer sb);
public StringBuilder append(CharSequence s);
public StringBuilder append(CharSequence s, int start, int end);
public StringBuilder append(char[] str);
public StringBuilder append(char[] str, int offset, int len);
public StringBuilder append(boolean b);
public StringBuilder append(char c);
public StringBuilder append(int i);
public StringBuilder append(long lng);
public StringBuilder append(float f);
public StringBuilder append(double d);
public StringBuilder appendCodePoint(int codePoint);

public StringBuilder delete(int start, int end);
public StringBuilder deleteCharAt(int index);

public StringBuilder replace(int start, int end, String str);

public StringBuilder insert(int index, char[] str, int offset, int len);
public StringBuilder insert(int offset, Object obj);
public StringBuilder insert(int offset, String str);
public StringBuilder insert(int offset, char[] str);
public StringBuilder insert(int dstOffset, CharSequence s);
public StringBuilder insert(int dstOffset, CharSequence s, int start, int end);
public StringBuilder insert(int offset, boolean b);
public StringBuilder insert(int offset, char c);
public StringBuilder insert(int offset, int i);
public StringBuilder insert(int offset, long l);
public StringBuilder insert(int offset, float f);
public StringBuilder insert(int offset, double d);

public int indexOf(String str);
public int indexOf(String str, int fromIndex);
public int lastIndexOf(String str);
public int lastIndexOf(String str, int fromIndex);

public StringBuilder reverse();

public String toString();
</script></code></pre>


## 包装类
基本类型（值类型）：`byte`、`short`、`int`、`long`、`float`、`double`、`boolean`、`char`
包装类（引用类型）：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Boolean`、`Character`

**值类型** -> **引用类型**，称为`装箱`；
**引用类型** -> **值类型**，称为`拆箱`。

在 jdk1.5 之前，装箱、拆箱需要我们手动干预，称为**手动装箱**、**手动拆箱**；
在 jdk1.5 之后，装箱、拆箱可以由编译器自动完成，称为**自动装箱**、**自动拆箱**。

除了 Boolean、Character 类型直接继承 Object 类，Byte、Short、Integer、Long、Float、Double 都是继承自 Number 类。

包装类对象一经创建，所封装的基本类型的值不会再改变；这一点和 String 是一样的。

**常量池**
常量池有两种：
1) `class文件常量池`：或称为"静态常量池"，用于存放编译器生成的各种`字面量`、`符号引用`，在类加载之后会放到方法区的**运行时常量池**中；
2) `运行时常量池`：或称为"动态常量池"，与静态常量池不同的是，它具有**动态性**，即可以在运行期间动态的将新的常量放入池中。

常量池的运用可以有效地减少相同常量的多次存储，减少不必要的存储空间浪费。

而动态常量池在开发中运用的最多的就是 String 的 intern() 成员方法；
并且基本类型包装类也存在运行时常量池，它们的作用和 String 是相似的。

八大基本类型的包装类中，除了`浮点型`的包装类（**Float**、**Double**）外，其他所有的包装类都存在常量池机制。

当一个 String 实例调用 intern() 方法时，首先会去查找 String 运行时常量池中是否有相同的字符串常量；
如果有，则返回常量池中该字符串的引用；如果没有，将当前对象的加入到常量池中，并返回其在常量池中的引用。

**String 的两种创建方式**
1) `String s = "www.zfl9.com";`
第一步，将字面量"www.zfl9.com"存放在 Class 文件的常量池中；
第二步，执行`String s`，新建一个 String 引用变量 s（String 类型的指针）；
第三步，将字面量"www.zfl9.com"的地址赋给引用变量 s。

在这种方式中，只创建了一个对象，即"www.zfl9.com"常量；

2) `String s = new String("www.zfl9.com");`
第一步，将字面量"www.zfl9.com"存放在 Class 文件的常量池中；
第二步，执行`new String()`，在堆中创建一个 String 对象，并使用常量"www.zfl9.com"进行初始化（拷贝构造）；
第三步，执行`String s`，新建一个 String 引用变量 s（String 类型的指针）；
第四步，将刚刚在堆中创建的匿名对象的指针赋给引用变量 s。

在这种方式中，创建了两个对象，一个在常量池中，一个在堆中；

因此，不建议使用第二种形式，会造成内存空间的浪费！

String.intern() 的例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        String s1 = "www.zfl9.com";
        String s2 = "www.zfl9.com";
        String s3 = new String("www.zfl9.com");
        String s4 = new String(s3);

        out.printf("s1 == s2 -> %b\n", s1 == s2); // true
        out.printf("s1 == s3 -> %b\n", s1 == s3); // false
        out.printf("s3 == s4 -> %b\n", s3 == s4); // false

        out.printf("s1.equals(s3) -> %b\n", s1.equals(s3)); // true
        out.printf("s3.equals(s4) -> %b\n", s3.equals(s4)); // true

        out.printf("s1 == s3.intern() == s4.intern() -> %b\n",
                   s1 == s3.intern() && s1 == s4.intern()); // true
        out.printf("s1.intern() == s3.intern() -> %b\n", s1.intern() == s3.intern()); // true
        out.printf("s3.intern() == s4.intern() -> %b\n", s3.intern() == s4.intern()); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:12:43]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:12:56]
$ java Main
s1 == s2 -> true
s1 == s3 -> false
s3 == s4 -> false
s1.equals(s3) -> true
s3.equals(s4) -> true
s1 == s3.intern() == s4.intern() -> true
s1.intern() == s3.intern() -> true
s3.intern() == s4.intern() -> true
</script></code></pre>



好吧，有些扯远了，我们回到包装类中来，包装类有一些共同方法，以 Integer 为例：
**手动装箱**
`public Integer(int value)`
`public Integer(String s) throws NumberFormatException`：解析字符串中的 int。
**自动装箱**
`public static Integer valueOf(int i)`：基本类型的值在区间`[-128, 127]`的对象将入池。
`public static Integer valueOf(String s) throws NumberFormatException`：同上。
`public static Integer valueOf(String s, int radix) throws NumberFormatException`：同上。

`public int intValue()`：返回所包装的基本类型的值；

`public static int parseInt(String s) throws NumberFormatException`：解析字符串中的数字；
`public static int parseInt(String s, int radix) throws NumberFormatException`：同上。

手动装箱因为每次都是使用`new`创建，所以每次创建的对象都是不同的，它们生死于堆上；
而自动装箱则有点不同，它不使用`new`创建，而是使用其静态方法`valueOf()`，`valueOf()`内部维护了一个常量池；

1) 初始时，该 cache 池为空，没有任何包装类对象；
2) 当调用`valueOf(10)`方法自动装箱时，发现 cache 池中没有值等于 10 的对象，于是新建一个对象并丢入 cache 池中；
3) 当再次调用`valueOf(10)`方法自动装箱时，发现 cache 池中已有值相同的对象，于是不再创建新对象，而是将已有对象返回；
4) 但是 cache 池并不是无限大的，是有一定范围的，在 Integer 中，它被限制为只缓存区间`[-128, 127]`的对象，即一个字节表示的整数；
5) 如果传入 valueOf() 的参数不在该范围中，那么等同于手动装箱，即每次都会 new 一个新的对象出来；

除了 Integer 有所谓的 cache 池，Boolean、Byte、Short、Long、Character 也有 cache，如下：
- Short、Long 和 Integer 一样，区间都是 [-128, 127]；
- Boolean、Byte 因为它们占用的内存长度都在 1 字节之内，因此全部取值范围都被缓存；
- 而 Character 相当于无符号的 Short 整型，因此在区间 [0, 127] 的对象也将被缓存；
- 但是 Float、Double 浮点型的对象并不会被缓存，不管它们的取值范围是多少；



为什么浮点数包装类不会被丢入池中，无论它们的大小是多少？
准确原因我也不是很明确，但是我猜测是因为"整数和小数在内存中的表示是不同的"；
对于整型变量，假设它们都是有符号的，如果其值的区间在`[-128, 127]`内，那么就可以将其压缩为一个字节（有符号）来存储。

还有一点要注意：
1) 当`==`运算符的两个操作数都是`引用类型`（包装类）时，比较引用的值，不触发自动拆箱；
2) 如果其中有一个操作数是`算数表达式/数值`则触发`自动拆箱`，这时比较的是`基本类型的值`。

例子一：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        Integer a1 = 40;
        Integer a2 = 40;
        Integer a3 = 0;
        Integer b1 = new Integer(40);
        Integer b2 = new Integer(40);
        Integer b3 = new Integer(0);

        out.printf("a1 == a2 -> %b\n", a1 == a2); // true
        out.printf("a1 == a2 + a3 -> %b\n", a1 == a2 + a3); // true
        out.printf("b1 == b2 -> %b\n", b1 == b2); // false
        out.printf("b1 == b2 + b3 -> %b\n", b1 == b2 + b3); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:58:27]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:58:42]
$ java Main
a1 == a2 -> true
a1 == a2 + a3 -> true
b1 == b2 -> false
b1 == b2 + b3 -> true
</script></code></pre>



如果你理解了前面的内容，那么这个例子就很容易理解了：
1) `a1 == a2`：a1、a2 自动装箱，值在区间 [-128, 127]，因此它们都引用同一个对象，而`==`两边的操作数都是引用类型，比较他们的引用的值，因为是同一个对象，所以返回 true；
2) `a1 == a2 + a3`：a1、a2、a3 都是自动装箱，a1 和 a2 都引用自池中的同一对象，`==`的右操作数是一个表达式，触发 a2、a3 的自动拆箱，变为`40 + 0`，即右操作数为 40，因为有一个操作数是值类型，所以触发 a1 的自动拆箱，最终比较的是`40 == 40`，返回 true；
3) `b1 == b2`：因为 b1、b2 都是手动装箱，所以他们引用的是不同的对象，因此返回 false；
4) `b1 == b2 + b3`：右操作数是一个表达式，触发自动拆箱，结果为 40，而 b1 也被触发自动拆箱，结果为 40，因此返回 true。


再来一个例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        Integer a1 = 1;
        Integer a2 = 2;
        Integer a3 = 3;
        Integer a4 = 3;

        Integer a5 = 200;
        Integer a6 = 200;

        Long b1 = 3L;
        Long b2 = 2L;

        out.printf("a3 == a4 -> %b\n", a3 == a4); // true
        out.printf("a5 == a6 -> %b\n", a5 == a6); // false
        out.printf("a3 == a1 + a2 -> %b\n", a3 == a1 + a2); // true
        out.printf("a3.equals(a1 + a2) -> %b\n", a3.equals(a1 + a2)); // true
        out.printf("b1 == a1 + a2 -> %b\n", b1 == a1 + a2); // true
        out.printf("b1.equals(a1 + a2) -> %b\n", b1.equals(a1 + a2)); // false
        out.printf("b1.equals(a1 + b2) -> %b\n", b1.equals(a1 + b2)); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:19:29]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:19:43]
$ java Main
a3 == a4 -> true
a5 == a6 -> false
a3 == a1 + a2 -> true
a3.equals(a1 + a2) -> true
b1 == a1 + a2 -> true
b1.equals(a1 + a2) -> false
b1.equals(a1 + b2) -> true
</script></code></pre>



1) `a3 == a4`：自动装箱，比较的是引用，因为在区间 [-128, 127]，true；
2) `a5 == a6`：自动装箱，比较的是引用，因为不在区间 [-128, 127]，false；
3) `a3 == a1 + a2`：触发自动拆箱，比较的是数值，true；
4) `a3.equals(a1 + a2)`：计算`a1 + a2`时触发自动拆箱，然后再次自动装箱，因此返回 true；
5) `b1 == a1 + a2`：计算`a1 + a2`时触发自动拆箱，结果为 int 类型的值 3，b1 也因此自动拆箱，是 long 类型的值 3；然后 int -> long 自动类型转换，因此返回 true；
6) `b1.equals(a1 + a2)`：计算`a1 + a2`时触发自动拆箱，结果为 int 类型的值 3，然后再次装箱为 Integer 引用类型，因为比较的两个对象的类型不同，所以返回 false；
7) `b1.equals(a1 + b2)`：计算`a1 + b2`时触发自动拆箱，并且发生自动类型转换 int -> long，然后装箱为 Long 引用类型，因此返回 true。


**Integer 相关方法**
<pre><code class="language-java line-numbers"><script type="text/plain">// 静态属性
public static final int MIN_VALUE = 0x80000000;
public static final int MAX_VALUE = 0x7fffffff;
public static final int SIZE = 32;
public static final int BYTES = SIZE / Byte.SIZE;

public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");

public static String toString(int i); // 默认为 10 进制
public static String toString(int i, int radix); // radix 为进制
public static String toBinaryString(int i); // 2 进制
public static String toOctalString(int i); // 8 进制
public static String toHexString(int i); // 16 进制

// 从字符串中解析 int 数值，radix 可指定进制，NumberFormatException 为运行时异常
public static int parseInt(String s, int radix) throws NumberFormatException;
public static int parseInt(String s) throws NumberFormatException; // radix = 10

// 内部调用 Integer.valueOf(parseInt(s,radix));
public static Integer valueOf(String s, int radix) throws NumberFormatException;
// 内部调用 Integer.valueOf(parseInt(s, 10));
public static Integer valueOf(String s) throws NumberFormatException;

public static Integer valueOf(int i); // 自动装箱
public Integer(int value); // 手动装箱
public Integer(String s) throws NumberFormatException;

public byte byteValue(); // 内部使用强制类型转换
public short shortValue();
public int intValue(); // 自动拆箱
public long longValue();
public float floatValue();
public double doubleValue();

public String toString();

public int hashCode();
public static int hashCode(int value); // JDK1.8
public boolean equals(Object obj);

public int compareTo(Integer anotherInteger);
public static int compare(int x, int y); // JDK1.7

public static int sum(int a, int b);
public static int max(int a, int b);
public static int min(int a, int b);
</script></code></pre>



**Character 相关方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public static final char MIN_VALUE = '\u0000';
public static final char MAX_VALUE = '\uFFFF';
public static final Class<Character> TYPE = (Class<Character>) Class.getPrimitiveClass("char");

public Character(char value);
public static Character valueOf(char c);

public char charValue();

public int hashCode();
public static int hashCode(char value);
public boolean equals(Object obj);

public String toString();
public static String toString(char c);

public static boolean isLowerCase(char ch);
public static boolean isUpperCase(char ch);

public static char toLowerCase(char ch);
public static char toUpperCase(char ch);
</script></code></pre>
