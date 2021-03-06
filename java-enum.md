---
title: Java Enum枚举
date: 2017-09-18 10:09:00
categories:
- java
tags:
- java
keywords: Java Enum枚举
---

> 
Java Enum枚举，枚举的定义及使用、枚举与类的区别、枚举的本质、java.lang.Enum抽象类

<!-- more -->

## Enum枚举
在 jdk1.5 之前，如果需要定义一个表示星期几的常量，一般通过`public static final ...`来定义：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        week(Week.Fri); // Friday
        week(5);        // Friday 显然不是类型安全的
    }

    public static void week(int day) {
        switch (day) {
            case Week.Sun : out.println("Sunday"); break;
            case Week.Mon : out.println("Monday"); break;
            case Week.Tues : out.println("Tuesday"); break;
            case Week.Wed : out.println("Wednesday"); break;
            case Week.Thur : out.println("Thursday"); break;
            case Week.Fri : out.println("Friday"); break;
            case Week.Sat : out.println("Saturday"); break;
            default : out.println("Error");
        }
    }
}

class Week {
    public static final int Sun = 0;
    public static final int Mon = 1;
    public static final int Tues = 2;
    public static final int Wed = 3;
    public static final int Thur = 4;
    public static final int Fri = 5;
    public static final int Sat = 6;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:32:22]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:32:40]
$ java Main
Friday
Friday
</script></code></pre>



使用 enum 关键字进行枚举的定义：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        week(Week.Fri); // Friday
        //week(5);      // 编译错误
    }

    public static void week(Week day) {
        switch (day) {
            case Sun : out.println("Sunday"); break;
            case Mon : out.println("Monday"); break;
            case Tues : out.println("Tuesday"); break;
            case Wed : out.println("Wednesday"); break;
            case Thur : out.println("Thursday"); break;
            case Fri : out.println("Friday"); break;
            case Sat : out.println("Saturday"); break;
            default : out.println("Error");
        }
    }
}

enum Week {
    Sun, Mon, Tues, Wed, Thur, Fri, Sat;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:41:35]
$ ls
Main.java

# root @ arch in ~/work on git:master x [10:41:47]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:41:49]
$ ls
'Main$1.class'   Main.class   Main.java   Week.class

# root @ arch in ~/work on git:master x [10:41:52]
$ java Main
Friday

# root @ arch in ~/work on git:master x [10:42:07]
$ javap -c -p Week
Compiled from "Main.java"
final class Week extends java.lang.Enum<Week> {
  public static final Week Sun;

  public static final Week Mon;

  public static final Week Tues;

  public static final Week Wed;

  public static final Week Thur;

  public static final Week Fri;

  public static final Week Sat;

  private static final Week[] $VALUES;

  public static Week[] values();
    Code:
       0: getstatic     #1                  // Field $VALUES:[LWeek;
       3: invokevirtual #2                  // Method "[LWeek;".clone:()Ljava/lang/Object;
       6: checkcast     #3                  // class "[LWeek;"
       9: areturn

  public static Week valueOf(java.lang.String);
    Code:
       0: ldc           #4                  // class Week
       2: aload_0
       3: invokestatic  #5                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
       6: checkcast     #4                  // class Week
       9: areturn

  private Week();
    Code:
       0: aload_0
       1: aload_1
       2: iload_2
       3: invokespecial #6                  // Method java/lang/Enum."<init>":(Ljava/lang/String;I)V
       6: return

  static {};
    Code:
       0: new           #4                  // class Week
       3: dup
       4: ldc           #7                  // String Sun
       6: iconst_0
       7: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      10: putstatic     #9                  // Field Sun:LWeek;
      13: new           #4                  // class Week
      16: dup
      17: ldc           #10                 // String Mon
      19: iconst_1
      20: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      23: putstatic     #11                 // Field Mon:LWeek;
      26: new           #4                  // class Week
      29: dup
      30: ldc           #12                 // String Tues
      32: iconst_2
      33: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      36: putstatic     #13                 // Field Tues:LWeek;
      39: new           #4                  // class Week
      42: dup
      43: ldc           #14                 // String Wed
      45: iconst_3
      46: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      49: putstatic     #15                 // Field Wed:LWeek;
      52: new           #4                  // class Week
      55: dup
      56: ldc           #16                 // String Thur
      58: iconst_4
      59: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      62: putstatic     #17                 // Field Thur:LWeek;
      65: new           #4                  // class Week
      68: dup
      69: ldc           #18                 // String Fri
      71: iconst_5
      72: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      75: putstatic     #19                 // Field Fri:LWeek;
      78: new           #4                  // class Week
      81: dup
      82: ldc           #20                 // String Sat
      84: bipush        6
      86: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      89: putstatic     #21                 // Field Sat:LWeek;
      92: bipush        7
      94: anewarray     #4                  // class Week
      97: dup
      98: iconst_0
      99: getstatic     #9                  // Field Sun:LWeek;
     102: aastore
     103: dup
     104: iconst_1
     105: getstatic     #11                 // Field Mon:LWeek;
     108: aastore
     109: dup
     110: iconst_2
     111: getstatic     #13                 // Field Tues:LWeek;
     114: aastore
     115: dup
     116: iconst_3
     117: getstatic     #15                 // Field Wed:LWeek;
     120: aastore
     121: dup
     122: iconst_4
     123: getstatic     #17                 // Field Thur:LWeek;
     126: aastore
     127: dup
     128: iconst_5
     129: getstatic     #19                 // Field Fri:LWeek;
     132: aastore
     133: dup
     134: bipush        6
     136: getstatic     #21                 // Field Sat:LWeek;
     139: aastore
     140: putstatic     #1                  // Field $VALUES:[LWeek;
     143: return
}

# root @ arch in ~/work on git:master x [10:42:17]
$ javap -c -p Main\$1
Compiled from "Main.java"
class Main$1 {
  static final int[] $SwitchMap$Week;

  static {};
    Code:
       0: invokestatic  #1                  // Method Week.values:()[LWeek;
       3: arraylength
       4: newarray       int
       6: putstatic     #2                  // Field $SwitchMap$Week:[I
       9: getstatic     #2                  // Field $SwitchMap$Week:[I
      12: getstatic     #3                  // Field Week.Sun:LWeek;
      15: invokevirtual #4                  // Method Week.ordinal:()I
      18: iconst_1
      19: iastore
      20: goto          24
      23: astore_0
      24: getstatic     #2                  // Field $SwitchMap$Week:[I
      27: getstatic     #6                  // Field Week.Mon:LWeek;
      30: invokevirtual #4                  // Method Week.ordinal:()I
      33: iconst_2
      34: iastore
      35: goto          39
      38: astore_0
      39: getstatic     #2                  // Field $SwitchMap$Week:[I
      42: getstatic     #7                  // Field Week.Tues:LWeek;
      45: invokevirtual #4                  // Method Week.ordinal:()I
      48: iconst_3
      49: iastore
      50: goto          54
      53: astore_0
      54: getstatic     #2                  // Field $SwitchMap$Week:[I
      57: getstatic     #8                  // Field Week.Wed:LWeek;
      60: invokevirtual #4                  // Method Week.ordinal:()I
      63: iconst_4
      64: iastore
      65: goto          69
      68: astore_0
      69: getstatic     #2                  // Field $SwitchMap$Week:[I
      72: getstatic     #9                  // Field Week.Thur:LWeek;
      75: invokevirtual #4                  // Method Week.ordinal:()I
      78: iconst_5
      79: iastore
      80: goto          84
      83: astore_0
      84: getstatic     #2                  // Field $SwitchMap$Week:[I
      87: getstatic     #10                 // Field Week.Fri:LWeek;
      90: invokevirtual #4                  // Method Week.ordinal:()I
      93: bipush        6
      95: iastore
      96: goto          100
      99: astore_0
     100: getstatic     #2                  // Field $SwitchMap$Week:[I
     103: getstatic     #11                 // Field Week.Sat:LWeek;
     106: invokevirtual #4                  // Method Week.ordinal:()I
     109: bipush        7
     111: iastore
     112: goto          116
     115: astore_0
     116: return
    Exception table:
       from    to  target type
           9    20    23   Class java/lang/NoSuchFieldError
          24    35    38   Class java/lang/NoSuchFieldError
          39    50    53   Class java/lang/NoSuchFieldError
          54    65    68   Class java/lang/NoSuchFieldError
          69    80    83   Class java/lang/NoSuchFieldError
          84    96    99   Class java/lang/NoSuchFieldError
         100   112   115   Class java/lang/NoSuchFieldError
}
</script></code></pre>



1) Week 类（枚举）的反编译分析：
1. `final class Week extends java.lang.Enum<Week>`：枚举 Week 实际上是一个继承自 Enum 的 final 类；
因此，枚举类型不能再继承其他基类，当然也不能被继承。但是枚举类型可以实现（一个或多个）接口；
注意，我们自己不能显式定义继承自 Enum 抽象基类的派生类，会产生一个编译错误，Enum 类只能由编译器调用；
2. 编译器自动为 Week 枚举定义了相应的`public static final Week`静态常量，也就是说，类似`Week.Fri`其实是一个引用类型！
3. `public static Week[] values()`返回 Week 枚举的所有取值的数组，其调用`Object.clone()`方法拷贝常量`$VALUES`；
4. `public static Week valueOf(java.lang.String)`返回字符串对应的枚举常量，内部通过调用`Enum.valueOf(Class<T> enumType, String name)`方法来实现；
5. `private Week(String name, int ordinal)`私有构造函数，内部通过`super(name, ordinal)`实现，name 为 枚举常量的字符串形式，ordinal 为序号（默认从 0 开始编号）；
Enum 类的唯一构造函数`protected Enum(String name, int ordinal)`；
如果我们需要显式定义枚举类型的构造函数，那么其访问性必须为`private`或者`[default]`；
枚举类型的构造函数只能由编译器来调用，否则产生编译错误；并且构造函数内部不能显式调用`super(name, ordinal)`基类的构造函数！
6. 最后是一个 static 静态初始块，例如对于 Week.Fri，`Fri = new Week("Fri", 5)`；


2) Main$1 匿名类的反编译分析：
我们知道，`switch...case`只能接受`整型字面量`、`字符/字符串字面量`、以及对应的 final 常量；
而 Week.Fri 是一个引用变量，不属于上面的任意一种，所以需要进行相应的修改才能编译通过；
于是编译器将对应的枚举常量替换为它们的 ordinal 序号（int 整型），这时候就没有任何问题了；

## Enum抽象类
通过上面的简单分析，我们得知，使用 enum 定义的枚举类型其实就是一个实实在在的类，基本可以把它当作类来使用（不过有一些特别的限制）；

它们都共同继承自`java.lang.Enum`抽象基类，Enum 类的常用方法：
<pre><code class="language-java line-numbers"><script type="text/plain">public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {
    // name 名称
    private final String name;
    public final String name() {
        return name;
    }

    // ordinal 序号
    private final int ordinal;
    public final int ordinal() {
        return ordinal;
    }

    // 构造函数
    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }

    // 转换为字符串形式
    public String toString() {
        return name;
    }

    // 比较两个对象
    public final boolean equals(Object other) {
        return this==other;
    }

    // 获取对象的 Hash 值
    public final int hashCode() {
        return super.hashCode();
    }

    // 该方法不被支持，将抛出 CloneNotSupportedException 异常
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }

    // 比较枚举常量的大小，返回它们的 ordinal 差
    public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }

    // 获取枚举类型的 Class 对象
    @SuppressWarnings("unchecked")
    public final Class<E> getDeclaringClass() {
        Class<?> clazz = getClass();
        Class<?> zuper = clazz.getSuperclass();
        return (zuper == Enum.class) ? (Class<E>)clazz : (Class<E>)zuper;
    }

    // 获取字符串形式对应的枚举常量对象
    public static <T extends Enum<T>> T valueOf(Class<T> enumType, String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }

    // 该方法不被支持，因为是 final 属性的方法，无法被重写
    protected final void finalize() { }
}
</script></code></pre>



需要注意的是，java.lang.Enum 类不能被显式的继承，只能由编译器使用，下面的例子演示了这种情况；
<pre><code class="language-java line-numbers"><script type="text/plain">enum Week {
    Sun, Mon, Tues, Wed, Thur, Fri, Sat;
}

class _Week extends Enum<_Week> {
    private static final long serialVersionUID = 1234567890L;
    public static final int Sun = 0;
    public static final int Mon = 1;
    public static final int Tues = 2;
    public static final int Wed = 3;
    public static final int Thur = 4;
    public static final int Fri = 5;
    public static final int Sat = 6;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:45:42]
$ javac Main.java
Main.java:5: error: classes cannot directly extend java.lang.Enum
class _Week extends Enum<_Week> {
^
1 error
</script></code></pre>



Enum.getClass() 和 Enum.getDeclaringClass() 的区别在哪里？
1) `getClass()`方法：继承自 Object 类，返回一个 Class 类对象（类型信息）；
2) `getDeclaringClass()`方法：Enum 类定义的 public 方法，返回枚举常量对应的枚举类型的 Class 对象；

[Stack Overflow 提问帖（已有答案）地址](https://stackoverflow.com/questions/5758660/java-enum-getdeclaringclass-vs-getclass)

讨论的结果：
1) 对于没有定义匿名类的枚举类型，这两个方法没有区别；比如：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        out.printf("Week.class == Week.Sun.getClass() == Week.Fri.getDeclaringClass() >>> %b\n",
                   Week.class == Week.Sun.getClass() && Week.class == Week.Fri.getDeclaringClass());
    }
}

enum Week {
    Sun, Mon, Tues, Wed, Thur, Fri, Sat;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:04:06]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:04:16]
$ java Main
Week.class == Week.Sun.getClass() == Week.Fri.getDeclaringClass() >>> true
</script></code></pre>



2) 对于定义了匿名类的复杂枚举，getClass() 始终返回当前对象对应的类的 Class 对象，getDeclaringClass() 返回枚举常量所属的枚举类型的 Class 对象；比如：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        out.printf("Week.class == Week.Sun.getClass() == Week.Fri.getDeclaringClass() >>> %b\n",
                   Week.class == Week.Sun.getClass() && Week.class == Week.Fri.getDeclaringClass()); // false
        out.printf("Week.Sun.getClass() == Week.Mon.getClass() >>> %b\n",
                   Week.Sun.getClass() == Week.Mon.getClass()); // false
        out.printf("Week.Mon.getClass() == Week.Fri.getClass() >>> %b\n",
                   Week.Mon.getClass() == Week.Fri.getClass()); // true
        out.printf("Week.Sun.getDeclaringClass() == Week.Fri.getDeclaringClass() >>> %b\n",
                   Week.Sun.getDeclaringClass() == Week.Fri.getDeclaringClass()); // true
        out.printf("Week.Sun.getDeclaringClass() == Week.Fri.getClass() >>> %b\n",
                   Week.Sun.getDeclaringClass() == Week.Fri.getClass()); // true
    }
}

enum Week {
    Sun{ // 内部匿名类
        @Override
        public String toString() {
            return "Sunday";
        }
    },
    Mon, Tues, Wed, Thur, Fri, Sat;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:13:46]
$ ls
Main.java

# root @ arch in ~/work on git:master x [13:13:46]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:13:49]
$ ls
 Main.class   Main.java  'Week$1.class'   Week.class

# root @ arch in ~/work on git:master x [13:13:49]
$ java Main
Week.class == Week.Sun.getClass() == Week.Fri.getDeclaringClass() >>> false
Week.Sun.getClass() == Week.Mon.getClass() >>> false
Week.Mon.getClass() == Week.Fri.getClass() >>> true
Week.Sun.getDeclaringClass() == Week.Fri.getDeclaringClass() >>> true
Week.Sun.getDeclaringClass() == Week.Fri.getClass() >>> true
</script></code></pre>



3) 因此，如果要获取枚举类型的 Class 对象，请统一使用 getDeclaringClass() 方法；

**values()方法、valueOf()方法**
这里指的是由 enum 关键字定义的枚举类型内自动生成的 values()、valueOf() 方法；
在 Enum 基类中并没有定义 values() 方法，只有在具体的枚举类中才有，它返回枚举类中所有的枚举常量集合（数组）；
在 Enum 基类中定义了 valueOf() 方法，但是它接受两个参数`public static <T extends Enum<T>> T valueOf(Class<T> enumType, String name)`，在具体的枚举类中的 valueOf() 只有一个 String 参数即枚举的字符串形式，本质还是调用的 Enum.valueOf() 方法；

使用例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Week fri = Week.valueOf("Fri");
        out.printf("Week.valueOf(\"Fri\") == Week.Fri >>> %b\n", fri == Week.Fri);

        for (Week day : Week.values()) {
            out.println(day.name());
        }
    }
}

enum Week {
    Sun, Mon, Tues, Wed, Thur, Fri, Sat;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:30:51] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [13:30:54]
$ java Main
Week.valueOf("Fri") == Week.Fri >>> true
Sun
Mon
Tues
Wed
Thur
Fri
Sat
</script></code></pre>



但是，如果将 Week 向上转型为 Enum 类型，那么就不能像这样直接调用 values() 方法了；
但是可以通过 Class 类的`public T[] getEnumConstants();`方法间接获取，`public boolean isEnum();`方法判断是否为 Enum 类型；例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Week days[] = Week.values(); // 正常使用
        Enum<Week> e = Week.Sun; // 向上转型为 Enum 类型
        // e.values(); // 编译错误，找不到 values() 方法

        // 通过 Class 的相关方法可获取
        Class<?> clasz = e.getDeclaringClass();
        if (clasz.isEnum()) {
            for (Week day : (Week[])clasz.getEnumConstants()) {
                out.println(day.name());
            }
        }
    }
}

enum Week {
    Sun, Mon, Tues, Wed, Thur, Fri, Sat;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:44:53]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:44:55]
$ java Main
Sun
Mon
Tues
Wed
Thur
Fri
Sat
</script></code></pre>



## Enum进阶用法
自定义 private/[default] 构造函数，需要注意的几点是：
1) 访问修饰符只能为`private`、`默认`；
2) 不能在构造函数内部调用`super(...)`基类的构造函数；
3) 必须先定义完枚举常量后才能自定义构造函数及其他成员，否则编译报错；

<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        for (Week day : Week.values()) {
            out.println(day); // 自动调用 toString() 方法
        }
    }
}

enum Week {
    Sun("Sunday"),
    Mon("Monday"),
    Tues("Tuesday"),
    Wed("Wednesday"),
    Thur("Thursday"),
    Fri("Friday"),
    Sat("Saturday"); // ;分号结束

    private String name;
    private Week(String name) {
        this.name = name;
    }

    // Getter、Setter 方法
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    // 重写 Enum.toString() 方法
    @Override
    public String toString() {
        return name;
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:59:24] C:130
$ javac Main.java

# root @ arch in ~/work on git:master x [13:59:26]
$ java Main
Sunday
Monday
Tuesday
Wednesday
Thursday
Friday
Saturday
</script></code></pre>



定义枚举常量的内部方法（实则为一个内部匿名类）
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        for (Week day : Week.values()) {
            out.println(day.getInfo());
        }
    }
}

enum Week {
    Sun("Sunday") {
        @Override
        public String getInfo() {
            return name + "_星期天";
        }
    },

    Mon("Monday") {
        @Override
        public String getInfo() {
            return name + "_星期一";
        }
    },

    Tues("Tuesday") {
        @Override
        public String getInfo() {
            return name + "_星期二";
        }
    },

    Wed("Wednesday") {
        @Override
        public String getInfo() {
            return name + "_星期三";
        }
    },

    Thur("Thursday") {
        @Override
        public String getInfo() {
            return name + "_星期四";
        }
    },

    Fri("Friday") {
        @Override
        public String getInfo() {
            return name + "_星期五";
        }
    },

    Sat("Saturday") {
        @Override
        public String getInfo() {
            return name + "_星期六";
        }
    };

    protected String name;
    private Week(String name) {
        this.name = name;
    }

    // 该方法不能被直接调用，需要在匿名类中 Override
    // 也可以将 getInfo() 方法定义为 abstract 抽象方法
    public String getInfo() {
        throw new AbstractMethodError();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:18:56]
$ ls
Main.java

# root @ arch in ~/work on git:master x [14:18:57]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:18:59]
$ ls
 Main.class  'Week$1.class'  'Week$3.class'  'Week$5.class'  'Week$7.class'
 Main.java   'Week$2.class'  'Week$4.class'  'Week$6.class'   Week.class

# root @ arch in ~/work on git:master x [14:19:00]
$ java Main
Sunday_星期天
Monday_星期一
Tuesday_星期二
Wednesday_星期三
Thursday_星期四
Friday_星期五
Saturday_星期六
</script></code></pre>



enum 枚举类型还可以实现接口；比如：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        A a = E.RED;
        a.fa();

        B b = E.GREEN;
        b.fb();
    }
}

interface A {
    public void fa();
}

interface B {
    public void fb();
}

enum E implements A, B {
    RED, GREEN, BLUE; // 定义枚举常量

    // 实现接口 A
    @Override
    public void fa() {
        out.println("A::fa()");
    }

    // 实现接口 B
    @Override
    public void fb() {
        out.println("B::fb()");
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:26:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:26:28]
$ java Main
A::fa()
B::fb()
</script></code></pre>



## EnumMap、EnumSet
**EnumMap**
1) EnumMap 是枚举的专属的 Map 键值对集合（key 必须为 Enum 类型）；
2) EnumMap 因为其内部是通过数组实现的，所以相对来说效率比普通的 HashMap 高；
3) EnumMap 虽说是枚举专属集合，但其操作与一般的 Map 差不多；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.List;
import java.util.ArrayList;
import java.util.Map;
import java.util.EnumMap;

public class Main {
    public static void main(String[] args) {
        // color 集合
        List<Color> colors = new ArrayList<Color>(10);
        colors.add(Color.RED);
        colors.add(Color.BLUE);
        colors.add(Color.BLUE);
        colors.add(Color.GREEN);
        colors.add(Color.RED);
        colors.add(Color.GREEN);
        colors.add(Color.BLUE);
        colors.add(Color.GREEN);
        colors.add(Color.RED);
        colors.add(Color.BLUE);

        // map 统计
        Map<Color, Integer> enumMap = new EnumMap<Color, Integer>(Color.class);
        for (Color color : colors) {
            if (enumMap.get(color) == null) { // 首次计数
                enumMap.put(color, 1);
            } else {
                enumMap.put(color, enumMap.get(color) + 1);
            }
        }

        // 统计结果
        out.println(enumMap.toString());
    }
}

enum Color {
    RED, GREEN, BLUE;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:01:54]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:02:07]
$ java Main
{RED=3, GREEN=3, BLUE=4}
</script></code></pre>



**EnumSet**
1) EnumSet 是与枚举类型一起使用的专用 Set 集合，EnumSet 中所有元素都必须是枚举类型；
2) EnumSet 与其他 Set 接口的实现类 HashSet/TreeSet（内部都是用对应的 HashMap/TreeMap 实现的）不同的是，EnumSet 在内部实现是`位向量`，它是一种极为高效的位运算操作，由于直接存储和操作都是 bit，因此 EnumSet 空间和时间性能都十分可观，足以媲美传统上基于 int 的“位标志”的运算；
3) EnumSet 不允许使用 null 元素；试图插入 null 元素将抛出 NullPointerException，但试图测试判断是否存在 null 元素或移除 null 元素则不会抛出异常
4) 与大多数 collection 实现一样，EnumSet 不是线程安全的；

创建 EnumSet 并不能使用 new 关键字，因为它是个抽象类，而应该使用其提供的静态工厂方法，常用的几个如下：
`public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType)`：创建一个空 Set；
`public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType)`：创建一个包含所有枚举常量的 Set；
`public static <E extends Enum<E>> EnumSet<E> range(E from, E to)`：创建一个指定范围的 Set；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.EnumSet;

public class Main {
    public static void main(String[] args) {
        // noneOf
        EnumSet<Week> s1 = EnumSet.noneOf(Week.class);
        out.println("添加前: " + s1.toString());
        s1.add(Week.Sun);
        s1.add(Week.Fri);
        s1.add(Week.Sat);
        s1.add(Week.Wed);
        s1.add(Week.Mon);
        s1.add(Week.Sun);
        out.println("添加后: " + s1.toString());

        // allOf
        EnumSet<Week> s2 = EnumSet.allOf(Week.class);
        out.println("allOf: " + s2.toString());

        // range
        EnumSet<Week> s3 = EnumSet.range(Week.Mon, Week.Fri);
        out.println("range: " + s3.toString());
    }
}

enum Week {
    Sun, Mon, Tues, Wed, Thur, Fri, Sat;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:25:23]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:25:33]
$ java Main
添加前: []
添加后: [Sun, Mon, Wed, Fri, Sat]
allOf: [Sun, Mon, Tues, Wed, Thur, Fri, Sat]
range: [Mon, Tues, Wed, Thur, Fri]
</script></code></pre>
