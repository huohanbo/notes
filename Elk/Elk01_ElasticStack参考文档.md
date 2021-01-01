# 参考资料
+ [Elastic Stack and Product Documentation](https://www.elastic.co/guide/index.html)
+ [Installation and Upgrade Guide [7.5]](https://www.elastic.co/guide/en/elastic-stack/current/index.html)
+ [Getting Started [7.5]](https://www.elastic.co/guide/en/elastic-stack-get-started/current/index.html)
+ [Glossary](https://www.elastic.co/guide/en/elastic-stack-glossary/current/index.html)
+ [Machine Learning [7.5]](https://www.elastic.co/guide/en/machine-learning/current/index.html)
+ [Elastic Common Schema (ECS) Reference [1.4]](https://www.elastic.co/guide/en/ecs/current/index.html)
+ [Azure Marketplace and Resource Manager (ARM) template [7.5]](https://www.elastic.co/guide/en/elastic-stack-deploy/current/index.html)
+ [Elastic Stack and Google Cloud’s Anthos](https://www.elastic.co/guide/en/elastic-stack-gke/current/index.html)

# Elastic Stack概述
Elastic Stack中的产品设计为一起使用，并同步发布，以简化安装和升级过程。全部系统栈包括：
+ [Beats 7.5](https://www.elastic.co/guide/en/beats/libbeat/7.5/index.html)
+ [APM Server 7.5](https://www.elastic.co/guide/en/apm/server/7.5/index.html)
+ [Elasticsearch 7.5](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/index.html)
+ [Elasticsearch Hadoop 7.5](https://www.elastic.co/guide/en/elasticsearch/hadoop/7.5/index.html)
+ [Kibana 7.5](https://www.elastic.co/guide/en/kibana/7.5/index.html)
+ [Logstash 7.5](https://www.elastic.co/guide/en/logstash/7.5/index.html)

# 安装Elastic Stack
安装Elastic Stack时，必须在整个堆栈中使用相同的版本。
例如，如果您使用的是Elasticsearch 7.5.1，则安装Beats 7.5.1，APM Server 7.5.1，Elasticsearch Hadoop 7.5.1，Kibana 7.5.1和Logstash 7.5.1。

## 安装清单
1. Elasticsearch（[安装说明](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/install-elasticsearch.html)）
2. Kibana（[安装说明](https://www.elastic.co/guide/en/kibana/7.5/install.html)）
3. Logstash（[安装说明](https://www.elastic.co/guide/en/logstash/7.5/installing-logstash.html)）
4. Beats（[安装说明](https://www.elastic.co/guide/en/beats/libbeat/7.5/getting-started.html)）
5. APM Server（[安装说明](https://www.elastic.co/guide/en/apm/server/7.0/installing.html)）
6. Elasticsearch Hadoop（[安装说明](https://www.elastic.co/guide/en/elasticsearch/hadoop/7.5/install.html)）

按此顺序安装可确保每个产品所依赖的组件均已安装到位。

## 在Elastic Cloud上安装 Elastic Stack
Elastic Cloud上的Elasticsearch Service是Elastic的官方托管Elasticsearch和Kibana产品。Elasticsearch Service在AWS和GCP上均可用。
在Elastic Cloud上安装很容易：单击即可创建一个配置为所需大小的Elasticsearch集群，无论是否具有高可用性。
X-Pack始终安装，因此您可以自动保护和监视群集。只需单击一下，即可在集群上启用Kibana，并且许多流行的插件都可以使用。

# 升级Elastic Stack
升级到新版本的Elasticsearch时，您需要升级Elastic Stack中的每个产品。

# Highlights 特性
## APM
## Beats
## Elasticsearch
## Kibana

# Breaking changes 重大更新
## APM
## Beats
## Elasticsearch
## Elasticsearch Hadoop
## Kibana
## Logstash

# Elastic Stack入门
## 首先安装核心产品
核心产品：
+ [Elasticsearch](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#install-elasticsearch)
+ [Kibana](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#install-kibana)
+ [Beats](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#install-beats)
+ [Logstash（可选）](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#install-logstash)

然后，学习如何实施使用Metricbeat收集服务器指标并将数据发送到Elasticsearch的系统监视解决方案，您可以在其中使用Kibana搜索和可视化数据。
完成基本设置后，添加Logstash进行其他解析。

## 开始前的准备
+ 有关受支持的操作系统和产品兼容性的信息， 请参阅《[Elastic Support Matrix](https://www.elastic.co/support/matrix)》。
+ 验证系统是否满足Logstash和Elasticsearch 的 [最低JVM要求](https://www.elastic.co/support/matrix#matrix_jvm)。

## 安装Elasticsearch
Elasticsearch是一个实时的分布式存储，搜索和分析引擎。它可以用于许多目的，但是它擅长的一种环境是索引半结构化数据流，例如日志或解码的网络数据包。

### linux:
```
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.1-linux-x86_64.tar.gz
tar -xzvf elasticsearch-7.5.1-linux-x86_64.tar.gz
cd elasticsearch-7.5.1
./bin/elasticsearch
```

### mac:
```
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.1-darwin-x86_64.tar.gz
tar -xzvf elasticsearch-7.5.1-darwin-x86_64.tar.gz
cd elasticsearch-7.5.1
./bin/elasticsearch
```

### win:
1. 从[Elasticsearch下载页面](https://www.elastic.co/downloads/elasticsearch)下载Elasticsearch 7.5.1 Windows zip文件 。
2. 将zip文件的内容提取到计算机上的目录中，例如 C:\Program Files。
3. 以管理员身份打开命令提示符，然后导航到包含解压缩文件的目录，例如：
```
cd C:\Program Files\elasticsearch-7.5.1
```
4. 启动Elasticsearch：
```
bin\elasticsearch.bat
```

## 检查Elasticsearch是否正常运行
要测试Elasticsearch守护程序是否已启动并正在运行，请尝试在端口9200上发送HTTP GET请求。
```
curl http://127.0.0.1:9200
```
在Windows上，如果未安装cURL，请将浏览器指向该URL。

您应该看到类似于以下内容的响应：
```
{
  "name" : "QtI5dUu",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "DMXhqzzjTGqEtDlkaMOzlA",
  "version" : {
    "number" : "7.5.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "00d8bc1",
    "build_date" : "2018-06-06T16:48:02.249996Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 安装Kibana
Kibana是一个旨在与Elasticsearch一起使用的开源分析和可视化平台。
您可以使用Kibana搜索，查看和与Elasticsearch索引中存储的数据进行交互。
您可以轻松地执行高级数据分析，并在各种图表，表格和地图中可视化数据。

建议您将Kibana与Elasticsearch安装在同一服务器上，但这不是必需的。
如果将产品安装在其他服务器上，则需要在Kibana配置文件中更改Elasticsearch服务器的URL（IP：PORT）kibana.yml，然后再启动Kibana。

### linux:
```
curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.5.1-linux-x86_64.tar.gz
tar xzvf kibana-7.5.1-linux-x86_64.tar.gz
cd kibana-7.5.1-linux-x86_64/
./bin/kibana
```

### mac:
```
curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.5.1-darwin-x86_64.tar.gz
tar xzvf kibana-7.5.1-darwin-x86_64.tar.gz
cd kibana-7.5.1-darwin-x86_64/
./bin/kibana
```

### win:
1. 从[Kibana下载页面](https://www.elastic.co/downloads/kibana)下载Kibana 7.5.1 Windows zip文件 。
2. 将zip文件的内容提取到计算机上的目录中，例如C:\Program Files。
3. 以管理员身份打开命令提示符，然后导航到包含解压缩文件的目录，例如：
```
cd C:\Program Files\kibana-7.5.1-windows
```
4. 启动Kibana：
```
bin\kibana.bat
```

## 使用Kibana
要启动Kibana Web界面，将浏览器指向端口5601。例如，http://127.0.0.1:5601 。

## 安装Beats
Beats是开源数据发送者，您可以将其作为代理安装在服务器上，以将运营数据发送到Elasticsearch。
Beats可以将数据直接发送到Elasticsearch或通过Logstash发送，您可以在其中进一步处理和增强数据。
每种Beat是单独安装的产品。下面将学习如何安装和运行 Metricbeat 收集系统指标。

各种Beats的用途：
+ Auditbeat 审核数据
+ Filebeat 日志文件
+ Functionbeat 云数据
+ Heartbeat 可用性监控
+ Journalbeat 系统日志
+ Metricbeat 系统指标
+ Packetbeat 网络流量
+ Winlogbeat Windows事件日志

## 安装Metricbeat
### linux:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.5.1-linux-x86_64.tar.gz
tar xzvf metricbeat-7.5.1-linux-x86_64.tar.gz
```

### mac:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.5.1-darwin-x86_64.tar.gz
tar xzvf metricbeat-7.5.1-darwin-x86_64.tar.gz
```

### win:
1. 从[Metricbeat下载页面](https://www.elastic.co/downloads/beats/metricbeat)下载Metricbeat Windows zip文件 。
2. 将zip文件的内容提取到中C:\Program Files。
3. 将metricbeat-7.5.1-windows目录重命名为Metricbeat。
4. 以管理员身份打开PowerShell提示符（右键单击PowerShell图标，然后选择“以管理员身份运行”）。
5. 在PowerShell提示符下，运行以下命令以将Metricbeat安装为Windows服务：
```
PS > cd 'C:\Program Files\Metricbeat'
PS C:\Program Files\Metricbeat> .\install-service-metricbeat.ps1
```

## 将系统指标发送到Elasticsearch
Metricbeat提供了预构建的模块，可用于在约5分钟内快速实施和部署系统监视解决方案，并带有示例仪表板和数据可视化。
在本节中，您将学习如何运行该system模块以从服务器上运行的操作系统和服务中收集指标。
系统模块收集系统级别的度量标准，例如CPU使用率，内存，文件系统，磁盘IO和网络IO统计信息，以及系统上运行的每个进程的顶级统计信息。

开始之前，确认Elasticsearch和Kibana正在运行，并且Elasticsearch准备好从Metricbeat接收数据。

### 从Metricbeat安装目录中，启用system模块：
mac and linux:
```
./metricbeat modules enable system
```

win:
```
PS C:\Program Files\Metricbeat> .\metricbeat.exe modules enable system
```

### 设置初始环境：
mac and linux:
```
./metricbeat setup -e
```

win:
```
PS C:\Program Files\Metricbeat> metricbeat.exe setup -e
```

### 启动Metricbeat：
mac and linux:
```
./metricbeat -e
```

win:
```
PS C:\Program Files\Metricbeat> Start-Service metricbeat
```

## 在Kibana可视化系统指标
要可视化系统指标，请打开浏览器并导航到Metricbeat系统概述仪表板：
http://localhost:5601/app/kibana#/dashboard/Metricbeat-system-overview-ecs

## 安装Logstash
Logstash是一个功能强大的工具，可与多种部署集成。
它提供了大量的插件，可帮助您解析，丰富，转换和缓冲来自各种来源的数据。
如果您的数据需要Beats中没有的其他处理，则需要将Logstash添加到您的部署中。

### linux:
```
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.5.1.tar.gz
tar -xzvf logstash-7.5.1.tar.gz
```

### mac:
```
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.5.1.tar.gz
tar -xzvf logstash-7.5.1.tar.gz
```

### win:
1. 从[Logstash下载页面](https://www.elastic.co/downloads/logstash)下载Logstash 7.5.1 Windows zip文件 。
2. 将zip文件的内容提取到计算机上的目录中，例如C:\Program Files。使用短路径（少于30个字符）以避免在Windows上遇到文件路径长度限制。

## 配置Logstash收听Beats输入
Logstash提供了用于从多种输入中读取的输入插件。
现在创建一个Logstash管道配置，该配置侦听Beats输入并将接收到的事件发送到Elasticsearch输出。

### 配置Logstash：
创建一个 Logstash 管道配置文件demo-metrics-pipeline.conf。
如果将Logstash作为deb或rpm软件包安装，请在Logstash config目录中创建文件 。
该文件必须包含：
+ 一个输入模块，将Logstash配置为在端口5044上侦听传入的Beats连接。
+ 一个输出模块，将事件导入入Elasticsearch，输出模块还配置Logstash写入Metricbeat索引。

例如：
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => "localhost:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

## 启动Logstash
使用适合您的系统的命令。
如果将Logstash作为deb或rpm软件包安装，请确保配置文件位于config目录中。
对于其他平台，config不需要目录，但这是保持一致的最佳实践。

### linux:
```
cd logstash-7.5.1
./bin/logstash -f path/to/config/demo-metrics-pipeline.conf
```
### mac:
```
cd logstash-7.5.1
./bin/logstash -f path/to/config/demo-metrics-pipeline.conf
```
### win:
```
bin\logstash.bat -f path\to\config\demo-metrics-pipeline.conf
```

## 配置Metricbeat以将事件发送到Logstash
Metricbeat默认将事件发送到Elasticsearch。要将事件发送到Logstash，请修改Metricbeat配置文件metricbeat.yml。
output.elasticsearch通过将其注释掉来禁用该部分，然后output.logstash通过取消注释来启用该部分。
```
#-------------------------- Elasticsearch output ------------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]
.
.
.
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```
保存文件，然后重新启动Metricbeat以应用配置更改。

Logstash从Beats输入中读取并将事件编入Elasticsearch。您尚未定义过滤器部分，因此Logstash只是将事件转发到Elasticsearch，而无需其他处理。

## 定义过滤器以从字段提取数据
Metricbeat收集的系统指标包括一个名为的字段cmdline ，其中包含用于启动系统进程的完整命令行参数。例如：
```
"cmdline": "/Applications/Firefox.app/Contents/MacOS/plugin-container.app/Contents/MacOS/plugin-container -childID 3
-isForBrowser -boolPrefs 36:1|299:0| -stringPrefs 285:38;{b77ae304-9f53-a248-8bd4-a243dbf2cab1}| -schedulerPrefs
0001,2 -greomni /Applications/Firefox.app/Contents/Resources/omni.ja -appomni
/Applications/Firefox.app/Contents/Resources/browser/omni.ja -appdir
/Applications/Firefox.app/Contents/Resources/browser -profile
/Users/dedemorton/Library/Application Support/Firefox/Profiles/mftvzeod.default-1468353066634
99468 gecko-crash-server-pipe.99468 org.mozilla.machname.1911848630 tab"
```

您可能只想发送命令路径，而不是将整个命令行参数发送给Elasticsearch。一种方法是使用Grok过滤器。[Grok过滤器参考文档](https://www.elastic.co/guide/en/logstash/7.5/plugins-filters-grok.html)

要提取路径，请在您之前创建的Logstash配置文件的输入和输出部分之间添加以下Grok过滤器：
```
filter {
  if [system][process] {
    if [system][process][cmdline] {
      grok {
        match => { 
          "[system][process][cmdline]" => "^%{PATH:[system][process][cmdline_path]}"
        }
        remove_field => "[system][process][cmdline]" 
      }
    }
  }
}
```
使用模式匹配路径，然后将路径存储在名为的字段中 cmdline_path。删除原始字段，cmdline因此不会在Elasticsearch中建立索引。

完成后，完整的配置文件应如下所示：
```
input {
  beats {
    port => 5044
  }
}

filter {
  if [system][process] {
    if [system][process][cmdline] {
      grok {
        match => {
          "[system][process][cmdline]" => "^%{PATH:[system][process][cmdline_path]}"
        }
        remove_field => "[system][process][cmdline]"
      }
    }
  }
}

output {
  elasticsearch {
    hosts => "localhost:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

重新启动Logstash以获取更改。现在，该事件包括一个名为的字段 cmdline_path，其中包含命令路径：
```
"cmdline_path": "/Applications/Firefox.app/Contents/MacOS/plugin-container.app/Contents/MacOS/plugin-container"
```

# Running the Elastic Stack on Docker


# Running the Elastic Stack on Kubernetes