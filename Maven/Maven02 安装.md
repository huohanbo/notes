# Maven 环境配置
**Maven 是一个基于 Java 的工具，所以要做的第一件事情就是安装 JDK。**
- Maven 3.3 要求 JDK 1.7 或以上
- Maven 3.2 要求 JDK 1.6 或以上
- Maven 3.0/3.1 要求 JDK 1.5 或以上

# Maven 下载
Maven 下载地址：http://maven.apache.org/download.cgi

不同平台下载对应的包：
- Windows:	apache-maven-3.3.9-bin.zip
- Linux:	apache-maven-3.3.9-bin.tar.gz
- Mac:		apache-maven-3.3.9-bin.tar.gz

# 设置 Maven 环境变量
新建系统变量 MAVEN_HOME，变量值：E:\Maven\apache-maven-3.3.9

编辑系统变量 Path，添加变量值：;%MAVEN_HOME%\bin