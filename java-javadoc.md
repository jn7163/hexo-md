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
`-charset <charset>`：指定生成的 HTML 文档的字符集

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
