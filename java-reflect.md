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
<pre><code class="language-java line-numbers"><script type="text/plain">
public static final Class<Integer> TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
</script></code></pre>



Integer.class、Integer.TYPE、int.class 例子：
<pre><code class="language-java line-numbers"><script type="text/plain">
import static java.lang.System.in;
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

<pre><code class="language-java line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [13:56:43]
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
<pre><code class="language-java line-numbers"><script type="text/plain">
// 根据给定的 className 获取其 Class 对象
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
<pre><code class="language-java line-numbers"><script type="text/plain">
public Class<?> getDeclaringClass(); // 获取声明此字段的类的 Class 对象
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
<pre><code class="language-java line-numbers"><script type="text/plain">
import static java.lang.System.in;
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

<pre><code class="language-java line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [20:23:50]
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
<pre><code class="language-java line-numbers"><script type="text/plain">
public Class<?> getDeclaringClass(); // 获取声明该方法的类的 Class 对象
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

// 调用当前方法
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException;
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">
import static java.lang.System.in;
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

<pre><code class="language-java line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [20:39:38]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:39:59]
$ java Main
Test::staticFunc()
result1 -> 30
result2 -> 30
</script></code></pre>


## Constructor 类
java.lang.reflect.Constructor 类和 java.lang.Class 类一样，都是泛型类；

<pre><code class="language-java line-numbers"><script type="text/plain">
public Class<T> getDeclaringClass(); // 获取所属类的 Class 对象
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
<pre><code class="language-java line-numbers"><script type="text/plain">
import static java.lang.System.in;
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

<pre><code class="language-java line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [20:51:26]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:51:38]
$ java Main
public Test::Test(int)
private Test::Test(float)
</script></code></pre>


## Array 类
反射除了可以用于操作类的成员变量、成员函数、构造函数之外，还可以创建 Array 数组；

java.lang.reflect.Array 类的常用方法：
<pre><code class="language-java line-numbers"><script type="text/plain">
/**
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
<pre><code class="language-java line-numbers"><script type="text/plain">
import static java.lang.System.in;
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

<pre><code class="language-java line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [18:35:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [18:35:30]
$ java Main
[0, 1, 4, 9, 16]
[Main@15db9742, Main@6d06d69c, Main@7852e922, Main@4e25154f, Main@70dea4e]
</script></code></pre>
