# 参考资料
+ [Maven官网](https://maven.apache.org/index.html)
+ [Maven下载](https://maven.apache.org/download.cgi)
+ [Maven用户中心](https://maven.apache.org/users/index.html)

# Maven 简介
Maven 是一个项目管理工具，可以对 Java 项目进行构建和依赖管理。

# Maven 特性
Maven的主要目标是允许开发人员在最短的时间内理解开发工作的完整状态。
为了实现此目标，Maven尝试解决以下几个方面的问题：
+ 简化构建过程
+ 提供统一的构建系统
+ 提供优质的项目信息
+ 提供最佳实践开发指南
+ 允许透明迁移到新功能

## 简化构建过程
虽然使用Maven并不能消除对底层机制的了解，但Maven确实为细节提供了很多保护。

## 提供统一的构建系统
Maven允许使用其项目对象模型（POM）和一组由Maven共享的插件来构建项目，从而提供统一的构建系统。
一旦熟悉了一个Maven项目的构建方式，您就会自动知道所有Maven项目的构建方式，从而在尝试浏览多个项目时为您节省了大量时间。

## 提供优质的项目信息
Maven提供了大量有用的项目信息，这些信息部分来自您的POM，部分来自项目源。例如，Maven可以提供：
+ 更改直接从源代码管理创建的日志文档
+ 交叉引用来源
+ 项目管理的邮件列表清单
+ 依赖清单
+ 单元测试报告，包括覆盖率

随着Maven的改进，所提供的信息集将得到改善，所有这些信息对Maven的用户都是透明的。

其他产品还可以提供Maven插件，以允许其项目信息集以及Maven给出的一些标准信息，而这些信息仍然基于POM。

## 提供最佳实践开发指南
Maven的目的是收集当前最佳实践开发的原则，并使其易于朝着这个方向指导项目。

例如，单元测试的规范，执行和报告是使用Maven的常规构建周期的一部分。当前的单元测试最佳实践被用作准则：
+ 将测试源代码保存在单独但并行的源树中
+ 使用测试用例命名约定来定位和执行测试
+ 让测试用例设置其环境，而不是依赖于自定义构建以进行测试准备

Maven还旨在协助项目工作流程，例如发布和问题管理。

Maven还建议了一些有关如何布局项目目录结构的准则。了解布局之后，您可以轻松导航任何其他的使用Maven和相同默认值的项目。

## 允许透明迁移到新功能
Maven为Maven客户端提供了一种轻松的方式来更新其安装，以便他们可以利用对Maven本身所做的任何更改。

因此，从第三方或Maven本身安装新的或更新的插件变得很简单。

# Maven 功能
+ 遵循最佳实践的简单项目设置-数秒内即可启动新项目或模块
+ 在所有项目中使用一致-意味着新开发者无需花更多时间来参与项目
+ 高级依赖项管理，包括自动更新，依赖项关闭（也称为传递依赖项）
+ 能够轻松同时处理多个项目
+ 开箱即用的庞大且不断增长的库和元数据存储库，以及与大型开放源代码项目的安排，可实时获取其最新版本
+ 可扩展，能够轻松用 Java或脚本语言编写插件
+ 几乎无需额外配置即可立即访问新功能
+ 用于Maven之外的依赖项管理和部署的Ant任务
+ 基于模型的构建：Maven能够将任何数量的项目构建为预定义的输出类型，例如JAR，WAR或基于有关该项目的元数据的分发，而在大多数情况下无需执行任何脚本。
+ 项目信息的一致站点：使用与构建过程相同的元数据，Maven可以生成一个网站或PDF，其中包括您希望添加的任何文档，并将有关项目开发状态的标准报告添加到该标准报告中。该信息的示例可以在本网站左侧导航的底部的“项目信息”和“项目报告”子菜单下看到。
+ 发布管理和发行发布：无需太多额外配置，Maven将与您的源代码控制系统（例如Subversion或Git）集成，并基于特定标签管理项目的发布。它还可以将其发布到分发位置，以供其他项目使用。Maven能够发布单个输出，例如JAR，包含其他依赖项和文档的存档或作为源分发。
+ 依赖关系管理：Maven鼓励使用JAR和其他依赖关系的中央存储库。Maven带有一种机制，您的项目的客户端可以使用该机制从中央JAR存储库中下载构建项目所需的任何JAR，就像Perl的CPAN一样。这使Maven的用户可以在项目之间重用JAR，并鼓励项目之间进行通信，以确保解决向后兼容性问题。

# Maven 使用
## Maven下载
下载地址：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

系统要求：Maven 3.3+需要JDK 1.7或更高版本才能执行。
- Maven 3.3 要求 JDK 1.7 或以上
- Maven 3.2 要求 JDK 1.6 或以上
- Maven 3.0/3.1 要求 JDK 1.5 或以上

## Maven安装
### 解压
Unix：
```
unzip apache-maven-3.6.3-bin.zip
or
tar xzvf apache-maven-3.6.3-bin.tar.gz
```

### 配置环境变量
Windows：
```
新建系统变量 MAVEN_HOME，变量值：
E:\Maven\apache-maven-3.3.9

编辑系统变量 Path，添加变量值：
;%MAVEN_HOME%\bin
```

Unix：
```
export PATH=/opt/apache-maven-3.6.3/bin:$PATH
```

### 验证
```
mvn -v
```

## Maven运行
运行Maven的语法如下：
```
mvn [options] [<goal(s)>] [<phase(s)>]

mvn [ 选项] [< 目标（s ）>] [< 阶段（s ）>]  
```

所有可用选项都记录在内置的帮助中：
```
mvn -h
```

使用Maven生命周期阶段构建Maven项目的典型调用：
```
mvn package
```

可以使用以下方法重新生成项目，即生成所有打包的输出和文档站点，并将其部署到存储库管理器:
```
mvn clean deploy site-deploy
```

只需创建软件包并将其安装在本地存储库中以供其他项目重用：
```
mvn verify
```

### 内置生命周期
```
clean - pre-clean, clean, post-clean
default - validate, initialize, generate-sources, process-sources, generate-resources, process-resources, compile, process-classes, generate-test-sources, process-test-sources, generate-test-resources, process-test-resources, test-compile, process-test-classes, test, prepare-package, package, pre-integration-test, integration-test, post-integration-test, verify, install, deploy
site - pre-site, site, post-site, site-deploy
```

```
清洁-预清洁，清洁，后清洁
默认值-验证，初始化，生成源，流程源，生成资源，流程资源，编译，流程类，生成测试源，流程测试源，生成测试资源，流程测试资源，测试编译，过程测试类，测试，准备打包，打包，集成前测试，集成测试，集成后测试，验证，安装，部署
站点-站点前，站点，后站点，站点部署
```

# Maven 配置
## MAVEN_OPTS 环境变量
此变量包含用于启动运行Maven的JVM的参数，并可用于为其提供其他选项。例如，可以使用值定义JVM内存设置-Xms256m -Xmx512m。

## settings.xml 文件
设置文件位于USER_HOME / .m2中，旨在包含跨项目使用Maven的任何配置。

# Maven 快速开始
## 创建项目
您将需要将项目放置在某个地方，在某个地方创建目录，然后在该目录中启动Shell。在命令行上，执行以下Maven指令：
```
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```
```
mvn archetype:generate 
-DgroupId=com.mycompany.app 
-DartifactId=my-app 
-DarchetypeArtifactId=maven-archetype-quickstart 
-DarchetypeVersion=1.4 
-DinteractiveMode=false
```
该指令生成了一个目录 my-app，该目录具有与artifactId相同的名称。

转到该目录，标准项目结构如下：
```
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```
目录src/main/java包含项目源代码，目录src/test/java包含测试源，pom.xml文件是项目的项目对象模型。

pom.xml 文件是Maven中项目配置的核心。它是一个配置文件，其中包含以所需方式构建项目所需的大多数信息。
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
 
  <properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

## 编译项目
```
mvn package
```

结果反馈：
```
 ...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.953 s
[INFO] Finished at: 2019-11-24T13:05:10+01:00
[INFO] ------------------------------------------------------------------------
```

## Maven工具
### Maven阶段
尽管这不是一个全面的列表，但它们是执行的最常见的默认生命周期阶段。
validate 验证：验证项目是否正确并且所有必要的信息均可用
compile 编译：编译项目的源代码
test 测试：使用合适的单元测试框架测试编译后的源代码。这些测试不应要求将代码打包或部署
package 打包：采用编译后的代码并将其打包为可分发格式，例如JAR。
integration-test 集成测试：如有必要，将程序包处理并部署到可以运行集成测试的环境中
verify 验证：运行任何检查以验证包装是否有效并符合质量标准
install 安装：将软件包安装到本地存储库中，以作为本地其他项目中的依赖项
deploy 部署：在集成或发布环境中完成，将最终软件包复制到远程存储库，以便与其他开发人员和项目共享。

除了上面的默认列表以外，还有其他两个Maven生命周期值得注意。
clean 清理：清理由先前版本创建的工件
site 站点：为此项目生成站点文档

**阶段实际上映射到基本目标。每个阶段执行的特定目标取决于项目的包装类型。**
例如，如果项目类型是JAR，则package将执行jar；如果项目类型是WAR ，则package将执行war。

### 生成站点
```
mvn site
```
此阶段根据有关项目pom的信息生成一个站点。您可以在 target/code下查看生成的文档。

## Java 9或更高版本
默认情况下，您的Maven版本可能使用的旧版本maven-compiler-plugin与Java 9或更高版本不兼容。
要定位Java 9或更高版本，您至少应使用的3.6.0版本maven-compiler-plugin，并将maven.compiler.release属性设置为您要定位的Java版本（例如9、10、11、12等）。

在以下示例中，我们将Maven项目配置为使用3.8.1的版本maven-compiler-plugin并针对Java 11：
```
    <properties>
        <maven.compiler.release>11</maven.compiler.release>
    </properties>
 
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```

# Maven 生命周期
## 生命周期简介
Maven基于构建生命周期，这意味着构建和分发特定项目的过程是明确的。

对于构建项目的人员来说，这意味着只需要学习一小部分命令就可以构建任何Maven项目，而POM将确保他们得到想要的结果。

Maven有三个内置的构建生命周期：default, clean, site。default负责项目部署，clean负责项目清理，site负责创建项目文档。

Lifecycle：生命周期，这是maven最高级别的的控制单元，它是一系列的phase组成，也就是说，一个生命周期，就是一个大任务的总称。
可以定义自己的lifecycle，包含自己想要的phase。

Phase：任务单元，lifecycle是总任务，phase就是总任务分出来的一个个子任务。
但是这些子任务是被规格化的，它可以同时被多个lifecycle所包含，一个lifecycle可以包含任意个phase，phase的执行是按顺序的。
一个phase可以绑定很多个goal，至少为一个，没有goal的phase是没有意义的。

Goal: 最小单元，它可以绑定到任意个phase中，一个phase有一个或多个goal。
goal也是按顺序执行的，一个phase被执行时，绑定到phase里的goal会按绑定的时间被顺序执行。
不管phase己经绑定了多少个goal，自己定义的goal都可以继续绑到phase中。

### 生命周期(Lifecycle)由阶段(Phases)组成
每个生命周期都由不同的阶段组成。
例如，默认生命周期由以下阶段组成：
+ validate-验证项目是正确的，并提供所有必要的信息
+ compile-编译项目的源代码
+ test-使用适当的单元测试框架测试编译的源代码。这些测试不应要求对代码进行打包或部署
+ package-将编译后的代码打包成可发行的格式，例如JAR
+ verify-对集成测试的结果进行检查，以确保符合质量标准
+ install-将包安装到本地存储库，以便在本地其他项目中用作依赖项
+ deploy-在构建环境中完成，将最终包复制到远程存储库，以便与其他开发人员和项目共享

这些生命周期阶段是按顺序执行的。

### 阶段(Phases)由目标(Plugin Goal)组成
尽管 阶段 是负责构建生命周期中的特定步骤，但它执行这些职责的方式可能有所不同。
这是通过声明与这些阶段绑定的 插件目标 来实现的。

目标代表一个特定的任务(比阶段更精细)，它有助于项目的构建和管理。
它可能被绑定到零或多个构建阶段。不绑定到任何构建阶段的目标可以在构建生命周期之外通过**直接调用**执行。
执行顺序取决于调用 阶段 和 目标 的顺序。
如下面的命令，clean和package参数是阶段，而dependency:copy-dependencies是目标。
```
mvn clean dependency:copy-dependencies package
```
执行此操作，将首先执行**clean阶段**(这意味着它将运行清洁生命周期的所有前几个阶段，加上clean本身)，
然后**dependency:copy-dependencies目标**，最终执行**package阶段**(以及之前默认生命周期的所有构建阶段)。

阶段可以绑定零或更多的目标。如果构建阶段没有绑定到目标，则该阶段将不会执行，如果它有一个或多个目标，它将执行所有这些目标。
如果一个目标被绑定到一个或多个阶段，那么该目标将在所有这些阶段中被调用。

### 常用命令行调用
根据目的选择合适的阶段。

如果想要打包，运行package。

如果想要打包，运行test。

若在开发环境中：
```
mvn clean deploy
```

## 使用生命周期设置项目
使用Maven构建项目时，如何将任务分配给每个构建阶段？

### 打包
第一种也是最常见的方法是通过同名的POM元素为项目设置包装<packaging>。
一些有效的包装价值的是jar，war，ear和pom。如果未指定包装值，则默认为jar。

### 插件
向阶段添加目标的第二种方法是在项目中配置插件。插件是为Maven提供目标的工件。
此外，一个插件可能有一个或多个目标，其中每个目标代表该插件的能力。
例如，编译器插件有两个目标：compile和testCompile。前者编译主代码的源代码，后者编译测试代码的源代码。

## 生命周期参考
### Clean 生命周期
```
阶段			描述
pre-clean	在实际项目清理之前执行所需的过程
clean		删除以前的版本生成的所有文件
post-clean	执行完成项目清理所需的过程
```

### Default 生命周期
```
阶段			描述
validate	验证项目是否正确以及所有必要的信息均可用。
initialize	初始化构建状态，例如设置属性或创建目录。
generate-sources	生成任何要包含在编译中的源代码。
process-sources	处理源代码，例如过滤任何值。
generate-resources	生成资源以包含在包中。
process-resources	将资源复制并处理到目标目录中，以备打包。
compile	编译项目的源代码。
process-classes	对编译后生成的文件进行后处理，例如对Java类进行字节码增强。
generate-test-sources	生成任何测试源代码以包含在编译中。
process-test-sources	处理测试源代码，例如过滤所有值。
generate-test-resources	创建测试资源。
process-test-resources	将资源复制并处理到测试目标目录中。
test-compile	将测试源代码编译到测试目标目录中
process-test-classes	对测试编译生成的文件进行后处理，例如对Java类进行字节码增强。
test	使用合适的单元测试框架运行测试。这些测试不应要求将代码打包或部署。
prepare-package	在实际包装之前执行准备包装所需的任何操作。这通常会导致包装的未包装，已处理版本。
package	获取编译后的代码，并将其打包为可分发格式，例如JAR。
pre-integration-test	在执行集成测试之前执行所需的操作。这可能涉及诸如设置所需环境的事情。
integration-test	处理该程序包，并在必要时将其部署到可以运行集成测试的环境中。
post-integration-test	在执行集成测试后执行所需的操作。这可能包括清理环境。
verify	运行任何检查以验证包装是否有效并符合质量标准。
install	将软件包安装到本地存储库中，以作为本地其他项目中的依赖项。
deploy	在集成或发布环境中完成后，将最终程序包复制到远程存储库，以便与其他开发人员和项目共享。
```

### Site 生命周期
```
阶段			描述
pre-site	在实际项目站点生成之前执行所需的过程
site		生成项目的站点文档
post-site	执行完成站点生成并为站点部署做准备所需的过程
site-deploy	将生成的站点文档部署到指定的Web服务器
```

## 内置生命周期绑定
默认情况下，某些阶段的目标已绑定。对于默认生命周期，这些绑定取决于包装的价值。这是一些目标到构建阶段的绑定。
...

# Maven POM
## POM简介
POM是Maven中的基本工作单元。这是一个XML文件，其中包含有关项目的信息以及Maven用于构建项目的配置详细信息。它包含大多数项目的默认值。

当执行任务或目标时，Maven在当前目录中查找POM。它读取POM，获取所需的配置信息，然后执行目标。

可以在POM中指定的一些配置，例如项目依赖项，可以执行的插件或目标，构建配置文件等等。
也可以指定其他信息，例如项目版本，描述，开发人员，邮件列表等。

## 超级POM
超级POM是Maven的**默认POM**。
除非明确设置，否则所有POM都会扩展Super POM，这意味着Super POM中指定的配置将由您为项目创建的POM继承。

超级POM预览：
```
<project>
  <modelVersion>4.0.0</modelVersion>
 
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
 
  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
 
  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.5.3</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
 
  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>
 
  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>
 
      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>
 
      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar-no-fork</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
 
</project>
```

## 最小POM
POM的最低要求如下：
+ 项目根 root
+ modelVersion-应设置为4.0.0
+ groupId-项目组的ID。
+ artifactId-项目的ID
+ version-项目的版本

例子：
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

## 项目继承!!!
合并的POM中的元素如下：
+ 依存关系
+ 开发者和贡献者
+ 插件列表（包括报告）
+ 具有匹配ID的插件执行
+ 插件配置
+ 资源

### 示例1
创建一个新项目 com.mycompany.app:my-module，作为项目com.mycompany.app:my-app的子项目。

指定其目录结构：
```
.
 |-- my-module
 |   `-- pom.xml
 `-- pom.xml
```
my-module/pom.xml是 com.mycompany.app:my-module 的POM，pom.xml是 com.mycompany.app:my-app 的POM。

my-module/pom.xml：
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```

修改my-module/pom.xml添加parent节点：
```
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
  </parent>
  
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```
通过此设置，我们的模块现在可以继承父POM的某些属性。

如果我们希望groupId和version与其父模块相同，则可以在其POM中删除groupId和version：
```
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```


### 示例2
如果父项目已经安装在我们的本地存储库中或位于特定目录结构中（父pom.xml目录比模块pom.xml目录高一个层级），pom.xml将起作用。

如果尚未安装父项目，并且目录结构如以下示例所示，该如何处理：
```
.
 |-- my-module
 |   `-- pom.xml
 `-- parent
     `-- pom.xml
```

为了解决这种目录结构（或任何其他目录结构），我们必须将<relativePath>元素添加到父节中：
```
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```
顾名思义，这是从 模块pom.xml 到 父级pom.xml 的相对路径。

## 项目聚合
项目聚合类似于项目继承。但不是从模块中指定父POM，而是从父POM中指定模块。
若要进行项目聚合，必须执行以下操作：
+ 将父POM的 packaging 设为 pom。
+ 在父POM中指定其模块的目录（子POM）。

### 示例3
给定原始项目POM和目录结构：
my-app
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```
my-module
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```
目录结构
```
.
 |-- my-module
 |   `-- pom.xml
 `-- pom.xml
```

将my-module聚合到my-app中，则只需修改my-app：
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>my-module</module>
  </modules>
</project>
```
在修改my-app时，添加了 packaging 部分和 modules 部分。

现在，每当Maven命令处理com.mycompany.app:my-app:1时，同样的Maven命令也将针对com.mycompany.app:my-module:1运行。

### 示例4
如果我们将目录结构更改为以下内容：
```
.
 |-- my-module
 |   `-- pom.xml
 `-- parent
     `-- pom.xml
```

通过指定模块的路径，与示例3相同：
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>../my-module</module>
  </modules>
</project>
```

## 项目继承与项目聚合
可以同时具有项目继承和项目聚合。
就是说，您可以让您的模块指定一个父项目，同时让该父项目将那些Maven项目指定为其模块。
您只需要应用所有三个规则：
+ 在每个子POM中指定其父POM是谁。
+ 将父POM的 packaging 更改为值 pom 。
+ 在父POM中指定其模块的目录（子POM）。

### 示例5
给定原始项目POM和目录结构：
my-app
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```
my-module
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```
目录结构
```
.
 |-- my-module
 |   `-- pom.xml
 `-- parent
     `-- pom.xml
```

同时进行项目继承和聚合：
my-app
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>../my-module</module>
  </modules>
</project>
```
my-module
```
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```

## 项目变量!!!
在某些情况下，您需要在几个不同的位置使用相同的值。为了帮助确保仅指定一次该值，Maven允许您在POM中使用您自己的变量和预定义的变量。

要访问project.version变量，您可以这样引用它：
```
<version>${project.version}</version>
```

### 可用变量
模型的任何值为单个值元素的字段都可以引用为变量。例如${project.groupId}，${project.version}，${project.build.sourceDirectory}等等。

### 特殊变量
```
project.basedir	当前项目所在的目录。
project.baseUri	当前项目所在的目录，以URI表示。从Maven 2.1.0开始
maven.build.timestamp	表示构建开始（UTC）的时间戳。从Maven 2.1.0-M1开始
```

### 自定义变量
properties
```
<project>
  ...
  <properties>
    <mavenVersion>3.0</mavenVersion>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-artifact</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-core</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
  </dependencies>
  ...
```

# Maven 配置文件 or 环境变量
...

# Maven 目录结构
通用的目录布局将使熟悉一个Maven项目的用户立即在另一个Maven项目中感到宾至如归。

Maven期望的目录布局：
```
src/main/java		应用程序/库资源
src/main/resources	应用/图书馆资源
src/main/filters	资源过滤器文件
src/main/webapp		Web应用程序源
src/test/java		测试来源
src/test/resources	测试资源
src/test/filters	测试资源过滤器文件
src/it				集成测试（主要用于插件）
src/assembly		程序集描述符
src/site			现场
LICENSE.txt			项目许可证
NOTICE.txt			项目所依赖的图书馆要求的注意事项和出处
README.txt			项目的自述文件
```
请尝试尽可能符合此结构，如果不能，则可以通过项目描述符覆盖这些设置。

顶层目录，有描述项目的文件 pom.xml，此外还有文本类的文档README.txt，LICENSE.txt等。
	该目录有两个主要的子目录：src和target，其他目录则是元数据，例如CVS，.git或.svn，以及多项目版本中的任何子项目

该src目录包含用于构建项目的所有源材料，其站点等。
	它包含每种类型的子目录：main用于主构建工件，test用于单元测试代码和资源，site等。
	在生成项目的源目录（即main和test）中，有一种语言目录java（在该目录下，存在正常的包层次结构），另一种目录下resources（将结构复制到给定默认资源定义的目标类路径中）。
	如果有其他有助于项目构建的源，它们将位于其他子目录下：例如，src/main/antlr将包含Antlr语法定义文件。

target目录用于容纳构建的所有输出。

# Maven 依赖机制
依赖管理是Maven的核心功能。

## 依赖传递
Maven通过自动包含传递性依赖关系，避免了指定自己的依赖关系所需的库的需求。

### 依赖中介
这决定了当遇到多个版本作为依赖项时，将选择哪个版本的工件。Maven选择“最近的定义”。
也就是说，它使用依赖树中与项目最接近的依赖项的版本。始终可以通过在项目的POM中显式声明版本来保证版本。
请注意，如果两个依赖关系版本在依赖树中处于相同的深度，则第一个声明将获胜。
	
“最近的定义”意味着所使用的版本将是与依赖树中的项目最接近的版本。
例如，如果A、B和C的依赖关系定义为A->B->C->D2.0和A->E->D1.0，则在构建A时将使用D1.0，因为从A到D到E的路径更短。
您可以在A中显式地向D2.0添加依赖项，以强制使用D2.0。

### 依赖管理
这允许项目作者直接指定在传递依赖项或未指定版本的依赖项中遇到的工件版本。
在上一节中的示例中，一个依赖项直接添加到A中，尽管A没有直接使用它。
相反，A可以将D作为依赖项包含在它的依赖项管理部分中，并直接控制在引用它时或如果引用它时使用哪个版本的D。

### 依赖范围
这允许您只包含适合于当前构建阶段的依赖项。

### 排除依赖
如果项目X依赖项目Y，项目Y依赖项目Z，则项目X的所有者可以使用“排除”元素明确排除项目Z为依赖项。

### 可选依赖关系
如果项目Y依赖项目Z，项目Y的所有者可以使用“可选”元素将项目Z标记为可选依赖项。
当项目X依赖于项目Y时，X将只依赖于Y，而不依赖于Y的可选依赖项Z。
然后项目X的所有者可以根据她的选择显式地添加对Z的依赖。(将可选依赖项视为“默认排除的”可能会有所帮助。)


虽然传递依赖项可以隐式地包含所需的依赖项，但是在您自己的源代码中显式指定直接使用的依赖项是一种很好的做法。
这个最佳实践证明了它的价值，特别是当您的项目的依赖项更改其依赖关系时。
例如，假设您的项目A指定了对另一个项目B的依赖关系，项目B指定了对项目C的依赖关系。
如果您直接使用项目C中的组件，并且您没有在项目A中指定项目C，则当项目B突然更新/移除其对项目C的依赖时，它可能会导致构建失败。

直接指定依赖项的另一个原因是它为您的项目提供了更好的文档：您可以通过在项目中读取POM文件来了解更多信息。

## 依赖范围
依赖关系范围用于限制依赖关系的可传递性，并且还影响用于各种构建任务的类路径

### compile 编译
这是默认范围，如果未指定范围，则使用此范围。编译依赖项在项目的所有类路径中均可用。此外，这些依赖项会传播到相关项目。

### provided 提供
类似于compile，但是表明您希望JDK或容器在运行时提供依赖项。例如，在为Java Enterprise Edition构建Web应用程序时，您将对Servlet API和相关Java EE API的依赖关系设置为范围，provided因为Web容器提供了这些类。该作用域仅在编译和测试类路径上可用，并且不可传递。

### runtime 运行
此作用域表明依赖关系不是编译所必需的，而是执行所必需的。它在运行时和测试类路径中，但不在编译类路径中。

### test 测试
此范围表明依赖关系对于正常使用应用程序不是必需的，并且仅在测试编译和执行阶段可用。此范围不是可传递的。

### system 系统
此范围类似于，provided除了必须提供显式包含它的JAR之外。该工件始终可用，并且不会在存储库中查找。

### import 引入
仅pom在本<dependencyManagement>节中的类型依赖项上支持此作用域。它指示要在指定的POM <dependencyManagement>部分中用有效的依赖关系列表替换的依赖关系。由于已替换它们，因此范围为的依赖项import实际上不会参与限制依赖项的可传递性。

## 依赖管理

## 系统依赖

## 可选依赖!!!
当无法将项目拆分为子模块时，将使用可选依赖项。
某些依赖项仅用于项目中的某些功能，如果不使用该功能，则不需要。

理想情况下，将此类功能拆分为取决于核心功能项目的子模块。
但是，由于无法拆分项目（同样由于某种原因），因此将这些依赖项声明为可选。
如果用户要使用与可选依赖项相关的功能，则必须在自己的项目中重新声明该可选依赖项。

### 使用可选标签
通过将依赖项中的<optional>元素设置为true，可以将依赖项声明为可选：
```
<project>
  ...
  <dependencies>
    <!-- declare the dependency to be set as optional -->
    <dependency>
      <groupId>sample.ProjectA</groupId>
      <artifactId>Project-A</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <optional>true</optional> <!-- value will be true or false only -->
    </dependency>
  </dependencies>
</project>
```

示例：
假设有一个名为X2的项目，其功能与Hibernate相似。它支持许多数据库，例如MySQL，PostgreSQL和Oracle的多个版本。
每个受支持的数据库都需要对驱动程序jar的其他依赖性。在构建X2时需要所有这些依赖项。
但是，您的项目仅使用一个特定的数据库，而不需要其他数据库的驱动程序。
X2可以将这些依赖项声明为可选，因此，当您的项目在其POM中将X2声明为直接依赖项时，X2支持的所有驱动程序都不会自动包含在项目的类路径中。
您的项目必须对它确实使用的一个数据库包含对特定驱动程序的显式依赖。

## 依赖排除!!!
由于Maven可传递地解决依赖关系，因此可能会将不需要的依赖关系包含在项目的类路径中。
例如，某个较旧的jar可能存在安全问题或与您使用的Java版本不兼容。为了解决这个问题，Maven允许您排除特定的依赖关系。

### 使用依赖项排除
在<dependency>元素中添加一个<exclusions>元素，设置需要排除的jar。
```
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>sample.ProjectA</groupId>
      <artifactId>Project-A</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <exclusions>
        <exclusion>  <!-- declare the exclusion here -->
          <groupId>sample.ProjectB</groupId>
          <artifactId>Project-B</artifactId>
        </exclusion>
      </exclusions> 
    </dependency>
  </dependencies>
</project>
```

示例：
```
Project-A
   -> Project-B
        -> Project-D <! -- This dependency should be excluded -->
              -> Project-E
              -> Project-F
   -> Project C
```

假设A不希望将项目D及其依赖项添加到项目A的类路径中，而项目B也没有设置对项目D的可选依赖，只能在A中使用排除依赖：
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>sample.ProjectA</groupId>
  <artifactId>Project-A</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
  ...
  <dependencies>
    <dependency>
      <groupId>sample.ProjectB</groupId>
      <artifactId>Project-B</artifactId>
      <version>1.0-SNAPSHOT</version>
      <exclusions>
        <exclusion>
          <groupId>sample.ProjectD</groupId> <!-- Exclude Project-D from Project-B -->
          <artifactId>Project-D</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
</project>
```

排除依赖一般是最后的选择，所依赖的配置没有设置可选依赖时，才会用到排除依赖。

# Maven Settings
## Settings简介
settings.xml文件可能存在两个位置：
+ Maven安装： ${maven.home}/conf/settings.xml
+ 用户安装： ${user.home}/.m2/settings.xml
前者settings.xml也称为全局设置，后者settings.xml称为用户设置。如果两个文件都存在，则它们的内容将合并，其中以用户特定settings.xml为主导。

## Settings详情
### 基础配置
```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>${user.home}/.m2/repository</localRepository>
  <interactiveMode>true</interactiveMode>
  <offline>false</offline>
  ...
</settings>
```
LocalRepository: 此值是本地存储库的路径。默认值是${user.home}/.m2/repository。此元素对于主构建服务器特别有用，它允许所有登录用户从公共本地存储库构建。
interactiveMode: true表示Maven尝试与用户交互输入。默认为true。
offline: true表示系统应在脱机模式下运行。默认为false。此元素对于无法连接到远程存储库的构建服务器非常有用，因为网络设置或安全原因。

### 插件组 Plugin Groups

### 服务器 Servers

### 镜像 Mirrors

### 代理 Proxies
...

# Maven POM
## POM简介
POM 全称 "Project Object Model"，即项目对象模型，在项目中体现为pom.xml文件。

POM包含配置文件，以及所涉及的开发人员及其所扮演的角色，缺陷跟踪系统，组织和许可证，
项目所在的URL，项目的依存关系等。

##  

# Maven 常见问题
## 如何创建Maven项目
```
mvn -B archetype:generate \
  -DarchetypeGroupId=org.apache.maven.archetypes \
  -DgroupId=com.mycompany.app \
  -DartifactId=my-app
```

## 如何编译项目
```
mvn compile
```

## 如何编译测试源并运行单元测试
```
mvn test
```

## 如何创建JAR并将其安装在本地存储库中
制作JAR文件非常简单，可以通过执行以下命令来完成：
```
mvn package
```

继续安装：
```
mvn install
```

## 什么是SNAPSHOT版本
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
  ...
  <groupId>...</groupId>
  <artifactId>my-app</artifactId>
  ...
  <version>1.0-SNAPSHOT</version>
  <name>Maven Quick Start Archetype</name>
  ...
```

SNAPSHOT值是指开发分支上的“最新”代码，不能保证该代码稳定或不变。
相反，“发行”版本（任何不带后缀的版本值SNAPSHOT）中的代码不变。
换句话说，SNAPSHOT版本是最终“发行”版本之前的“开发”版本。SNAPSHOT比其发行版要“高”。

在发行过程中，xy-SNAPSHOT的版本更改为xy。发布过程还将开发版本增加到 x.(y+1)-SNAPSHOT。
例如，版本1.0-SNAPSHOT被发布为版本1.0，而新的开发版本是版本1.1-SNAPSHOT。

......


