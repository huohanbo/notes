# 参考资料
[Logstash Reference](https://www.elastic.co/guide/en/logstash/current/index.html)

# Logstash简介
Logstash是具有实时流水线功能的开源数据收集引擎。

Logstash可以动态统一来自不同来源的数据，并将数据规范化为您选择的目标。
清除所有数据并使其民主化，以用于各种高级下游分析和可视化用例。

虽然Logstash最初推动了日志收集方面的创新，但其功能远远超出了该用例。
任何类型的事件都可以通过各种各样的输入，过滤器和输出插件来丰富和转换，许多本机编解码器进一步简化了提取过程。
Logstash通过利用大量数据和各种数据来加快您的见解。

## Logstash优势
+ Elasticsearch等的摄取主力
具有强大的Elasticsearch和Kibana协同功能的水平可扩展数据处理管道
+ 可插拔管线架构
混合，匹配和编排不同的输入，滤波器和输出，以协调管道
+ 可扩展社区且对开发人员友好的插件生态系统
超过200个可用的插件，以及创建和贡献自己的灵活性

## Logstash善于收集数据
收集更多，以便您了解更多。Logstash 可以收集各种类型和大小的数据。

### 日志和指标
+ 处理所有类型的日志数据
	+ 轻松获取大量Web日志（如Apache）和应用程序日志（如log4j for Java）
	+ 捕获许多其他日志格式，例如syslog，网络和防火墙日志等
+ 通过Filebeat享受补充的安全日志转发功能
+ 通过TCP和UDP 从Ganglia，collectd， NetFlow，JMX以及许多其他基础结构和应用程序平台收集度量

### 网络
+ 将HTTP请求转换为事件
	+ 从Twitter之类的网络服务中消费，以进行社会情感分析
	+ Webhook对GitHub，HipChat，JIRA和无数其他应用程序的支持
	+ 启用许多Watcher警报用例
+ 通过按需轮询HTTP端点来创建事件
	+ 从Web应用程序界面通用捕获运行状况，性能，指标和其他类型的数据
	+ 非常适合优先选择轮询控制而不是接收的方案

### 数据存储和流编辑
从已经拥有的数据中发现更多价值。
+ 使用JDBC接口可以更好地了解来自任何关系数据库或NoSQL存储的数据
+ 统一来自Apache Kafka， RabbitMQ和Amazon SQS等消息队列的各种数据流

### 传感器和物联网编辑
探索其他数据的广泛性。
+ 在这个技术进步的时代，庞大的物联网世界通过捕获和利用来自连接传感器的数据来释放无尽的用例
+ Logstash是常见事件收集主干，用于提取从移动设备传送到智能家居，联网车辆，医疗保健传感器和许多其他特定于行业的应用程序的数据

### 能够轻松地丰富一切
在摄取期间清理和转换数据，以便在索引或输出时立即获得近乎实时的洞察力。
Logstash开箱即用，具有许多聚合和变异以及模式匹配，地理映射和动态查找功能。
+ Grok是Logstash过滤器的基础，广泛用于从非结构化数据中导出结构。享受多种旨在帮助快速解决Web，系统，网络和其他类型事件格式的集成模式。
+ 通过从IP地址解密地理坐标，标准化 日期复杂性，简化键值对和 CSV数据，对敏感信息进行指纹识别（匿名化），以及通过本地查找或Elasticsearch 查询进一步丰富数据，来扩展您的视野。
+ 编解码器通常用于简化对常见事件结构（如JSON 和多行事件）的处理。

### 选择合适的Stash
将数据路由到最重要的地方。通过存储，分析数据并对数据采取行动，解锁各种下游分析和操作用例。
+ 分析
	+ Elasticsearch
	+ Data stores such as MongoDB and Riak
+ 封存
	+ HDFS
	+ S3
+ 监控方式
	+ Nagios
	+ Ganglia
	+ Zabbix
	+ Graphite
	+ Datadog
	+ CloudWatch
+ 警示
	+ Watcher with Elasticsearch
	+ Email
	+ Pagerduty
	+ IRC
	+ SNS

# Logstash入门
## 安装Logstash
Logstash需要Java 8或Java11。请使用 正式的Oracle发行版或开源发行版，例如OpenJDK。

### 通过二进制文件安装
从[Logstash下载页面](https://www.elastic.co/downloads/logstash)下载适用于主机环境的Logstash安装文件-TARG.GZ，DEB，ZIP或RPM。
解压缩文件。
不要将Logstash安装到包含冒号（:)字符的目录路径中。

在受支持的Linux操作系统上，可以使用程序包管理器来安装Logstash。

### YUM安装
下载并安装公共签名密钥：
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

下面，将以下内容添加到/etc/yum.repos.d/目录中的带.repo后缀的文件中logstash.repo
```
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

下面，可以使用以下方法安装它：
```
sudo yum install logstash
```

## 保存一个事件
首先，让我们通过运行最基本的Logstash管道来测试Logstash安装。

Logstash管道具有两个必需元素input和output，以及一个可选元素filter。
输入插件使用来自源的数据，过滤器插件根据您的指定修改数据，输出插件将数据写入目标。

要测试Logstash安装，请运行最基本的Logstash管道。例如：
```
cd logstash-7.5.1
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

## 使用Logstash解析日志
下面将创建一个Logstash管道，该管道使用Filebeat来获取Apache Web日志作为输入，
解析这些日志以从日志中创建特定的命名字段，并将解析后的数据写入Elasticsearch集群。
无需在命令行上定义管道配置，而是在配置文件中定义管道。

### 配置Filebeat以将日志行发送到Logstash
在创建Logstash管道之前，将配置Filebeat以将日志行发送到Logstash。
该Filebeat客户端是一个轻量级的，资源友好的工具，从服务器上的文件和转发这些日志到您Logstash实例进行处理收集日志。
Filebeat专为可靠性和低延迟而设计。Filebeat在主机上的资源占用很少，该Beats input插件使Logstash实例上的资源需求最小化。

在一般用例中，Filebeat与运行Logstash实例的计算机在不同的计算机上运行。就本教程而言，Logstash和Filebeat在同一台计算机上运行。

安装Filebeat之后，您需要对其进行配置。打开filebeat.yml位于Filebeat安装目录中的文件，然后用以下几行替换内容。
```
filebeat.inputs:
- type: log
  paths:
    - /path/to/file/logstash-tutorial.log 
output.logstash:
  hosts: ["localhost:5044"]
```

之后使用以下命令运行Filebeat：
```
./filebeat -e -c filebeat.yml -d "publish"
```

### 配置Logstash接收Filebeat
创建一个使用Beats输入插件从Beats接收事件的Logstash配置管道。
以下是配置管道的框架：
```
* The # character at the beginning of a line indicates a comment. Use
* comments to describe your configuration.
input {
}
* The filter part of this file is commented out to indicate that it is
* optional.
* filter {
*
* }
output {
}
```

首先，将配置管道的框架复制并粘贴到first-pipeline.conf主Logstash目录中命名的文件中。

接下来，通过在 first-pipeline.conf 文件 input 部分添加以下行，将Logstash实例配置为使用Beats输入插件：
```
beats {
	port => "5044"
}
```

可以output将以下行添加到该部分，以便在运行Logstash时将输出打印到stdout：
```
stdout { codec => rubydebug }
```

最后，first-pipeline.conf 的内容应如下所示：
```
input {
	beats {
		port => "5044"
	}
}
* The filter part of this file is commented out to indicate that it is
* optional.
* filter {
*
* }
output {
	stdout { codec => rubydebug }
}
```

要验证配置，请运行以下命令：
```
bin/logstash -f first-pipeline.conf --config.test_and_exit
```
--config.test_and_exit选项解析您的配置文件并报告任何错误。

如果配置文件通过配置测试，请使用以下命令启动Logstash：
```
bin/logstash -f first-pipeline.conf --config.reload.automatic
```
--config.reload.automatic选项启用自动重新加载配置，因此您不必在每次修改配置文件时都停止并重新启动Logstash。

如果管道正常运行，则应该在控制台上看到一系列类似以下的事件：
```
{
    "@timestamp" => 2017-11-09T01:44:20.071Z,
        "offset" => 325,
      "@version" => "1",
          "beat" => {
            "name" => "My-MacBook-Pro.local",
        "hostname" => "My-MacBook-Pro.local",
         "version" => "6.0.0"
    },
          "host" => "My-MacBook-Pro.local",
    "prospector" => {
        "type" => "log"
    },
    "input" => {
        "type" => "log"
    },
        "source" => "/path/to/file/logstash-tutorial.log",
       "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
          "tags" => [
        [0] "beats_input_codec_plain_applied"
    ]
}
...
```

### 使用Grok过滤器插件解析Web日志
现在，您有了一个工作管道，该管道从Filebeat中读取日志行。但是，您会注意到日志消息的格式不是理想的。您想解析日志消息以从日志中创建特定的命名字段。
为此，您将使用grok过滤器插件。

使用grok过滤器插件，您可以将非结构化日志数据解析为结构化和可查询的内容。

由于grok过滤器插件会在传入的日志数据中查找模式，因此配置插件需要您做出如何确定用例感兴趣的模式的决策。

假设Web服务器日志示例中的代表行如下所示：
```
83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png
HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel
Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
```
该行开头的IP地址很容易识别，括号中的时间戳也很容易识别。

编辑first-pipeline.conf文件，并将整个filter部分替换为以下文本：
```
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
```

完成后，的内容first-pipeline.conf应如下所示：
```
input {
    beats {
        port => "5044"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
output {
    stdout { codec => rubydebug }
}
```

保存您的更改。由于您已启用自动配置重载，因此无需重新启动Logstash即可获取更改。但是，您确实需要强制Filebeat从头读取日志文件。
为此，请转到运行Filebeat的终端窗口，然后按Ctrl + C关闭Filebeat。然后删除Filebeat注册表文件。例如，运行：
```
rm data/registry
```

接下来，使用以下命令重新启动Filebeat：
```
filebeat -e -c filebeat.yml -d "publish"
```

在Logstash应用grok模式之后，事件将具有以下JSON表示形式：
```
{
        "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
         "offset" => 325,
           "auth" => "-",
          "ident" => "-",
           "verb" => "GET",
     "prospector" => {
        "type" => "log"
    },
     "input" => {
        "type" => "log"
    },
         "source" => "/path/to/file/logstash-tutorial.log",
        "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
           "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
       "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
     "@timestamp" => 2017-11-09T02:51:12.416Z,
       "response" => "200",
          "bytes" => "203023",
       "clientip" => "83.149.9.216",
       "@version" => "1",
           "beat" => {
            "name" => "My-MacBook-Pro.local",
        "hostname" => "My-MacBook-Pro.local",
         "version" => "6.0.0"
    },
           "host" => "My-MacBook-Pro.local",
    "httpversion" => "1.1",
      "timestamp" => "04/Jan/2015:05:13:42 +0000"
}
```

### 使用Geoip过滤器插件增强数据
除了解析日志数据以进行更好的搜索外，筛选器插件还可以从现有数据中获取补充信息。
例如，geoip插件查找IP地址，从地址中获取地理位置信息，然后将该位置信息添加到日志中。

### 索引你的数据到Elasticsearch
现在，Web日志已细分为特定字段，准备好将数据导入Elasticsearch。

Logstash管道可以将数据索引到Elasticsearch集群中。编辑first-pipeline.conf文件，并将整个output部分替换为以下文本：
```
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
```
通过此配置，Logstash使用http协议连接到Elasticsearch。

至此，first-pipeline.conf文件已正确配置了输入，过滤器和输出部分，如下所示
```
input {
    beats {
        port => "5044"
    }
}
 filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    geoip {
        source => "clientip"
    }
}
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
```

重新启动Filebeat：
```
filebeat -e -c filebeat.yml -d "publish"
```

### 测试您的管道

## 将多个输入和输出插件拼接在一起

# Logstash工作原理
Logstash事件处理管道包括三个阶段：输入→过滤器→输出。
输入会生成事件，过滤器会对其进行修改，输出会将它们发送到其他地方。
输入和输出支持编解码器，使您可以在数据进入或退出管道时对其进行编码或解码，而不必使用单独的过滤器。

## 输入 Inputs 
可以使用输入将数据获取到Logstash。

一些常用的输入：
+ file：从文件系统上的文件读取，非常类似于UNIX命令 tail -0F
+ syslog：在知名端口514上侦听syslog消息并根据RFC3164格式进行解析
+ redis：使用redis通道和redis列表从redis服务器读取。Redis经常在集中式Logstash安装中用作“代理”，该安装会将来自远程Logstash“托运人”的Logstash事件排队
+ beats：进程的事件发送的节拍

## 过滤器 Filters
过滤器是Logstash管道中的中间处理设备。如果事件符合特定条件，则可以将过滤器与条件语句结合使用以对事件执行操作。

一些常用的过滤器：
+ grok：解析和构造任意文本。Grok当前是Logstash中将非结构化日志数据解析为结构化和可查询内容的最佳方法。Logstash内置有120种模式，很可能会找到满足您需求的模式
+ mutate：对事件字段执行常规转换。您可以重命名，删除，替换和修改事件中的字段
+ drop：完全删除事件，例如调试事件
+ clone：复制事件，可能会添加或删除字段
+ geoip：添加有关IP地址地理位置的信息

## 输出 Outputs
输出是Logstash管道的最后阶段。**一个事件可以通过多个输出**，但是一旦完成所有输出处理，该事件就完成了执行。

一些常用的输出：
+ elasticsearch：将事件数据发送到Elasticsearch。如果您打算以一种高效，便捷且易于查询的格式保存数据，那么Elasticsearch是您的最佳选择
+ file：将事件数据写入磁盘上的文件
+ graphite：将事件数据发送到石墨，石墨是一种流行的开源工具，用于存储和绘制指标图形
+ statsd：将事件数据发送到statsd，该服务“通过UDP侦听统计信息（如计数器和计时器），并将聚合发送到一个或多个可插拔后端服务”

## 编解码器 Codecs
编解码器基本上是流过滤器，可以作为输入或输出的一部分进行操作。编解码器使您可以轻松地将消息的传输与序列化过程分开。

常用的的编解码器包括json，msgpack和plain：
+ json：以JSON格式编码或解码数据
+ multiline：将多行文本事件（例如java异常和stacktrace消息）合并为一个事件

# 设置并运行Logstash
## Logstash目录布局

## Logstash配置文件
Logstash有两种类型的配置文件：管道配置文件（用于定义Logstash处理管道）和设置文件（用于指定控制Logstash启动和执行的选项）。

### 管道配置文件
在定义Logstash处理管道的阶段时，可以创建管道配置文件。

### 设置文件
设置文件已在Logstash安装中定义。Logstash包括以下设置文件：

#### logstash.yml
包含Logstash配置标志。您可以在此文件中设置标志，而不是在命令行中传递标志。
您在命令行上设置的所有标志都将覆盖logstash.yml文件中的相应设置。

#### pipelines.yml
包含用于在单个Logstash实例中运行多个管道的框架和说明。

#### jvm.options
包含JVM配置标志。使用此文件来设置总堆空间的初始值和最大值。您也可以使用此文件设置Logstash的语言环境。
在单独的行上指定每个标志。此文件中的所有其他设置都被视为专家设置。

#### log4j2.properties
包含log4j 2库的默认设置。

#### startup.options （Linux）
包含该system-install脚本/usr/share/logstash/bin用来为您的系统构建适当的启动脚本的选项。
在安装Logstash软件包时，system-install脚本将在安装过程结束时执行，并使用中指定的设置startup.options来设置选项，例如用户，组，服务名称和服务描述。
默认情况下，Logstash服务安装在用户下logstash。该startup.options文件使您可以更轻松地安装Logstash服务的多个实例。
可以复制文件并更改特定设置的值。请注意，startup.options启动时不会读取文件。
如果要更改Logstash启动脚本（例如，要更改Logstash用户或从其他配置路径读取），则必须重新运行system-install 脚本（以root用户身份）以传入新设置。

## logstash.yml
## 秘密密钥库用于安全设置
## 从命令行运行Logstash
## 在Debian或RPM上将Logstash作为服务运行
## 在Docker上运行Logstash
## 为Docker配置Logstash
## 在Windows上运行Logstash
## 日志
## 关闭Logstash

# 升级Logstash
# 配置Logstash
要配置Logstash，需要创建一个配置文件，该文件指定要使用的插件以及每个插件的设置。
可以在配置中引用事件字段，并在条件满足特定条件时使用条件处理事件。
运行logstash时，您可以使用-f来指定您的配置文件。 

首先，创建一个名为“ logstash-simple.conf”的文件，并将其保存在与Logstash相同的目录中。
```
input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

然后，运行logstash并使用该-f标志指定配置文件。
```
bin/logstash -f logstash-simple.conf	
```
Logstash读取指定的配置文件，并输出到Elasticsearch和stdout。

## 配置文件的结构
Logstash配置文件针对要添加到事件处理管道中的每种插件类型都有一个单独的部分。例如：
```
input {
  ...
}

filter {
  ...
}

output {
  ...
}
```
每个部分都包含一个或多个插件的配置选项。
如果指定多个过滤器，则会按照它们在配置文件中**出现的顺序**进行应用。

### 插件配置
插件的配置包括插件名称，以及该插件的一组设置。
例如，此输入部分配置两个文件输入：
```
input {
  file {
    path => "/var/log/messages"
    type => "syslog"
  }

  file {
    path => "/var/log/apache/access.log"
    type => "apache"
  }
}
```
在此示例中，为每个文件输入配置插件了两个设置：path 和 type。
可以进行配置的配置项因插件类型而异。

### 值类型
插件可以要求设置的值是某种类型，例如boolean，list或hash。
具体支持以下值类型：

#### 数组
现在不建议使用此类型，而建议使用标准类型，例如string使用插件定义:list => true属性以更好地进行类型检查。
仍然需要处理不需要类型检查的哈希表或混合类型列表。
```
 users => [ {id => 1, name => bob}, {id => 2, name => jane} ]
```

#### 列表
它本身不是类型，但是属性类型可以具有。这样就可以键入检查多个值。插件作者可以通过:list => true在声明参数时指定来启用列表检查。
```
path => [ "/var/log/messages", "/var/log/*.log" ]
uris => [ "http://elastic.co", "http://example.net" ]
```

#### 布尔
布尔值必须为true或false。请注意，true和false关键字没有用引号引起来。
```
ssl_enable => true 
```

#### 字节
字节是代表有效字节单位的字符串字段。这是在插件选项中声明特定大小的便捷方法。
SI（k MGTPEZY）和Binary（Ki Mi Gi Ti Pi Ei Zi Yi）单元均受支持。
二进制单位为base-1024，SI单位为base-1000。该字段不区分大小写，并且接受值和单位之间的空格。
如果未指定单位，则整数字符串表示字节数。
```
my_bytes => "1113"   # 1113 bytes
my_bytes => "10MiB"  # 10485760 bytes
my_bytes => "100kib" # 102400 bytes
my_bytes => "180 mb" # 180000000 bytes
```

#### 编解码器
编解码器是用于表示数据的Logstash编解码器的名称。编解码器可用于输入和输出。

输入编解码器提供了一种在数据输入之前解码数据的便捷方法。
输出编解码器提供了一种方便的方式，可以在数据离开输出之前对其进行编码。
使用输入或输出编解码器，无需在Logstash管道中使用单独的过滤器。
```
codec => "json"
```

#### 哈希
哈希是格式指定的键值对的集合"field1" => "value1"。请注意，多个键值条目由空格而不是逗号分隔
```
match => {
  "field1" => "value1"
  "field2" => "value2"
  ...
}

match => { "field1" => "value1" "field2" => "value2" }
```

#### 数字
数字必须是有效的数值（浮点数或整数）
```
port => 33
```

#### 密码
密码是没有记录或打印的具有单个值的字符串。
```
my_password => "password"
```

#### URI
URI可以是任何内容，从完整的URL（如http://elastic.co/）到简单的标识符（如foob​​ar）。
如果URI包含诸如http：// user：pass@example.net之类的密码，则URI的密码部分将不会被记录或打印。
```
my_uri => “ http：// foo：bar@example.net” 
```

#### 路径
路径是代表有效操作系统路径的字符串。
```
my_path => “ / tmp / logstash” 
```

#### 字符串
字符串必须是单个字符序列。请注意，字符串值用双引号或单引号引起来。

#### 转义序列
默认情况下，不启用转义序列。
如果您希望在带引号的字符串使用转义序列，您将需要设置 config.support_escapes: true你的logstash.yml。
当时true，带引号的字符串（双精度和单精度）将具有以下转换：
```
\ r     回车（ASCII 13）
\ n		换行（ASCII 10）
\ t		标签（ASCII 9）
\\		反斜杠（ASCII 92）
\“		双引号（ASCII 34）
\'		单引号（ASCII 39）
```

```
name => "Hello world"
name => 'It\'s a beautiful day'
```

### 注释
注释与perl，ruby和python中的注释相同。注释以＃字符开头，不必在行首。例如：
```
	* this is a comment

	input { * comments can appear at the end of a line, too
	  * ...
	}
```

## 在配置中访问事件数据和字段
Logstash代理是一个包含3个阶段的处理管道：输入→过滤器→输出。输入生成事件，过滤器修改它们，输出将它们发送到其他地方。

所有事件都有属性。例如，一个Apache访问日志将具有状态码（200、404），请求路径（“ /"、"index.html")、HTTP动词（GET，POST），客户端IP地址等内容。Logstash将这些属性称作 “字段”。

Logstash中的某些配置选项需要存在字段才能运行。因为输入时生成事件，所以输入中没有要评估的字段。
由于它们依赖于事件和字段，因此以下配置选项仅在过滤器和输出内起作用。

### 字段引用
能够按名称引用字段通常很有用。

访问字段的基本语法为 [fieldname]。
如果是 顶级字段，则可以省略[]，而只需使用即可fieldname。
要引用 嵌套字段，请指定该字段的完整路径，如：[top-level field] [nested field]。

以下事件具有五个顶级字段（agent, ip, request, response, ua）和三个嵌套字段（status, bytes, os）：
```
{
  "agent": "Mozilla/5.0 (compatible; MSIE 9.0)",
  "ip": "192.168.24.44",
  "request": "/index.html"
  "response": {
    "status": 200,
    "bytes": 52353
  },
  "ua": {
    "os": "Windows 7"
  }
}
```
要引用诸如的顶级字段 request，您只需指定字段名称。要引用该os字段，请指定 [ua] [os]。

### sprintf格式
字段引用格式也用在Logstash所谓的sprintf格式中。这种格式使您可以从其他字符串中引用字段值。
例如，statsd output 具有一个增量设置，使您可以通过状态代码来保留apache日志的计数：
```
output {
  statsd {
    increment => "apache.%{[response][status]}"
  }
}
```

同样，可以将@timestamp字段中的时间戳转换为字符串。而不是在花括号内指定字段名称，而应使用 +FORMAT语法。
例如，如果要根据事件的时间以及type字段使用文件输出来写入日志：
```
output {
  file {
    path => "/var/log/%{type}.%{+yyyy.MM.dd.HH}"
  }
}
```

### 条件判断
有时，想在特定条件下过滤或输出事件。这是可以使用条件判断。

Logstash中的条件语句在外观和行为上与在编程语言中相同。条件语句支持if，else if以及else报表和可以被嵌套。

条件判断的语法为：
```
if EXPRESSION {
  ...
} else if EXPRESSION {
  ...
} else {
  ...
}
```

### @metadata字段
在Logstash 1.5和更高版本中，有一个名为 @metadata 的特殊字段。
@metadata的内容在输出时，将不属于任何事件，因此非常适合用于条件条件，或使用字段引用和sprintf格式扩展和构建事件字段。

以下配置文件将产生来自STDIN的事件。键入的内容将成为message事件中的字段。mutate过滤器中的事件将添加一些字段，其中一些嵌套在该@metadata字段中。
```
input { stdin { } }

filter {
  mutate { add_field => { "show" => "This data will be in the output" } }
  mutate { add_field => { "[@metadata][test]" => "Hello" } }
  mutate { add_field => { "[@metadata][no_show]" => "This data will not be in the output" } }
}

output {
  if [@metadata][test] == "Hello" {
    stdout { codec => rubydebug }
  }
}
```

效果如下：
```
$ bin/logstash -f ../test.conf
Pipeline main started
asdf
{
    "@timestamp" => 2016-06-30T02:42:51.496Z,
      "@version" => "1",
          "host" => "example.com",
          "show" => "This data will be in the output",
       "message" => "asdf"
}
```
输入的“ asdf”成为message字段内容，并且条件test语句成功评估了嵌套在该@metadata字段中的字段 的内容。但是输出未显示名为的字段@metadata或其内容。

rubydebug编解码器可以显示@metadata的内容，需要添加一个配置标志字段，metadata => true：
```
stdout { codec => rubydebug { metadata => true } }
```

配置修改完后的效果，可以看到@metadata字段及其子字段：
```
$ bin/logstash -f ../test.conf
Pipeline main started
asdf
{
    "@timestamp" => 2016-06-30T02:46:48.565Z,
     "@metadata" => {
           "test" => "Hello",
        "no_show" => "This data will not be in the output"
    },
      "@version" => "1",
          "host" => "example.com",
          "show" => "This data will be in the output",
       "message" => "asdf"
}
```

**每当需要临时字段但不希望其出现在最终输出中时，请使用该字段@metadata。**

此新字段最常见的用例之一可能是使用date 过滤器并具有临时时间戳。过去，您必须先删除timestamp字段，然后再使用它来覆盖该@timestamp 字段。
对于@metadata字段，这不再是必需的：
```
input { stdin { } }

filter {
  grok { match => [ "message", "%{HTTPDATE:[@metadata][timestamp]}" ] }
  date { match => [ "[@metadata][timestamp]", "dd/MMM/yyyy:HH:mm:ss Z" ] }
}

output {
  stdout { codec => rubydebug }
}
```

此配置将提取的日期放入过滤器的 [@metadata] [timestamp]字段中grok。让我们为该配置提供一个示例日期字符串，然后看看结果如何：
```
$ bin/logstash -f ../test.conf
Pipeline main started
02/Mar/2014:15:36:43 +0100
{
    "@timestamp" => 2014-03-02T14:36:43.000Z,
      "@version" => "1",
          "host" => "example.com",
       "message" => "02/Mar/2014:15:36:43 +0100"
}
```

## 在配置中使用环境变量

## Logstash配置示例
### 配置过滤器
筛选器是一种在线处理机制，可灵活地对数据进行切片和切块以适应您的需求。
让我们看一下一些实际使用的过滤器。以下配置文件设置grok和date过滤器。
```
input { stdin { } }

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

### 处理Apache日志
做一些实际上有用的事情，处理apache2访问日志文件。从本地主机上的文件中读取输入，并根据我们的需要使用条件处理事件。
```
input {
  file {
    path => "/tmp/access_log"
    start_position => "beginning"
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { "type" => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
  stdout { codec => rubydebug }
}
```

### 使用条件判断
可以使用条件控件来控制过滤器或输出处理哪些事件。
例如，可以根据事件出现在哪个文件中来标记每个事件（access_log，error_log以及其他以“ log”结尾的随机文件）。
```
input {
  file {
    path => "/tmp/*_log"
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { type => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
  } else if [path] =~ "error" {
    mutate { replace => { type => "apache_error" } }
  } else {
    mutate { replace => { type => "random_logs" } }
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

### 处理系统日志消息
Syslog是Logstash的最常见用例之一，它处理得非常好（只要日志行大致符合RFC3164）。
Syslog是事实上的UNIX网络记录标准，它通过rsyslog将消息从客户端计算机发送到本地文件或集中式日志服务器。
```
input {
  tcp {
    port => 5000
    type => syslog
  }
  udp {
    port => 5000
    type => syslog
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

## Logstash配置安全性（X-Pack）

# 高级Logstash配置
# 管理Logstash
# 使用Logstash模块
# 使用Filebeat模块
# 数据弹性
# 转换资料
# 部署和扩展Logstash
# 性能调优
# 使用API​​监视Logstash
# 使用X-Pack监视Logstash
# 使用插件
Logstash具有丰富的输入，过滤器，编解码器和输出的插件集合。插件以称为gems的自包含软件包形式提供，并托管在RubyGems.org上。
通过bin/logstash-plugin脚本访问的插件管理器用于管理Logstash部署中插件的生命周期。
您可以使用下面描述的命令行界面（CLI）调用来安装，删除和升级插件。

## 代理配置
大多数插件管理器命令都需要访问Internet才能访问RubyGems.org。
```
export http_proxy=http://localhost:3128
export https_proxy=http://localhost:3128
```

## 查看插件
Logstash发行包捆绑了常见插件，因此可以立即使用它们。列出部署中当前可用的插件：
+ 将列出所有已安装的插件
+ 将列出已安装的插件以及版本信息
+ 将列出所有安装的包含名称片段的插件
+ 将列出特定组的所有已安装插件（输入，过滤器，编解码器，输出
```
bin/logstash-plugin list 
bin/logstash-plugin list --verbose 
bin/logstash-plugin list '*namefragment*' 
bin/logstash-plugin list --group output 
```

## 安装插件
安装插件时最常见的情况是可以访问Internet。使用此方法，您将能够检索公共资源库（RubyGems.org）上托管的插件，并在Logstash安装的顶部进行安装。
```
bin/logstash-plugin install logstash-output-kafka
```

## 更新插件
插件有其自己的发布周期，并且通常独立于Logstash的核心发布周期进行发布。使用update子命令，您可以获得插件的最新版本。
```
bin/logstash-plugin update 
bin/logstash-plugin update logstash-output-kafka 
```

## 删除插件
如果您需要从Logstash安装中删除插件，请执行以下操作：
```
bin/logstash-plugin remove logstash-output-kafka
```

# 集成插件
集成插件，是将相关的插件（输入，输出，有时还包括过滤器和编解码器）组合到一个软件包。
+ kafka：与Kafka分布式流平台一起使用的插件
+ Rabbitmq：用于处理往返RabbitMQ代理的事件的插件

# 输入插件
输入插件使Logstash可以读取特定的事件源。

常用输入插件：
+ azure_event_hubs 从Azure事件中心接收事件
+ beats 从Elastic Beats框架接收事件
+ cloudwatch 从Amazon Web Services CloudWatch API提取事件
+ couchdb_changes 从CouchDB的changesURI 流事件
+ dead_letter_queue 从Logstash的死信队列中读取事件
+ elasticsearch 从Elasticsearch集群读取查询结果
+ exec 将shell命令的输出捕获为事件
+ file 从文件流事件
+ ganglia 通过UDP读取Ganglia数据包
+ gelf 从Graylog2读取GELF格式的消息作为事件
+ generator 生成随机日志事件以进行测试
+ github 从GitHub Webhook读取事件
+ google_cloud_storage 从Google Cloud Storage存储桶中的文件中提取事件
+ google_pubsub 消费来自Google Cloud PubSub服务的事件
+ graphite 从graphite工具读取指标
+ heartbeat 生成心跳事件以进行测试
+ http 通过HTTP或HTTPS接收事件
+ http_poller 将HTTP API的输出解码为事件
+ imap 从IMAP服务器读取邮件
+ irc 从IRC服务器读取事件
+ java_generator 生成综合日志事件
+ java_stdin 从标准输入读取事件
+ jdbc 从JDBC数据创建事件
+ jms 从Jms Broker读取事件
+ jmx 通过JMX从远程Java应用程序检索指标
+ kafka 读取来自Kafka主题的事件
+ kinesis 通过AWS Kinesis流接收事件
+ log4j 从Log4j SocketAppender对象通过TCP套接字读取事件
+ lumberjack 使用Lumberjack协议接收事件
+ meetup 将命令行工具的输出捕获为事件
+ pipe 从长时间运行的命令管道流式传输事件
+ puppet_facter 接收来自Puppet服务器的事实
+ rabbitmq 从RabbitMQ交换中提取事件
+ redis 从Redis实例读取事件
+ relp 通过TCP套接字接收RELP事件
+ rss 将命令行工具的输出捕获为事件
+ s3 从S3存储桶中的文件流式传输事件
+ salesforce 根据Salesforce SOQL查询创建事件
+ snmp 使用简单网络管理协议（SNMP）轮询网络设备
+ snmptrap 根据SNMP陷阱消息创建事件
+ sqlite 根据SQLite数据库中的行创建事件
+ sqs 从Amazon Web Services简单队列服务队列中提取事件
+ stdin 从标准输入读取事件
+ stomp 创建使用STOMP协议接收的事件
+ syslog 读取系统日志消息作为事件
+ tcp 从TCP套接字读取事件
+ twitter 从Twitter Streaming API读取事件
+ udp 通过UDP读取事件
+ unix 通过UNIX套接字读取事件
+ varnishlog 从varnish缓存共享内存日志中读取
+ websocket 从网络套接字读取事件
+ wmi 根据WMI查询的结果创建事件
+ xmpp 通过XMPP / Jabber协议接收事件

## Beats 输入插件
该输入插件使Logstash可以从Elastic Beats框架接收事件。

以下示例显示如何配置Logstash以在端口5044上侦听传入的Beats连接并索引到Elasticsearch：
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}" 
  }
}
```

### Beats 索引设置
为了最大程度地减少未来架构更改对Elasticsearch中现有索引和映射的影响，请配置Elasticsearch输出以写入版本索引：
```
index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
```
+ %{[@metadata] [beat]} 将索引名称的第一部分设置为beat元数据字段的值，例如filebeat。
+ %{[@metadata] [version]} 将名称的第二部分设置为Beat版本，例如7.5.1。
+ %{+YYYY.MM.dd} 根据Logstash @timestamp字段将名称的第三部分设置为日期。

### Beats 配置项
该插件支持以下 配置选项 以及 通用选项。

#### add_hostname
标记以确定是否host使用该节拍中的节拍提供的值将字段添加到事件中hostname。
在6.0.0中已弃用。在7.0.0中，此设置将被删除。
+ 值类型为布尔值
+ 默认值为 false

#### cipher_suites
要使用的密码套件列表，按优先级列出。
+ 值类型为数组
+ 默认值为 java.lang.String[TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256]@459cfcca

#### client_inactivity_timeout
X秒钟不活动后，关闭空闲客户端。
+ 值类型是数字
+ 默认值为 60

#### host
要监听的IP地址。
+ 值类型为字符串
+ 默认值为 "0.0.0.0"

#### include_codec_tag
+ 值类型为布尔值
+ 默认值为 true

#### port
要监听的端口。
+ 值类型是数字
+ 这是必需的设置，没有默认值

#### ssl
默认情况下，事件以纯文本发送。您可以通过设置ssl为true并配置ssl_certificate和ssl_key选项来启用加密。
+ 值类型为布尔值
+ 默认值为 false

#### ssl_certificate
要使用的SSL证书。
+ 值类型是路径
+ 此设置没有默认值

#### ssl_certificate_authorities
根据这些权限验证客户证书。您可以定义多个文件或路径。所有证书将被读取并添加到信任存储中。您需要配置ssl_verify_mode 到peer或force_peer启用验证。
+ 值类型为数组
+ 默认值为 []

#### ssl_handshake_timeout
ssl握手不完整超时的时间（以毫秒为单位）
+ 值类型是数字
+ 默认值为 10000

#### ssl_key
要使用的SSL密钥。注意：该密钥必须为PKCS8格式，您可以使用OpenSSL对其 进行转换以获取更多信息。
+ 值类型是路径
+ 此设置没有默认值

#### ssl_key_passphrase
要使用的SSL密钥密码。
+ 值类型为密码
+ 此设置没有默认值

#### ssl_verify_mode
默认情况下，服务器不执行任何客户端验证。
peer将使服务器要求客户端提供证书。如果客户端提供证书，则将对其进行验证。
force_peer将使服务器要求客户端提供证书。如果客户端不提供证书，则连接将关闭。
此选项需要与ssl_certificate_authoritiesCA 一起使用，并已定义了CA列表。
+ 值可以是任何的：none，peer，force_peer
+ 默认值为 "none"

#### ssl_peer_metadata
允许在事件的元数据中存储客户端证书信息。
仅当ssl_verify_mode设置为peer或时，此选项才有效force_peer。
+ 值类型为布尔值
+ 默认值为 false

#### tls_max_version
加密连接允许的最大TLS版本。该值必须是以下之一：TLS 1.0的1.0，TLS 1.1的1.1，TLS 1.2的1.2
+ 值类型是数字
+ 默认值为 1.2

#### tls_min_version
加密连接允许的最低TLS版本。该值必须是以下之一：TLS 1.0的1.0，TLS 1.1的1.1，TLS 1.2的1.2
+ 值类型是数字
+ 默认值为 1

### 通用配置项
#### add_field
向事件添加字段。
+ 值类型为哈希
+ 默认值为 {}

#### codec
用于输入数据的编解码器。输入编解码器是一种在数据输入之前解码数据的便捷方法，而无需在Logstash管道中使用单独的过滤器。
+ 值类型为编解码器
+ 默认值为 "plain"

#### enable_metric
默认情况下，为此特定插件实例禁用或启用度量标准日志记录，我们会记录所有可以度量的数据，但是您可以禁用特定插件的度量标准收集。
+ 值类型为布尔值
+ 默认值为 true

#### id
ID向插件配置添加唯一。如果未指定ID，Logstash将生成一个。
+ 值类型为字符串
+ 此设置没有默认值

强烈建议在您的配置中设置此ID。当您有两个或多个相同类型的插件时（例如，如果您有2个拍子输入），这特别有用。
在这种情况下，添加命名ID将有助于在使用监视API时监视Logstash。
```
input {
  beats {
    id => "my_plugin_id"
  }
}
```

#### tags
将任意数量的任意标签添加到您的事件中，这可以帮助以后进行处理。
+ 值类型为数组
+ 此设置没有默认值

#### type
type向此输入处理的所有事件添加一个字段。
+ 值类型为字符串
+ 此设置没有默认值

type主要用于过滤器激活。该类型存储为事件本身的一部分，因此您也可以使用该类型在Kibana中进行搜索。

# 输出插件
输出插件将事件数据发送到特定的目的地。输出是事件管道中的最后阶段。

常用输入插件：
+ boundary 根据Logstash事件将注释发送到边界
+ circonus 根据Logstash事件将注释发送到Circonus
+ cloudwatch 汇总指标数据并将其发送到AWS CloudWatch
+ csv 以分隔格式将事件写入磁盘
+ datadog 根据Logstash事件将事件发送到DataDogHQ
+ datadog_metrics 根据Logstash事件将指标发送到DataDogHQ
+ elastic_app_search 将事件发送到Elastic App Search解决方案
+ elasticsearch 将日志存储在Elasticsearch中
+ email 收到输出后将电子邮件发送到指定的地址
+ exec 运行匹配事件的命令
+ file 将事件写入磁盘上的文件
+ ganglia 将指标写入Ganglia的 gmond
+ gelf 为Graylog2生成GELF格式的输出
+ google_bigquery 将事件写入Google BigQuery
+ google_cloud_storage 将日志事件上传到Google Cloud Storage
+ google_pubsub 将日志事件上传到Google Cloud Pubsub
+ graphite 将指标写入Graphite
+ graphtastic 在Windows上发送指标数据
+ http 将事件发送到通用HTTP或HTTPS端点
+ influxdb 将指标写入InfluxDB
+ irc 将事件写入IRC
+ java_sink 丢弃收到的所有事件
+ java_stdout 将事件打印到外壳的STDOUT
+ juggernaut 将消息推送到Juggernaut Websockets服务器
+ kafka 将事件写入Kafka主题
+ librato 根据Logstash事件将指标，注释和警报发送到Librato
+ loggly 将日志发送到Loggly
+ lumberjack 使用lumberjack协议发送事件
+ metriccatcher 将指标写入MetricCatcher
+ mongodb 将事件写入MongoDB
+ nagios 将被动检查结果发送给Nagios
+ nagios_nsca 使用NSCA协议将被动检查结果发送到Nagios
+ opentsdb 将指标写入OpenTSDB
+ pagerduty 根据预先配置的服务和升级策略发送通知s
+ pipe 将事件通过管道传递到另一个程序的标准输入
+ rabbitmq 将事件推送到RabbitMQ交换
+ redis 使用以下RPUSH命令将事件发送到Redis队列
+ redmine 使用Redmine API创建票证
+ riak 将事件写入Riak分布式键/值存储
+ riemann 将指标发送给黎曼
+ s3 将Logstash事件发送到Amazon Simple Storage Service
+ sns 将事件发送到亚马逊的简单通知服务
+ solr_http 在Solr中存储和索引日志
+ sqs 将事件推送到Amazon Web Services简单队列服务队列
+ statsd 使用statsd网络守护程序发送指标
+ stdout 将事件打印到标准输出
+ stomp 使用STOMP协议写入事件
+ syslog 将事件发送到syslog服务器
+ tcp 通过TCP套接字写入事件
+ timber 将事件发送到Timber.io日志记录服务
+ udp 通过UDP发送事件
+ webhdfs 使用webhdfsREST API将Logstash事件发送到HDFS
+ websocket 将消息发布到Websocket
+ xmpp 通过XMPP发布事件
+ zabbix 将事件发送到Zabbix服务器

## Http 输出插件
此输出使您可以将事件发送到通用HTTP（S）端点。
此输出将最多并行执行pool_max请求以提高性能。调整此插件的性能时请考虑这一点。
另外，请注意，使用并行执行时，不能保证事件的严格排序！
当心，该gem还不支持编解码器。请现在使用格式选项。

### Http 配置项
#### automatic_retries
客户端应重试失败的URL多​​少次。如果启用了保持活动，我们强烈建议不要将此值设置为零。某些服务器错误地提前终止keepalive，需要重试！
注意：如果retry_non_idempotent设置，则仅重试GET，HEAD，PUT，DELETE，OPTIONS和TRACE请求。
+ 值类型是数字
+ 默认值为 1

#### cacert
如果您需要使用自定义X.509 CA（.pem证书），请在此处指定路径。
+ 值类型是路径
+ 此设置没有默认值

#### client_cert
如果您想使用客户端证书（请注意，大多数人不希望这样做），请在此处设置x509证书的路径。
+ 值类型是路径
+ 此设置没有默认值

#### client_key
如果您使用的是客户端证书，请在此处指定加密密钥的路径。
+ 值类型是路径
+ 此设置没有默认值

#### connect_timeout
等待建立连接的超时时间（以秒为单位）。默认是10s
+ 值类型是数字
+ 默认值为 10

#### content_type
内容类型。
+ 值类型为字符串
+ 此设置没有默认值

如果未指定，则默认为以下内容：
如果格式为“ json”，则“application / json”。
如果格式为“ json_batch”，则为“application / json”。每个Logstash事件批次将被串联到一个数组中，并在一个请求中发送。
如果格式为“form”，“ application/ x-www-form-urlencoded”。

#### cookies
启用cookie支持。启用此功能后，客户端将像正常的Web浏览器一样在请求之间保留cookie。
+ 值类型为布尔值
+ 默认值为 true

#### follow_redirects
是否应遵循重定向？默认为true。
+ 值类型为布尔值
+ 默认值为 true

#### format
设置http正文的格式。
+ 值可以是任何的：json，json_batch，form，message
+ 默认值为 "json"

如果为json_batch，则此输出接收到的每批事件将放入单个JSON数组中，并在一个请求中发送。这对于高吞吐量方案（例如在Logstash实例之间发送数据）特别有用。
如果是form，则将主体（将映射（或整个事件））转换为查询参数字符串，例如 foo=bar&baz=fizz...
如果是message，则正文将是根据消息格式化事件的结果。
否则，事件将作为json发送。

#### headers
自定义标题使用的格式是 headers => ["X-My-Header", "%{host}"]。
+ 值类型为哈希
+ 此设置没有默认值

#### http_compression
启用请求压缩支持。启用此功能后，插件将使用gzip压缩http请求。
+ 值类型为布尔值
+ 默认值为 false

#### http_method
HTTP动词。"put", "post", "patch", "delete", "get", "head"之一
+ 这是必需的设置。
+ 值可以是任何的：put，post，patch，delete，get，head
+ 此设置没有默认值。

#### ignorable_codes
如果您认为某些非2xx代码是成功的，请在此处枚举它们。返回这些代码的响应将被视为成功。
+ 值类型是数字
+ 此设置没有默认值。

#### keepalive
启用此功能以启​​用HTTP Keepalive支持。我们强烈建议automatic_retries为此设置至少一个，以修复与失效的keepalive实现的交互。
+ 值类型为布尔值
+ 默认值为 true

#### keystore
如果您需要使用自定义密钥库（.jks），请在此处指定。这不适用于.pem键！
+ 值类型是路径
+ 此设置没有默认值

#### keystore_password
在此处指定密钥库密码。请注意，使用keytool创建的大多数.jks文件都需要密码！
+ 值类型为密码
+ 此设置没有默认值

#### keystore_type
在此处指定密钥库类型。其中一个JKS或PKCS12。默认是JKS。
+ 值类型为字符串
+ 默认值为 "JKS"

#### mapping
这使您可以选择发送事件的结构和部分。
+ 值类型为哈希
+ 此设置没有默认值

例如：
```
mapping => {"foo" => "%{host}" "bar" => "%{type}"}
```

#### message
+ 值类型为字符串
+ 此设置没有默认值

#### pool_max
最大并发连接数。默认为50
+ 值类型是数字
+ 默认值为 50

#### pool_max_per_route
到单个主机的最大并发连接数。默认为25。
+ 值类型是数字
+ 默认值为 25

#### proxy
如果使用HTTP代理。
+ 值类型为字符串
+ 此设置没有默认值

这支持多种配置语法：
代理主机的形式： http://proxy.org:1234
代理主机的形式： {host => "proxy.org", port => 80, scheme => 'http', user => 'username@host', password => 'password'}
代理主机的形式： {url =>  'http://proxy.org:1234', user => 'username@host', password => 'password'}

#### request_timeout
这个模块可以很容易地基于[Manticore]向日志记录添加非常完整配置的HTTP客户端。
+ 值类型是数字
+ 默认值为 60

#### retry_failed
如果您不希望此输出重试失败的请求，请将其设置为false。
+ 值类型为布尔值
+ 默认值为 true

#### retry_non_idempotent
如果automatic_retries启用此选项，将导致重试非幂等HTTP动词（例如POST）。
+ 值类型为布尔值
+ 默认值为 false

#### retryable_codes
如果遇到响应代码，此插件将重试这些请求。
+ 值类型是数字
+ 默认值为 [429, 500, 502, 503, 504]

#### socket_timeout
等待套接字上的数据超时（以秒为单位）。默认是10s。
+ 值类型是数字
+ 默认值为 10

#### truststore
如果您需要使用自定义信任库（.jks），请在此处指定。这不适用于.pem证书！
+ 值类型是路径
+ 此设置没有默认值

#### truststore_password
在此处指定信任库密码。请注意，使用keytool创建的大多数.jks文件都需要密码！
+ 值类型为密码
+ 此设置没有默认值

#### truststore_type
在此处指定信任库类型。其中一个JKS或PKCS12。默认是JKS。
+ 值类型为字符串
+ 默认值为 "JKS"

#### url
使用的网址。
这是必需的设置。
值类型为字符串
此设置没有默认值。

#### validate_after_inactivity
在使用keepalive对连接执行请求之前，检查连接是否陈旧之前需要等待的时间。
您可能希望将此值设置为较低，如果经常收到连接错误，则可能设置为0。
+ 值类型是数字
+ 默认值为 200

### 通用配置项
#### codec
用于输出数据的编解码器。输出编解码器是在数据离开输出之前进行编码的便捷方法，而无需在Logstash管道中使用单独的过滤器。
+ 值类型为编解码器
+ 默认值为 "plain"

#### enable_metric
禁用或启用此特定插件实例的度量标准日志记录。默认情况下，我们记录我们可以记录的所有指标，但是您可以禁用特定插件的指标收集。
+ 值类型为布尔值
+ 默认值为 true

#### id
ID向插件配置添加唯一。如果未指定ID，Logstash将生成一个。强烈建议在您的配置中设置此ID。当您有两个或多个相同类型的插件时，此功能特别有用。
例如，如果您有2个http输出。在这种情况下，添加命名ID将有助于在使用监视API时监视Logstash。
+ 值类型为字符串
+ 此设置没有默认值

# 过滤器插件
过滤器插件对事件执行中间处理。通常根据事件的特征有条件地应用过滤器。

常用过滤器插件：
+ aggregate 汇总来自单个任务的多个事件的信息
+ alter 对mutate过滤器无法处理的字段执行常规更改
+ bytes 将计算机存储大小（例如“ 123 MB”或“ 5.6gb”）的字符串表示形式解析为字节形式的数值
+ cidr 根据网络阻止列表检查IP地址
+ cipher 对事件应用或删除密码
+ clone 重复事件
+ csv 将逗号分隔的值数据解析为单个字段
+ date 解析字段中的日期以用作事件的Logstash时间戳
+ de_dot 计算上昂贵的过滤器，可从字段名称中删除点
+ dissect 使用定界符将非结构化事件数据提取到字段中
+ dns 执行标准或反向DNS查找
+ drop 删除所有事件
+ elapsed 计算两次事件之间的经过时间
+ elasticsearch 将字段从Elasticsearch中的先前日志事件复制到当前事件
+ environment 将环境变量存储为元数据子字段
+ extractnumbers 从字符串中提取数字
+ fingerprint 通过用一致的哈希替换值来指纹字段
+ geoip 添加有关IP地址的地理信息
+ grok 将非结构化事件数据解析为字段
+ http 提供与外部Web服务/ REST API的集成
+ i18n 从字段中删除特殊字符
+ java_uuid 生成一个UUID并将其添加到每个已处理的事件
+ jdbc_static 通过从远程数据库预加载的数据来丰富事件
+ jdbc_streaming 利用数据库数据丰富事件
+ json 解析JSON事件
+ json_encode 将字段序列化为JSON
+ kv 解析键值对
+ memcached 提供与Memcached中外部数据的集成
+ metricize 接收包含多个度量标准的复杂事件，并将其拆分为多个事件，每个事件都包含一个度量标准
+ metrics 汇总指标
+ mutate 在字段上执行突变
+ prune 根据要列入黑名单或白名单的字段列表修剪事件数据
+ range 检查指定的字段是否在给定的大小或长度限制内
+ ruby 执行任意的Ruby代码
+ sleep 休眠指定时间
+ split 将多行消息拆分为不同的事件
+ syslog_pri 解析消息的PRI（优先级）字段syslog
+ threats_classifier 利用有关攻击者意图的信息丰富安全日志
+ throttle 限制事件数量
+ tld 用您在配置中指定的内容替换默认消息字段的内容
+ translate 根据哈希或YAML文件替换字段内容
+ truncate 截断长度超过给定长度的字段
+ urldecode 解码URL编码的字段
+ useragent 将用户代理字符串解析为字段
+ uuid 将UUID添加到事件
+ xml 将XML解析为字段

## mutate 过滤器插件
转换（mutate）过滤器可让您对字段执行常规操作。您可以重命名，删除，替换和修改事件中的字段。

### 处理顺序
+ coerce 胁迫
+ rename 重命名
+ update 更新
+ replace 代替
+ convert 类型转换
+ gsub 替换
+ uppercase 大写
+ capitalize 首字母大写
+ lowercase 小写
+ strip 去空
+ remove 删除
+ split 分割
+ join 拼接
+ merge 合并
+ copy 复制

可以通过使用单独的变异块来控制顺序：
```
filter {
    mutate {
        split => ["hostname", "."]
        add_field => { "shortHostname" => "%{hostname[0]}" }
    }

    mutate {
        rename => ["shortHostname", "hostname" ]
    }
}
```

### 通用配置项
#### convert
+ 值类型为哈希
+ 此设置没有默认值

将字段的值转换为其他类型，例如将字符串转换为整数。如果字段值为数组，则将转换所有成员。如果该字段是哈希，则不会采取任何措施。

有效的转换目标及其在不同输入下的预期行为是：
+ integer：
+ integer_eu：
+ float：
+ float_eu：
+ string：
+ boolean：

该插件可以转换同一文档中的多个字段，请参见下面的示例：
```
filter {
  mutate {
	convert => {
	  "fieldname" => "integer"
	  "booleanfield" => "boolean"
	}
  }
}
```

#### copy
+ 值类型为哈希
+ 此设置没有默认值

将现有字段复制到另一个字段。目标字段若已存在将被覆盖。
```
filter {
  mutate {
	 copy => { "source_field" => "dest_field" }
  }
}
```

#### gsub
+ 值类型为数组
+ 此设置没有默认值

将正则表达式与字段值匹配，然后将所有匹配项替换为替换字符串。
仅支持字符串或字符串数​​组的字段。对于其他类型的字段，将不采取任何措施。
```
filter {
  mutate {
	gsub => [
	  #replace all forward slashes with underscore
	  "fieldname", "/", "_",
	  #replace backslashes, question marks, hashes, and minuses
	  with a dot "."
	  "fieldname2", "[\\?#-]", "."
	]
  }
}
```

#### join
+ 值类型为哈希
+ 此设置没有默认值

用分隔符连接数组。对非数组字段不执行任何操作。
```
filter {
 mutate {
   join => { "fieldname" => "," }
 }
}
```

#### lowercase
+ 值类型为数组
+ 此设置没有默认值

将字符串转换为其小写形式。
```
filter {
  mutate {
	lowercase => [ "fieldname" ]
  }
}
```

#### merge
+ 值类型为哈希
+ 此设置没有默认值
+ 
合并两个数组或哈希字段。字符串字段将自动转换为数组。
```
filter {
  mutate {
	 merge => { "dest_field" => "added_field" }
  }
}
```

#### coerce
+ 值类型为哈希
+ 此设置没有默认值

设置存在但为空的字段的默认值。
```
filter {
  mutate {
	#Sets the default value of the 'field1' field to 'default_value'
	coerce => { "field1" => "default_value" }
  }
}
```

#### rename
+ 值类型为哈希
+ 此设置没有默认值

重命名一个或多个字段。
```
filter {
  mutate {
	#Renames the 'HOSTORIP' field to 'client_ip'
	rename => { "HOSTORIP" => "client_ip" }
  }
}
```

#### replace
+ 值类型为哈希
+ 此设置没有默认值

用新值替换字段的值。新值可以包含%{foo}字符串，以帮助您从事件的其他部分构建新值。
```
filter {
  mutate {
	replace => { "message" => "%{source_host}: My new message" }
  }
}
```

#### split
+ 值类型为哈希
+ 此设置没有默认值

使用分隔符将字段拆分为数组。仅适用于字符串字段。
```
filter {
  mutate {
	 split => { "fieldname" => "," }
  }
}
```

#### strip
+ 值类型为数组
+ 此设置没有默认值

从字段中删除空格。注意：这仅适用于前导和尾随空白。
```
filter {
  mutate {
	 strip => ["field1", "field2"]
  }
}
```

#### update
+ 值类型为哈希
+ 此设置没有默认值

用新值更新现有字段。如果该字段不存在，则不会采取任何措施。
```
filter {
  mutate {
	update => { "sample" => "My new message" }
  }
}
```

#### uppercase
+ 值类型为数组
+ 此设置没有默认值

将字符串转换为其大写形式。
```
filter {
  mutate {
	uppercase => [ "fieldname" ]
  }
}
```

#### capitalize
+ 值类型为数组
+ 此设置没有默认值

将字符串转换为其首字母大写形式。
```
filter {
  mutate {
	capitalize => [ "fieldname" ]
  }
}
```

#### tag_on_failure
+ 值类型为字符串
+ 此设置的默认值为 _mutate_error

如果在应用此mutate过滤器期间发生故障，则其余操作将中止，并将提供的标签添加到事件中。

# 编解码器插件
# 提示和最佳做法
# 解决常见问题
# 为Logstash贡献
# 贡献一个Java插件
# 专业术语
# 重大变化
# 发行说明