# 前言

# configuration - 根节点
三个属性，scan、scanPeriod 和 debug。
- scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true.
- scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
- debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

```
<configuration scan="true" scanPeriod="60 second" debug="false">  
	<!-- 其他配置省略 -->
</configuration>
```

# contextName - 设置上下文名称
每个logger都关联到logger上下文，默认上下文名称为“default”。
但可以使用contextName设置成其他名字，用于区分不同应用程序的记录。
一旦设置，不能修改。

```
<configuration scan="true" scanPeriod="60 second" debug="false"> 
    <contextName>myAppName</contextName> 
    <!-- 其他配置省略--> 
</configuration>
```

# property - 设置变量
两个属性，name 和 value。
- name：变量的名称
- value：变量的值

通过property定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

例如使用property定义上下文名称，然后在contentName设置logger上下文名称时使用：
```
<configuration scan="true" scanPeriod="60 second" debug="false">  
    <property name="APP_Name" value="myAppName" />   
	
    <contextName>${APP_Name}</contextName>  
    <!-- 其他配置省略-->  
</configuration>
```

# timestamp - 获取时间戳字符串
两个属性，key 和 datePattern。
- key：标识此timestamp 的名字
- datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式

例如将解析配置文件的时间作为上下文名称：
```
<configuration scan="true" scanPeriod="60 second" debug="false">  
    <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>   
    
	<contextName>${bySecond}</contextName>  
    <!-- 其他配置省略-->  
</configuration>
```

# appender - 定义输出形式
<appender>是<configuration>的子节点，是负责写日志的组件。<appender>有两个必要属性name和class。
- name：指定appender名称
- class：指定appender的全限定名。

## ConsoleAppender
把日志添加到控制台，有以下子节点：
- <encoder>：对日志进行格式化。（具体参数稍后讲解 ）
- <target>：字符串 System.out 或者 System.err ，默认 System.out
```
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>
        </encoder>
    </appender>
    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

## FileAppender
把日志添加到文件，有以下子节点：
- <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
- <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
- <encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）
- <prudent>：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。
```
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>testFile.log</file>        
        <append>true</append>
        <encoder>  
            <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="DEBUG">
          <appender-ref ref="FILE" />
    </root>
</configuration>
```

## RollingFIleAppender
滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：
- <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
- <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
- <encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）
- <rollingPolicy>:当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。
- <triggeringPolicy >: 告知 RollingFileAppender 何时激活滚动。
- <prudent>：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。

### rollingPolicy
**TimeBasedRollingPolicy**： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责触发滚动。有以下子节点：
- <fileNamePattern>: 必要节点，包含文件名及“%d”转换符，%d”可以包含一个Java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
- <maxHistory>: 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且<maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

**FixedWindowRollingPolicy**： 根据固定窗口算法重命名文件的滚动策略。有以下子节点：
- <minIndex>:窗口索引最小值。
- <maxIndex>:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
- <fileNamePattern >: 必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip

### triggeringPolicy
**SizeBasedTriggeringPolicy**： 查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:
- <maxFileSize>:这是活动文件的大小，默认值是10MB。

例如，每天生产一个日志文件，保存30天的日志文件：
```
<configuration>   
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>   
            <maxHistory>30</maxHistory>    
        </rollingPolicy>

        <encoder>   
            <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>   
        </encoder>   
    </appender>

    <root level="DEBUG">   
        <appender-ref ref="FILE" />   
    </root>
</configuration>
```

# <encoder>
负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。
目前PatternLayoutEncoder 是唯一有用的且默认的encoder ，有一个<pattern>节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“%”进行转义。例如：
```
<encoder>
    <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>   
</encoder>
```

# logger - 对包或类设置日志输出形式
用来设置某一个包或者某一个类的日志打印级别、以及指定appender。
三个属性name、level 和 additivity。
- name：用来指定受此logger约束的某一个包或者具体的某一个类。
- level：用来设置打印级别，如果未设置此属性，那么当前logger将会继承上级的级别。大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。
- additivity：是否向上级logger传递打印信息。默认是true，将此logger的打印信息向上级传递，。

logger可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个logger用于输出日志，没有设置appender-ref，则此logger本身不打印任何信息。

```
<configuration>
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
	
    <!-- logback为java中的包 -->
    <logger name="logback" />

    <!--logback.LogbackDemo：类的全路径 -->
    <logger name="logback.LogbackDemo" level="INFO" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>
</configuration>
```

# root - 全局设置日志输出形式
也是logger元素，且是根logger。

只有一个level属性。
- level：用来设置打印级别，默认是DEBUG。大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。

root可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个root用于输出日志。

```
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    
	<root level="ERROR">
        <appender-ref ref="STDOUT"/> 
    </root> 
</configuration>
```

# 其他
## 日志级别
用来设置打印级别，大小写无关。
由低到高：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。

## 格式 
%m 输出代码中指定的消息 
%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL 
%r 输出自应用启动到输出该log信息耗费的毫秒数 
%c 输出所属的类目，通常就是所在类的全名 
%t 输出产生该日志事件的线程名 
%n 输出一个回车换行符，Windows平台为"rn"，Unix平台为"n" 
%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921 
%l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(Test Log4.java:10)


# 完整配置案例
```
<?xml version="1.0" encoding="UTF-8"?>
<!--
-scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true
-scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
-           当scan为true时，此属性生效。默认的时间间隔为1分钟
-debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-
- configuration 子节点为 appender、logger、root
-->
<configuration scan="true" scanPeriod="60 second" debug="false">
 
    <!-- 负责写日志,控制台日志 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
 
        <!-- 一是把日志信息转换成字节数组,二是把字节数组写入到输出流 -->
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
 
    <!-- 文件日志 -->
    <appender name="DEBUG" class="ch.qos.logback.core.FileAppender">
        <file>debug.log</file>
        <!-- append: true,日志被追加到文件结尾; false,清空现存文件;默认是true -->
        <append>true</append>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- LevelFilter: 级别过滤器，根据日志级别进行过滤 -->
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
 
    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>info.log</File>
 
        <!-- ThresholdFilter:临界值过滤器，过滤掉 TRACE 和 DEBUG 级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
 
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
 
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天生成一个日志文件，保存30天的日志文件
            - 如果隔一段时间没有输出日志，前面过期的日志不会被删除，只有再重新打印日志的时候，会触发删除过期日志的操作。
            -->
            <fileNamePattern>info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender >
 
    <!--<!– 异常日志输出 –>-->
    <!--<appender name="EXCEPTION" class="ch.qos.logback.core.rolling.RollingFileAppender">-->
        <!--<file>exception.log</file>-->
        <!--<!– 求值过滤器，评估、鉴别日志是否符合指定条件. 需要额外的两个JAR包，commons-compiler.jar和janino.jar –>-->
        <!--<filter class="ch.qos.logback.core.filter.EvaluatorFilter">-->
            <!--<!– 默认为 ch.qos.logback.classic.boolex.JaninoEventEvaluator –>-->
            <!--<evaluator>-->
                <!--<!– 过滤掉所有日志消息中不包含"Exception"字符串的日志 –>-->
                <!--<expression>return message.contains("Exception");</expression>-->
            <!--</evaluator>-->
            <!--<OnMatch>ACCEPT</OnMatch>-->
            <!--<OnMismatch>DENY</OnMismatch>-->
        <!--</filter>-->
 
        <!--<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">-->
            <!--<!– 触发节点，按固定文件大小生成，超过5M，生成新的日志文件 –>-->
            <!--<maxFileSize>5MB</maxFileSize>-->
        <!--</triggeringPolicy>-->
    <!--</appender>-->
 
    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>error.log</file>
 
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
 
        <!-- 按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。
        -    窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。
        -    可以指定文件压缩选项
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>error.%d{yyyy-MM}(%i).log.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>3</maxIndex>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>
 
    <!-- 异步输出 -->
    <appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold >0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref ="ERROR"/>
    </appender>
 
    <!--
    - 1.name：包名或类名，用来指定受此logger约束的某一个包或者具体的某一个类
    - 2.未设置打印级别，所以继承他的上级<root>的日志级别“DEBUG”
    - 3.未设置additivity，默认为true，将此logger的打印信息向上级传递；
    - 4.未设置appender，此logger本身不打印任何信息，级别为“DEBUG”及大于“DEBUG”的日志信息传递给root，
    -  root接到下级传递的信息，交给已经配置好的名为“STDOUT”的appender处理，“STDOUT”appender将信息打印到控制台；
    -->
    <logger name="ch.qos.logback" />
 
    <!--
    - 1.将级别为“INFO”及大于“INFO”的日志信息交给此logger指定的名为“STDOUT”的appender处理，在控制台中打出日志，
    -   不再向次logger的上级 <logger name="logback"/> 传递打印信息
    - 2.level：设置打印级别（TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF），还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。
    -        如果未设置此属性，那么当前logger将会继承上级的级别。
    - 3.additivity：为false，表示此logger的打印信息不再向上级传递,如果设置为true，会打印两次
    - 4.appender-ref：指定了名字为"STDOUT"的appender。
    -->
    <logger name="com.weizhi.common.LogMain" level="INFO" additivity="false">
        <appender-ref ref="STDOUT"/>
        <!--<appender-ref ref="DEBUG"/>-->
        <!--<appender-ref ref="EXCEPTION"/>-->
        <!--<appender-ref ref="INFO"/>-->
        <!--<appender-ref ref="ERROR"/>-->
        <appender-ref ref="ASYNC"/>
    </logger>
 
    <!--
    - 根logger
    - level:设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。
    -       默认是DEBUG。
    -appender-ref:可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger
    -->
    <root level="DEBUG">
        <appender-ref ref="STDOUT"/>
        <!--<appender-ref ref="DEBUG"/>-->
        <!--<appender-ref ref="EXCEPTION"/>-->
        <!--<appender-ref ref="INFO"/>-->
        <appender-ref ref="ASYNC"/>
    </root>
</configuration>
```