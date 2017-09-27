---
title: Java 反射
date: 2017-09-25 11:24:00
categories:
- java
tags:
- java
keywords: Java 反射 reflect
---

> 
Java 反射

<!-- more -->

## 反射是什么
`反射(Reflection)`是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性；

Java 反射框架主要提供以下功能：
1、在运行时判断任意一个对象所属的类；
2、在运行时构造任意一个类的对象；
3、在运行时获取任意一个类的任意成员；
4、在运行时调用任意一个类的任意方法；

简而言之，我们可以利用反射在运行时动态地创建、操作一个对象。

**反射最重要的用途就是开发各种通用框架**。

对于框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心；
对于一般的开发者来说，即使不开发框架，了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

反射相关的类基本都在 java.lang.reflect 包中。

## Class 类
要使用反射，第一步就是获取类对应的 Class 对象。

java.lang.Class 对象是什么？
1、一个 Class 对象就像 C++ 中的一个 type_info 对象，它保存着一个特定的类型信息；
2、当我们将 .java 源文件编译为 .class 字节码文件后，编译器同时为我们创建了一个 Class 对象保存在 .class 文件中。

如何获取 Class 对象？
1、通过类的`.class`静态属性，如`String.class`；
2、通过继承自 Object 的`getClass()`成员方法，如`obj.getClass()`；
3、通过 Class 类的`forName()`静态方法，如`Class<?> clasz = Class.forName("java.lang.String")`；

**包装类的`.class`、`.TYPE`属性区别**
1) `.class`静态属性：由 Java 自动生成，不需要也不能显式的声明该属性，它代表当前类的类型信息；
2) `.TYPE`静态属性：包装类独有的一个属性，是包装类显式声明的代表其基本类型的 Class 对象。

注意，除了可以通过包装类的`.TYPE`属性获取基本类型的 Class 对象，也可以直接通过基本类型来获取，比如`int.class`，它与`Integer.TYPE`是等价的。

以 Integer 为例，TYPE 的声明：
<pre><code class="language-java line-numbers"><script type="text/plain">public static final Class<Integer> TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
</script></code></pre>



Integer.class、Integer.TYPE、int.class 例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.in;
import static java.lang.System.out;
import static java.lang.System.err;

public class Main {
    public static void main(String[] args) {
        out.println(Integer.class);
        out.println(Integer.TYPE);
        out.println(int.class);

        out.println(int.class == Integer.class); // false
        out.println(int.class == Integer.TYPE); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:56:43]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:57:13]
$ java Main
class java.lang.Integer
int
int
false
true
</script></code></pre>



**Class 类的常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">// 根据给定的 className 获取其 Class 对象
public static Class<?> forName(String className) throws ClassNotFoundException;
public native Class<? super T> getSuperclass(); // 获取直接基类的 Class 对象
public Type getGenericSuperclass(); // 获取直接基类的 Class 对象(泛型)

// 调用无参构造函数创建对象，等同于执行 new T();
public T newInstance() throws InstantiationException, IllegalAccessException;

public native boolean isInstance(Object obj); // 判断给定对象是否为当前类的实例
public native boolean isAssignableFrom(Class<?> cls); // 判断给定类是否为当前类或其子类
public T cast(Object obj); // 将给定对象转换为当前类的对象

public boolean isEnum(); // 判断是否为 Enum 枚举类型
public T[] getEnumConstants(); // 获取枚举类型的所有枚举常量(同enum.values()方法)

public native boolean isInterface(); // 判断是否为 interface 接口
public native boolean isArray(); // 判断是否为 Array 静态数组
public native boolean isPrimitive(); // 判断是否为 基本类型
public boolean isAnnotation(); // 判断是否为 注解类型
public boolean isSynthetic(); // 判断是否为 合成类
public boolean isMemberClass(); // 判断是否为 成员类
public boolean isAnonymousClass(); // 判断是否为 匿名类

public String getName(); // 获取类型名称
public String getSimpleName(); // 获取简单名称(不包含package包信息)
public String getCanonicalName(); // 获取规范名称(基本同getSimpleName)
public String toString(); // 获取字符串描述
public String toGenericString(); // 获取泛型字符串描述

// 返回其声明类的 Class 对象 (如果当前类为某个类的成员类)
public Class<?> getDeclaringClass() throws SecurityException;

/*
 * getXXX()         获取 public 成员(包括继承而来的)
 * getDeclaredXXX() 获取其声明的所有成员(不包括继承而来的)
 * 下同.
 */

// 获取该类的成员类(内部类)
public Class<?>[] getClasses();
public Class<?>[] getDeclaredClasses() throws SecurityException;

public Package getPackage(); // 获取 package 信息
public native Class<?> getComponentType(); // 获取数组元素的 Class 对象
public native int getModifiers(); // 获取修饰符，需使用 Modifier 进行解析
public Class<?>[] getInterfaces(); // 获取当前类实现的所有接口(不包括继承的)
public Type[] getGenericInterfaces(); // (泛型)

// 获取指定字段、所有字段，字段即成员变量
public Field getField(String name) throws NoSuchFieldException, SecurityException;
public Field[] getFields() throws SecurityException;
public Field getDeclaredField(String name) throws NoSuchFieldException, SecurityException;
public Field[] getDeclaredFields() throws SecurityException;

// 获取指定方法、所有方法，parameterTypes 为参数对应的 Class 对象
public Method getMethod(String name, Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException;
public Method[] getMethods() throws SecurityException;
public Method getDeclaredMethod(String name, Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException;
public Method[] getDeclaredMethods() throws SecurityException;

// 获取指定构造函数、所有构造函数，parameterTypes 为参数对应的 Class 对象
public Constructor<T> getConstructor(Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException;
public Constructor<?>[] getConstructors() throws SecurityException;
public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException;
public Constructor<?>[] getDeclaredConstructors() throws SecurityException;
}
</script></code></pre>


## Field 类
所谓的字段（Field），就是我们平常所说的成员变量、属性。

java.lang.reflect.Field 类的相关方法：
<pre><code class="language-java line-numbers"><script type="text/plain">public Class<?> getDeclaringClass(); // 获取声明此字段的类的 Class 对象
public String getName(); // 获取字段名称
public int getModifiers(); // 获取字段修饰符

public boolean isEnumConstant(); // 判断当前字段是否为枚举常量
public boolean equals(Object obj); // 判断与给定字段是否相同
public int hashCode(); // 获取 hashCode 值
public String toString(); // 获取字符串描述

// 获取当前字段在给定对象 obj 中的值
public Object get(Object obj) throws IllegalArgumentException, IllegalAccessException;
public boolean getBoolean(Object obj) throws IllegalArgumentException, IllegalAccessException;
public byte getByte(Object obj) throws IllegalArgumentException, IllegalAccessException;
public char getChar(Object obj) throws IllegalArgumentException, IllegalAccessException;
public short getShort(Object obj) throws IllegalArgumentException, IllegalAccessException;
public int getInt(Object obj) throws IllegalArgumentException, IllegalAccessException;
public long getLong(Object obj) throws IllegalArgumentException, IllegalAccessException;
public float getFloat(Object obj) throws IllegalArgumentException, IllegalAccessException;
public double getDouble(Object obj) throws IllegalArgumentException, IllegalAccessException;

// 设置当前字段在给定对象 obj 中的值
public void set(Object obj, Object value) throws IllegalArgumentException, IllegalAccessException;
public void setBoolean(Object obj, boolean z) throws IllegalArgumentException, IllegalAccessException;
public void setByte(Object obj, byte b) throws IllegalArgumentException, IllegalAccessException;
public void setChar(Object obj, char c) throws IllegalArgumentException, IllegalAccessException;
public void setShort(Object obj, short s) throws IllegalArgumentException, IllegalAccessException;
public void setInt(Object obj, int i) throws IllegalArgumentException, IllegalAccessException;
public void setLong(Object obj, long l) throws IllegalArgumentException, IllegalAccessException;
public void setFloat(Object obj, float f) throws IllegalArgumentException, IllegalAccessException;
public void setDouble(Object obj, double d) throws IllegalArgumentException, IllegalAccessException;
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.in;
import static java.lang.System.out;
import static java.lang.System.err;
import java.lang.reflect.Field;

public class Main {
    public int pub = 10;
    protected int prot = 20;
    private int priv = 30;
    int defa = 40;

    @Override
    public String toString() {
        return String.format("[Main] -> {%d, %d, %d, %d}", pub, prot, priv, defa);
    }

    public static void main(String[] args) {
        // 获取 public 字段 [Main]
        for (Field f : Main.class.getFields()) {
            out.println(f);
        }

        // 获取 public 字段 [Test]
        out.println("-------------------------");
        for (Field f : Test.class.getFields()) {
            out.println(f);
        }

        // 获取所有声明的字段 [Main]
        out.println("-------------------------");
        for (Field f : Main.class.getDeclaredFields()) {
            out.println(f);
        }

        // 获取所有声明的字段 [Test]
        out.println("-------------------------");
        for (Field f : Test.class.getDeclaredFields()) {
            out.println(f);
        }

        // 操作 Main 类的私有成员
        out.println("-------------------------");
        Main main = new Main();
        out.println(main);
        try {
            Field mainPriv = Main.class.getDeclaredField("priv");
            // mainPriv.setAccessible(true); // 可有可无
            mainPriv.setInt(main, 999);
            out.println(main);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }

        // 操作 Test 类的私有成员
        out.println("-------------------------");
        Test test = new Test();
        out.println(test);
        try {
            Field testPriv = Test.class.getDeclaredField("priv");
            testPriv.setAccessible(true); // 必须修改访问性 [破坏类的封装性]
            testPriv.setInt(test, 999);
            out.println(test);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}

class Test {
    public int pub = 100;
    protected int prot = 200;
    private int priv = 300;
    int defa = 400;

    @Override
    public String toString() {
        return String.format("[Test] -> {%d, %d, %d, %d}", pub, prot, priv, defa);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:23:50]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:24:04]
$ java Main
public int Main.pub
-------------------------
public int Test.pub
-------------------------
public int Main.pub
protected int Main.prot
private int Main.priv
int Main.defa
-------------------------
public int Test.pub
protected int Test.prot
private int Test.priv
int Test.defa
-------------------------
[Main] -> {10, 20, 30, 40}
[Main] -> {10, 20, 999, 40}
-------------------------
[Test] -> {100, 200, 300, 400}
[Test] -> {100, 200, 999, 400}
</script></code></pre>


## Method 类
<pre><code class="language-java line-numbers"><script type="text/plain">public Class<?> getDeclaringClass(); // 获取声明该方法的类的 Class 对象
public String getName(); // 获取方法名称
public int getModifiers(); // 获取方法修饰符
public Class<?> getReturnType(); // 获取返回值的 Class 对象
public Class<?>[] getParameterTypes(); // 获取参数的 Class 对象
public int getParameterCount(); // 获取参数的个数
public Class<?>[] getExceptionTypes(); // 获取异常声明的 Class 对象

public boolean equals(Object obj); // 判等
public int hashCode(); // hashCode 值
public String toString(); // 获取字符串描述

public boolean isVarArgs(); // 判断当前方法是否使用可变参数
public boolean isDefault(); // 判断是否为接口中的默认方法

// 调用当前方法
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException;
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.in;
import static java.lang.System.out;
import static java.lang.System.err;
import java.lang.reflect.Method;
import java.lang.reflect.InvocationTargetException;

public class Main {
    public static void main(String[] args) {
        Class<?> clazz = Test.class;

        try {
            Method staticFunc = clazz.getMethod("staticFunc");
            Method pubAdd = clazz.getMethod("pubAdd", int.class, int.class);
            Method priAdd = clazz.getDeclaredMethod("priAdd", int.class, int.class);

            staticFunc.invoke(null); // static方法

            Integer result1 = (Integer) pubAdd.invoke(new Test(), 10, 20);
            out.println("result1 -> " + result1);

            priAdd.setAccessible(true);
            Integer result2 = (Integer) priAdd.invoke(new Test(), 10, 20);
            out.println("result2 -> " + result2);
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

class Test {
    public static void staticFunc() {
        out.println("Test::staticFunc()");
    }

    public int pubAdd(int a, int b) {
        return a + b;
    }

    private int priAdd(int a, int b) {
        return a + b;
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:39:38]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:39:59]
$ java Main
Test::staticFunc()
result1 -> 30
result2 -> 30
</script></code></pre>


## Constructor 类
java.lang.reflect.Constructor 类和 java.lang.Class 类一样，都是泛型类；

<pre><code class="language-java line-numbers"><script type="text/plain">public Class<T> getDeclaringClass(); // 获取所属类的 Class 对象
public String getName(); // 获取所属类的名称
public int getModifiers(); // 获取修饰符
public Class<?>[] getParameterTypes(); // 获取参数列表的 Class 对象
public int getParameterCount(); // 获取参数的个数
public Class<?>[] getExceptionTypes(); // 获取异常列表的 Class 对象

public boolean equals(Object obj); // 判等
public int hashCode(); // hashCode 值
public String toString(); // 获取字符串描述

public boolean isVarArgs(); // 判断是否为变参函数

// 创建新对象
public T newInstance(Object ... initargs) throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException;
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.in;
import static java.lang.System.out;
import static java.lang.System.err;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class Main {
    public static void main(String[] args) {
        Class<Test> clazz = Test.class;

        try {
            Constructor<Test> testPubCon = clazz.getConstructor(int.class);
            Constructor<Test> testPriCon = clazz.getDeclaredConstructor(float.class);

            testPubCon.newInstance(10);

            testPriCon.setAccessible(true);
            testPriCon.newInstance(3.14f);
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException | InstantiationException e) {
            e.printStackTrace();
        }
    }
}

class Test {
    public Test(int arg) {
        out.println("public Test::Test(int)");
    }

    private Test(float arg) {
        out.println("private Test::Test(float)");
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:51:26]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:51:38]
$ java Main
public Test::Test(int)
private Test::Test(float)
</script></code></pre>


## Array 类
反射除了可以用于操作类的成员变量、成员函数、构造函数之外，还可以创建 Array 数组；

java.lang.reflect.Array 类的常用方法：
<pre><code class="language-java line-numbers"><script type="text/plain">/**
 * 创建一个 length 长的元素类型为 componentType 的静态数组.
 * @param componentType 数组元素的 Class 对象
 * @param length 数组长度
 * @throws NegativeArraySizeException RuntimeException 异常，当 length 为负数时抛出
 * @throws IllegalArgumentException RuntimeException 异常，数组维数超过 255 时抛出
 * @return Object 返回数组的引用
 */
public static Object newInstance(Class<?> componentType, int length) throws NegativeArraySizeException;
// 获取数组长度
public static native int getLength(Object array) throws IllegalArgumentException;

// 获取指定索引的元素
public static native Object get(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native boolean getBoolean(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native byte getByte(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native char getChar(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native short getShort(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native int getInt(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native long getLong(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native float getFloat(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native double getDouble(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;

// 修改指定索引的元素
public static native void set(Object array, int index, Object value) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setBoolean(Object array, int index, boolean z) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setByte(Object array, int index, byte b) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setChar(Object array, int index, char c) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setShort(Object array, int index, short s) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setInt(Object array, int index, int i) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setLong(Object array, int index, long l) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setFloat(Object array, int index, float f) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
public static native void setDouble(Object array, int index, double d) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.in;
import static java.lang.System.out;
import static java.lang.System.err;
import java.util.Arrays;
import java.lang.reflect.Array;

public class Main {
    public static void main(String[] args) {
        // 基本类型
        int[] intArr = (int[]) Array.newInstance(int.class, 5);
        for (int i = 0; i < intArr.length; i++) {
            intArr[i] = i * i;
        }
        out.println(Arrays.toString(intArr));

        // 引用类型
        Main[] intMain = (Main[]) Array.newInstance(Main.class, 5);
        for (int i = 0; i < intMain.length; i++) {
            intMain[i] = new Main();
        }
        out.println(Arrays.toString(intMain));
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:35:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [18:35:30]
$ java Main
[0, 1, 4, 9, 16]
[Main@15db9742, Main@6d06d69c, Main@7852e922, Main@4e25154f, Main@70dea4e]
</script></code></pre>


## Annotation 注解
**什么是注解**
注解是 Java5 的新特性。注解可以看做一种注释或者元数据（MetaData），可以把它插入到我们的 Java 代码中，用来描述我们的 Java 类，从而影响 Java 类的行为。

**注解的构成**
一个 Java 注解由一个`@`符后面跟一个`标识符`构成，类似于这样：`@MyAnnotation`。

Java 注解中一般包含一些**元素**，这些元素类似于属性或者参数，可以用来设置值；
比如我们有一个包含两个元素的`@Entity`注解：`@Entity(tableName = "vehicles", primaryKey = "id")`
上面注解中有两个元素，`tableName`和`primaryKey`，它们各自都被赋予了自己的元素值。

**注解的位置**
Java 注解可用于注解：`package包`、`类/接口/enum枚举`、`注解`(**元注解**)、`构造函数`、`方法`、`方法参数`、`字段`、`局部变量`等。

在下边这个例子中，注解分别用在了类、字段、方法、参数和局部变量中：
<pre><code class="language-java line-numbers"><script type="text/plain">// 注解一个类
@Entity
public class Vehicle {
    // 注解一个字段
    @Persistent
    protected String vehicleName = null;

    // 注解一个方法
    @Getter
    public String getVehicleName() {
        return this.vehicleName;
    }

    // 注解一个参数
    public void setVehicleName(@Optional vehicleName) {
        this.vehicleName = vehicleName;
    }

    public List addVehicleNameToList(List names) {
        // 注解一个局部变量
        @Optional
        List localNames = names;

        if (localNames == null) {
            localNames = new ArrayList();
        }

        localNames.add(getVehicleName());
        return localNames;
    }
}
</script></code></pre>



**三种内置注解**
Java 本身提供了三个内置注解：`@Override`、`@Deprecated`、`@SuppressWarnings`；
它们都位于 java.lang 包。

1) `@Override`，可注解**方法**；**源码**(Source)注解，是一个标记注解（没有元素）；
用于在编译期间检查当前方法是否正确的重写了基类中的被重写方法。
如果没有则抛出一个编译错误，若已正确重写则该注解被忽略，不会保存到 .class 字节码文件中。

2) `@Deprecated`，可注解除注解外的所有 Target，**运行时**(Runtime)注解，该注解会被包含至 java doc 中，是一个标记注解（没有元素）；
表示被注解的 Target 不被 Java 赞成使用，这些 Target 是过时的，废弃的 API，如果使用了这些 Target 则抛出编译警告。

3) `@SuppressWarnings`，可注解除注解、Package包外的所有 Target，**源码**(Source)注解，该注解有一个元素`String[] value();`；
用于压制被注解的 Target 中出现的 Warnings 编译警告，使得在编译的时候不会出现编译警告。

**创建自定义注解**
除了可以使用 Java 内置注解外，我们还可以创建自定义注解；

定义注解使用关键字`@interface`，就和定义一个 interface 接口一样简单；
Annotation 注解和 interface 接口其实很相似，可以说注解就是一种特殊的 interface 接口。

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.in;
import static java.lang.System.out;
import static java.lang.System.err;

public class Main {
    public static void main(String[] args) {
        @MyAnnotation
        Object obj = new Object();
    }
}

// 标记注解，默认作用范围为所有 Target & 默认生命周期为 CLASS
@interface MyAnnotation {
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:07:48]
$ ls
Main.java

# root @ arch in ~/work on git:master x [10:07:49]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:07:52]
$ java Main

# root @ arch in ~/work on git:master x [10:07:57]
$ ls
Main.class  Main.java  MyAnnotation.class

# root @ arch in ~/work on git:master x [10:07:58]
$ javap -c -p MyAnnotation
Compiled from "Main.java"
interface MyAnnotation extends java.lang.annotation.Annotation {
}

# root @ arch in ~/work on git:master x [10:08:07]
$ javap -c -p Main
Compiled from "Main.java"
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/Object
       3: dup
       4: invokespecial #1                  // Method java/lang/Object."<init>":()V
       7: astore_1
       8: return
}
</script></code></pre>



从反编译结果来看，注解类型 MyAnnotation 实际上就是一个继承自 java.lang.annotation.Annotation 接口的接口；
定义一个注解使用关键字：`@interface`、定义一个接口使用关键字：`interface`；

那么我们可以大胆推断，注解也可以有成员变量和成员方法，并且和接口是一样的：

首先我们来看一下 java.lang.annotation.Annotation 接口的定义：
<pre><code class="language-java line-numbers"><script type="text/plain">public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();
    Class<? extends Annotation> annotationType();
}
</script></code></pre>



我们只需要实现 annotationType() 方法，其它的几个 Object 类都帮我们提供了默认实现；

1) 尝试模拟注解类型，失败：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.lang.annotation.Annotation;

public class Main {
    public static void main(String[] args) {
        @MyAnnotation
        Main main = null;
    }
}

interface MyAnnotation extends Annotation {}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:36:42]
$ javac Main.java
Main.java:5: error: MyAnnotation is not an annotation type
        @MyAnnotation
         ^
1 error
</script></code></pre>



2) 尝试实现 Annotation 接口，成功：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.lang.annotation.Annotation;

class A implements Annotation {
    @Override
    public Class<A> annotationType() {
        return A.class;
    }
}

@interface MyAnnotation {
}

class B implements MyAnnotation {
    @Override
    public Class<B> annotationType() {
        return B.class;
    }
}
</script></code></pre>



3) 注解也可以有方法和属性，成功：
<pre><code class="language-java line-numbers"><script type="text/plain">@interface MyAnnotation {
    int field = 100;
    int method();
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:48:45]
$ javap -c -p MyAnnotation
Compiled from "Main.java"
interface MyAnnotation extends java.lang.annotation.Annotation {
  public static final int field;

  public abstract int method();
}
</script></code></pre>



和接口一样，注解中的字段都是`public static final`修饰的，方法都是`public abstract`修饰的。

**注解中的元素**
前面我们创建的自定义注解是一个标记注解，即没有元素的注解。
注解也可以有元素，元素可以有多个，元素其实就是接口中的抽象方法。

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        @MyAnnotation(name = "ZhangSan",
                      age = 14,
                      score = 89.5f,
                      array = {1, 2, 3, 4, 5})
        Main main = null;
    }
}

@interface MyAnnotation {
    String name();
    int age();
    float score();
    int[] array();
}
</script></code></pre>



在给元素赋值的时候需要指明元素的名称，语法和调用一个函数是一样的；

元素的类型可以是：
1) 八大基本类型（boolean、byte、char、short、int、long、float、double）
2) String 字符串
3) Class 类
4) enum 枚举
5) Annotation 注解接口类型
6) 以上类型的数组

并且只能使用这些类型，使用其它类型会抛出一个编译错误；

**元素的默认值**
只需要使用 default 关键字就可以为一个元素指定其默认值，如：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        @MyAnnotation(name = "ZhangSan",
                      age = 14,
                      score = 89.5f/*,
                      array = {1, 2, 3, 4, 5}*/)
        Main main = null;
    }
}

@interface MyAnnotation {
    String name();
    int age();
    float score();
    int[] array() default {1, 2, 3, 4, 5};
}
</script></code></pre>



注意，提供的默认值不能为 null 空指针，只能为一个字面量或者是一个 final 常量（其值为一个字面量）；

**value 元素**
如果注解只有一个元素，并且元素名为`value`，那么可以省略元素名，直接赋值，如：
<pre><code class="language-java line-numbers"><script type="text/plain">@TestA("A")
public class Main {
    @TestB({"A", "B", "C"})
    public static void main(String[] args) {}
}

@interface TestA {
    String value();
}

@interface TestB {
    String[] value();
}
</script></code></pre>



所以，如果注解只有一个元素，请务必将它命名为 value。


**四种元注解**
元注解：即注解的注解，你可以和数据、元数据进行类比。

java.lang.annotation 包提供了 4 种元注解：
1) `@Target`，限制注解的**作用范围**，作用范围由 java.lang.annotation.ElementType 枚举定义：
- `TYPE`：类、接口(接口包括注释类型)、enum 枚举
- `FIELD`：字段，即属性、成员变量
- `METHOD`：方法
- `PARAMETER`：方法参数
- `CONSTRUCTOR`：构造器
- `LOCAL_VARIABLE`：局部变量
- `ANNOTATION_TYPE`：注解类型
- `PACKAGE`：包
- `TYPE_PARAMETER`：类型参数（JDK1.8）
- `TYPE_USE`：类型使用声明（JDK1.8）



2) `@Retention`，限制注解的**生命周期**，生命周期由 java.lang.annotation.RetentionPolicy 枚举定义：
- `SOURCE`：注解仅存于源代码中，在编译过程中有效，并不会保存至 .class 字节码文件中
- `CLASS`：默认值，注解在 .class 字节码文件中可用，在类被加载时被 JVM 丢弃
- `RUNTIME`：注解在运行时也可用，JVM 并不会将注解信息丢弃，可以通过 reflect 反射 API 获取注解信息



3) `@Inherited`，表示此注解是可继承的，如果父类被此注解标示，那么该父类的子类也会继承父类的该注解

4) `@Documented`，注解可以由 javadoc 及类似工具记录


**通过反射获取 Annotation 注解信息**
若想通过反射获取注解信息，那么注解的生命周期必须为 RUNTIME，否则在被 VM 加载到内存中之前注解就被丢弃了。

1) Class 类中与 Annotation 相关的方法：
<pre><code class="language-java line-numbers"><script type="text/plain">public boolean isAnnotation(); // 判断当前 Class 代表的是否为 注解类型

// 判断当前类上是否有指定注解，通常用于标记注解
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass);

// 获取当前类的指定注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass);

// 获取当前类的所有注解
public Annotation[] getAnnotations();
// 获取当前类的所有注解(不包含继承的注解)
public Annotation[] getDeclaredAnnotations();
</script></code></pre>



2) Field 类中与 Annotation 相关的方法：
<pre><code class="language-java line-numbers"><script type="text/plain">// 获取当前字段上的指定注解
public <T extends Annotation> T getAnnotation(Class<T> annotationClass);

// 获取当前字段上的所有注解
public Annotation[] getDeclaredAnnotations();
</script></code></pre>



3) Method 类中与 Annotation 相关的方法：
<pre><code class="language-java line-numbers"><script type="text/plain">// 获取当前方法上的指定注解
public <T extends Annotation> T getAnnotation(Class<T> annotationClass);

// 获取当前方法上的所有注解
public Annotation[] getDeclaredAnnotations();

// 获取当前方法的所有参数的所有注解
public Annotation[][] getParameterAnnotations();
</script></code></pre>



4) Constructor 类中与 Annotation 相关的方法：
<pre><code class="language-java line-numbers"><script type="text/plain">// 获取当前构造器上的指定注解
public <T extends Annotation> T getAnnotation(Class<T> annotationClass);

// 获取当前构造器上的所有注解
public Annotation[] getDeclaredAnnotations();

// 获取当前构造器的所有参数的所有注解
public Annotation[][] getParameterAnnotations();
</script></code></pre>
