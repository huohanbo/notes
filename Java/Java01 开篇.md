# Java 简介
Java是一门面向对象编程语言，不仅吸收了C++语言的各种优点，还摒弃了C++里难以理解的多继承、指针等概念，因此Java语言具有**功能强大**和**简单易用**两个特征。

Java语言作为静态面向对象编程语言的代表，极好地实现了面向对象理论，允许程序员以优雅的思维方式进行复杂的编程。

Java具有**简单性、面向对象、分布式、健壮性、安全性、平台独立与可移植性、多线程、动态性**等特点。

Java可以编写桌面应用程序、Web应用程序、分布式系统和嵌入式系统应用程序等。

# Java 发展历程 [参考地址](https://www.cnblogs.com/springmorning/p/10253223.html)
**1996年1月（1.0版本发布）**
JDK1.0版本发布，这个版本为JDK1.0.2

**1997年2月（1.1版本发布）**
JDK1.1版本发布。主要特点是：
- AWT 事件模型
- 内部类
- JavaBeans
- JDBC
- RMI
- 仅仅支持内省形式的反射，具体在java.beans包中实现

**1998年12月（1.2版本发布）**
JDK1.2版本发布，代号Playground。该版本通常被称为Java 2版本，是见证重大转变的最流行版本。主要特点是：
- 增加了strictfp 关键字
- Swing图形API
- Sun的JVM首次配备了JIT编译器
- Java插件技术：https://www.oracle.com/technetwork/java/index-jsp-141438.html
- 集合框架
- 支持windows系统的JIT编译器

**2000年5月（1.3版本发布）**
JDK1.3版本发布，代号Kestrel。主要特点是：
- Sun的JVM配备HotSpot JVM
- 支持Java命名与目录接口
- 支持Java平台调试体系
- JavaSound
- 支持代理类

**2002年2月（1.4版本发布）**
J2SE1.4版本发布，代号Merlin。主要特点是：
- 增加assert关键字
- 支持正则表达式
- 异常链
- 支持IPv6
- NIO
- 日志API
- Image I/O API
- 集成XML解析器和JAXP
- 集成JCE、JSSE、JAAS
- 支持Java Web Start
- Preferences API：java.util.prefs

**2004年9月（5.0版本发布）**
J2SE5.0发布，代号Tiger。主要特点是：
- 泛型
- 注解
- 自动装箱/拆箱
- 枚举
- 可变参数
- 增强for each循环
- 静态导入
- java.util.concurrent中新的并发实用程序
- Scanner类

**2006年12月（6.0版本发布）**
Java SE 6版本发布，代号Mustang。主要特点是：
- 支持脚本语言
- 性能上的提高
- JAX-WS
- JDBC 4.0
- JavaCompiler API
- JAXB 2.0 和 Streaming API for XML (StAX)
- 插件化注解处理API
- 新的GC算法

**2011年7月（7.0版本发布）**
Java SE 7.0版本发布，代号Dolphin。这个版本距上次发布有5年之久，并且只有这个版本花费了这么久。主要特点是：
- JVM支持动态语言
- 压缩的64位指针
- switch语句支持String
- try-with-resources
- <>操作符：https://www.javaworld.com/article/2074080/core-java/core-java-jdk-7-the-diamond-operator.html
- 简化可变参数方法声明
- 二进制整数字面值：https://docs.oracle.com/javase/7/docs/technotes/guides/language/binary-literals.html
- 允许下划线数字字面值：https://docs.oracle.com/javase/7/docs/technotes/guides/language/underscores-literals.html
- 异常处理优化：https://howtodoinjava.com/java7/improved-exception-handling/
- ForkJoin框架
- NIO2.0
- WatchService
- Timsort算法用于Collections.sort和Arrays.sort
- 图形功能API增强
- 支持SCTP和SDP这两种新的网络协议

**2014年3月（8.0版本发布）**
代号名字文化丢弃，主要特点是：
- 在API上支持Lambda表达式
- 函数接口和默认方法
- Optional
- 提供 Nashorn JavaScript引擎
- Annotation新特性：类型注解和重复注解
- 新的日期和时间API
- 支持静态链接JNI库
- 支持从jar文件启动JavaFX应用程序
- 从GC中移除永久代