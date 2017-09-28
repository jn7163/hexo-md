---
title: javadoc 生成 API 文档
date: 2017-09-27 16:28:00
categories:
- java
tags:
- java
keywords: javadoc API文档
---

> 
javadoc 生成 API 文档

<!-- more -->

## javadoc 是什么
`javadoc`是 JDK 中自带的一个很有用的工具，它从源代码中抽取类、方法、成员等注释形成一个和源代码配套的 API 帮助文档。
也就是说，只要在编写程序时以一套特定的标签作注释，在程序编写完成后，通过 javadoc 工具就可以同时形成程序的开发文档了。

Java 中有 3 种注释方法：
1) `// ...`：单行注释
2) `/* ... */`：多行注释
3) `/** ... */`：多行注释

其中第三种是 Java 专门为 javadoc 设计的，它可以被 javadoc 工具解析为相关的 HTML 文档。

[Oracle 官方使用文档 - javadoc](http://docs.oracle.com/javase/8/docs/technotes/tools/windows/javadoc.html)

如果你经常查阅[Java API - Oracle 官方文档](https://docs.oracle.com/javase/8/docs/api/)
你会发现，这个庞大的在线 API 文档其实就是使用 javadoc 工具生成的。


## 文档注释的格式
**文档注释的位置**
`包`(**`package-info.java`**)、`类`、`接口`、`构造器`、`方法`、`属性`

**文档注释的三部分**
1) **简述**：简述位于注释的起始位置，直到出现第一个`.`句号(包括句号)；
2) **详细说明**：详细说明包括简述，对格式没有特别要求，可以有若干个句号；
3) **标记部分**：所有的标记都以`@`开头，包括版本说明、参数说明、返回值说明等；

注意，在 javadoc 注释中可以使用 HTML 标签，比如：
换行不应该直接敲入一个回车，而应使用标签`<br>`；
起一个段落应使用标签`<p>`放在段落的起始位置；

**`@`块标记**
常用的块标记有：
1) `@author 作者`：用于包、类、接口，可以有多个；
2) `@version 版本号`：用于包、类、接口；
3) `@since JDKx.x`：支持的 JDK 版本，用于包、类、接口、构造器、方法、属性；
4) `@param 参数名 描述`：形参的说明，用于构造器、方法；
5) `@return 描述`：函数返回值的说明，用于方法；
6) `@throws 异常类名 描述`：可能会抛出的异常说明，用于构造器、方法；
7) `@exception 异常类名 描述`：同`@throws`标记；
8) `@deprecated 描述`：对已过期 API 的描述，用于包、类、接口、构造器、方法、属性；
9) `@see 包.类#字段/方法(param...)`：如果跳转的 url 就是当前类的其他方法/字段，则`#`之前的可以省略，用于包、类、接口、构造器、方法、属性；
10) `{@link 包.类#字段/方法(param...)}`：同`@see`标记；
11) `{@value 包.类#字段}`：引用 final 常量的值，用于 final 属性；
12) `{@code text}`：不进行 HTML 解析，表示一个纯文本的代码；
13) `{@inheritDoc}`：从继承的类、接口中继承注释，可以进行拼接，参数、返回值、异常可以在子类中覆盖父类的相关注释；


常用 HTML 标签：
1) `<br>`：换行标签
2) `<p>`：段落标签
3) `<blockquote><pre> ... </pre></blockquote>`：多行代码块
4) `<strong> ... </strong>`：粗体强调
5) `<em> ... </em>`：斜体强调
6) `<b> ... </b>`：显示粗体
7) `<i> ... </i>`：显示斜体


## javadoc 命令
`javadoc`命令的格式：`javadoc [选项] [包名] [源文件] [@files]`，常用选项：

`-public`：显示 public 成员
`-protected`：显示 protected/public 成员（默认）
`-private`：显示所有成员

`-sourcepath <pathlist>`：指定源文件的路径列表
`-classpath <pathlist>`：指定 CLASSPATH 搜索路径列表
`-cp <pathlist>`：同上

`-source <release>`：提供与指定发行版的源兼容性

`-locale <name>`：指定语言环境（须放在所有选项的前面），如 en_US、en_US_WIN、zh_CN
`-encoding <name>`：指定源文件的字符编码

`-quiet`：安静模式
`-verbose`：详细输出模式

**以下参数由标准 doclet 提供**
`-d <directory>`：输出文件的目标路径

`-version`：包含`@version`字段
`-author`：包含`@author`字段
`-serialwarn`：生成有关`@serial`标记的警告
`-use`：创建类和 package 的用法页面

`-nodeprecated`：不包含`@deprecated`字段
`-nosince`：不包含`@since`字段

`-nodeprecatedlist`：不生成已过时的 API 列表
`-nocomment`：不生成说明和标记，只生成声明
`-notimestamp`：不包含隐藏时间戳
`-notree`：不生成类分层结构
`-noindex`：不生成索引
`-nohelp`：不生成帮助链接
`-nonavbar`：不生成导航栏

`-windowtitle <text>`：指定文档的 title 标题
`-docencoding <name>`：指定生成的 HTML 文档的字符编码

**常用命令举例**
<pre><code class="language-bash"><script type="text/plain">javadoc -locale zh_CN -d html/ -sourcepath /root/work/src/:/root/work/test/ com.zfl9.test1 com.zfl9.test2 /root/work/src/MyClass.java
# 语言环境为 zh_CN
# 生成的文档目录为 html/
# 源码搜索路径为 /root/work/src/:/root/work/test/
# 要生成 doc 的包为 com.zfl9.test1、com.zfl9.test2
# 另加一个单独的源文件 /root/work/src/MyClass.java

javadoc -locale zh_CN -d html/ -sourcepath /root/work/src/:/root/work/test/ @target.txt
# 使用 target.txt 文件中指定的包、源文件
# target.txt 文件的内容为：
com.zfl9.test1
com.zfl9.test2
/root/work/src/MyClass.java
</script></code></pre>



**例子**
1) 创建两个 package 包：com.zfl9.test1、com.zfl9.test2；
2) 在每个包中创建一个 package-info.java 包描述文件；
> 
因为两个包的内容很相似，所以这里只做 com.zfl9.test1 包的演示。

**package-info.java**
<pre><code class="language-java line-numbers"><script type="text/plain">/**
 * 测试包1，用于测试 javadoc.
 * 目前为止，这个包只有一个类 - {@code Test}<br>
 * 该类没有实际意义，仅用于测试.
 * @author Otokaze
 * @version v1.0-beta
 * @since JDK1.8
 */
package com.zfl9.test1;
</script></code></pre>

**Test.java**
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.test1;
import static java.lang.System.out;

/**
 * 这是一个测试类 - {@code Test}.<br>
 * 仅仅为了测试 javadoc，(￣▽￣)／
 * @author Otokaze
 * @version v1.0-beta
 * @since JDK1.8
 */
public class Test {
    /**
     * 字符串常量 - {@value #MESSAGE}.
     * 该字符串将被 {@code test()} 方法使用
     * @see com.zfl9.test1.Test#test()
     */
    private static final String MESSAGE = "www.zfl9.com";

    /**
     * 打印 MESSAGE 字符串.
     * {@code MESSAGE} 字符串是一个 {@code public static final} 变量
     * @see #MESSAGE
     */
    public void test() {
        out.println(MESSAGE);
    }

    /**
     * 打印一个指定的字符串.
     * @param str 要打印的字符串
     * @param newline 是否追加换行符{@code '\n'}
     */
    public void print(String str, boolean newline) {
        if (newline) {
            out.println(str);
        } else {
            out.print(str);
        }
    }
}
</script></code></pre>

**生成文档并部署**
<pre><code class="language-bash line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:54:26]
$ tree
.
└── com
    └── zfl9
        ├── test1
        │   ├── package-info.java
        │   └── Test.java
        └── test2
            ├── package-info.java
            └── Test.java

4 directories, 4 files

# root @ arch in ~/work on git:master x [10:54:28]
$ javadoc -locale zh_CN -d html -private -version -author -use -docencoding UTF-8 com.zfl9.test1 com.zfl9.test2
正在加载程序包com.zfl9.test1的源文件...
正在加载程序包com.zfl9.test2的源文件...
正在构造 Javadoc 信息...
正在创建目标目录: "html/"
标准 Doclet 版本 1.8.0_144
正在构建所有程序包和类的树...
正在生成html/com/zfl9/test1/Test.html...
正在生成html/com/zfl9/test2/Test.html...
正在生成html/overview-frame.html...
正在生成html/com/zfl9/test1/package-frame.html...
正在生成html/com/zfl9/test1/package-summary.html...
正在生成html/com/zfl9/test1/package-tree.html...
正在生成html/com/zfl9/test2/package-frame.html...
正在生成html/com/zfl9/test2/package-summary.html...
正在生成html/com/zfl9/test2/package-tree.html...
正在生成html/constant-values.html...
正在生成html/com/zfl9/test1/class-use/Test.html...
正在生成html/com/zfl9/test2/class-use/Test.html...
正在生成html/com/zfl9/test1/package-use.html...
正在生成html/com/zfl9/test2/package-use.html...
正在构建所有程序包和类的索引...
正在生成html/overview-tree.html...
正在生成html/index-all.html...
正在生成html/deprecated-list.html...
正在构建所有类的索引...
正在生成html/allclasses-frame.html...
正在生成html/allclasses-noframe.html...
正在生成html/index.html...
正在生成html/overview-summary.html...
正在生成html/help-doc.html...

# root @ arch in ~/work on git:master x [10:54:44]
$ ls html
allclasses-frame.html    com                   deprecated-list.html  index-all.html  overview-frame.html    overview-tree.html  script.js
allclasses-noframe.html  constant-values.html  help-doc.html         index.html      overview-summary.html  package-list        stylesheet.css

# root @ arch in ~/work on git:master x [10:54:52]
$ scp -P2333 -Cpr html/* root@192.168.0.100:nginx-html/java/
allclasses-frame.html                                                                                             100%  735   126.8KB/s   00:00
allclasses-noframe.html                                                                                           100%  695   114.4KB/s   00:00
package-tree.html                                                                                                 100% 4311   741.3KB/s   00:00
package-summary.html                                                                                              100% 5250   972.2KB/s   00:00
Test.html                                                                                                         100% 4112   949.7KB/s   00:00
package-frame.html                                                                                                100%  759   195.9KB/s   00:00
package-use.html                                                                                                  100% 3904   959.7KB/s   00:00
Test.html                                                                                                         100%   11KB 190.1KB/s   00:00
package-tree.html                                                                                                 100% 4311   890.4KB/s   00:00
package-summary.html                                                                                              100% 5250   930.9KB/s   00:00
Test.html                                                                                                         100% 4112     1.0MB/s   00:00
package-frame.html                                                                                                100%  759   183.9KB/s   00:00
package-use.html                                                                                                  100% 3904   745.7KB/s   00:00
Test.html                                                                                                         100%   11KB   2.4MB/s   00:00
constant-values.html                                                                                              100% 5347     1.1MB/s   00:00
deprecated-list.html                                                                                              100% 3524   817.8KB/s   00:00
help-doc.html                                                                                                     100% 8301   152.6KB/s   00:00
index-all.html                                                                                                    100% 6943   913.2KB/s   00:00
index.html                                                                                                        100% 2882   521.7KB/s   00:00
overview-frame.html                                                                                               100%  866   194.6KB/s   00:00
overview-summary.html                                                                                             100% 4145   910.5KB/s   00:00
overview-tree.html                                                                                                100% 4155   810.4KB/s   00:00
package-list                                                                                                      100%   30     7.2KB/s   00:00
script.js                                                                                                         100%  827   179.4KB/s   00:00
stylesheet.css                                                                                                    100%   13KB 216.2KB/s   00:00

# root @ home in ~/nginx-html/java [10:53:13]
$ pwd
/root/nginx-html/java

# root @ home in ~/nginx-html/java [10:55:14]
$ chown -R nginx.nginx *

# root @ home in ~/nginx-html/java [10:55:19]
$ ll
total 92K
-rw-r--r-- 1 nginx nginx  735 Sep 28 10:54 allclasses-frame.html
-rw-r--r-- 1 nginx nginx  695 Sep 28 10:54 allclasses-noframe.html
drwxr-xr-x 3 nginx nginx 4.0K Sep 28 10:54 com
-rw-r--r-- 1 nginx nginx 5.3K Sep 28 10:54 constant-values.html
-rw-r--r-- 1 nginx nginx 3.5K Sep 28 10:54 deprecated-list.html
-rw-r--r-- 1 nginx nginx 8.2K Sep 28 10:54 help-doc.html
-rw-r--r-- 1 nginx nginx 6.8K Sep 28 10:54 index-all.html
-rw-r--r-- 1 nginx nginx 2.9K Sep 28 10:54 index.html
-rw-r--r-- 1 nginx nginx  866 Sep 28 10:54 overview-frame.html
-rw-r--r-- 1 nginx nginx 4.1K Sep 28 10:54 overview-summary.html
-rw-r--r-- 1 nginx nginx 4.1K Sep 28 10:54 overview-tree.html
-rw-r--r-- 1 nginx nginx   30 Sep 28 10:54 package-list
-rw-r--r-- 1 nginx nginx  827 Sep 28 10:54 script.js
-rw-r--r-- 1 nginx nginx  13K Sep 28 10:54 stylesheet.css
</script></code></pre>



浏览一下效果：
![javadoc 效果 - 1](/images/java-javadoc-1.png)
![javadoc 效果 - 2](/images/java-javadoc-2.png)
![javadoc 效果 - 3](/images/java-javadoc-3.png)
![javadoc 效果 - 4](/images/java-javadoc-4.png)
