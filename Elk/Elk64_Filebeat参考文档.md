# 参考资料
+ [Filebeat Reference](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)

# Filebeat简介
Filebeat是用于转发和集中日志数据的轻量级传送程序。Filebeat是一个 Elastic Beat，它基于libbeat框架。
Filebeat作为代理（agent），安装在服务器上的，监视您指定的日志文件或位置，收集日志事件，并将它们转发到 Elasticsearch 或 Logstash 进行检索。

## Filebeat的工作方式
启动Filebeat时，它将启动一个或多个输入（inputs），这些输入将在为日志数据指定的位置中查找。
对于Filebeat所找到的每个日志，Filebeat都会启动收集器（harvester）。
每个收集器都读取单个日志以获取新内容，并将新日志数据发送到libbeat，libbeat将聚集事件并将聚集的数据发送到为Filebeat配置的输出（output）。
![filebeat](image/filebeat.png)

# Filebeat入门
要开始使用自己的Filebeat，请安装和配置以下相关产品：
+ 用于存储和索引数据的Elasticsearch。
+ 用于UI的Kibana。
+ 用于解析和增强数据的Logstash（可选）。

## 安装Filebeat
### linux:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-linux-x86_64.tar.gz
tar xzvf filebeat-7.5.1-linux-x86_64.tar.gz
```

### mac:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-darwin-x86_64.tar.gz
tar xzvf filebeat-7.5.1-darwin-x86_64.tar.gz
```

### win：
1. 从[Filebeat下载页面](https://www.elastic.co/downloads/beats/filebeat)下载Filebeat Windows zip文件 。
2. 将zip文件的内容提取到中C:\Program Files。
3. 将filebeat-<version>-windows目录重命名为Filebeat。
4. 以管理员身份打开PowerShell提示符（右键单击PowerShell图标，然后选择“以管理员身份运行”）。
5. 在PowerShell提示符下，运行以下命令以将Filebeat安装为Windows服务：
```
PS > cd 'C:\Program Files\Filebeat'
PS C:\Program Files\Filebeat> .\install-service-filebeat.ps1
```

## 配置Filebeat
要配置Filebeat，请编辑配置文件。默认配置文件称为 filebeat.yml。文件的位置因平台而异。要找到文件，请参阅目录布局。
还有一个完整的示例配置文件filebeat.reference.yml ，该文件显示了所有未弃用的选项。

Filebeat对大多数配置选项使用预定义的默认值。下面是filebeat配置文件filebeat.yml的部分示例。
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
    #- c:\programdata\elasticsearch\logs\*
```

### 配置日志文件的路径
对于最基本的Filebeat配置，可以定义具有单个路径的单个输入。例如：
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log	
```

本示例中的输入/var/log/*.log将收获路径中的所有文件，这意味着Filebeat将收获目录中/var/log/以结尾的所有文件.log。
要从预定义级别的子目录中提取所有文件，可以使用以下模式： /var/log/*/*.log。这.log将从的子文件夹中提取所有文件/var/log。它不会从/var/log文件夹本身获取日志文件。
当前，不能以递归方式获取目录所有子目录中的所有文件。

### 配置输出
Filebeat支持多种输出，但是通常您将事件直接发送到Elasticsearch或Logstash进行其他处理。

要将输出直接发送到Elasticsearch（不使用Logstash），请设置Elasticsearch安装的位置：
+ 如果在Elastic Cloud上运行我们的 托管Elasticsearch Service，请指定您的Cloud ID。例如：
```
cloud.id: "staging:dXMtZWFzdC0xLmF3cy5mb3VuZC5pbyRjZWM2ZjI2MWE3NGJmMjRjZTMzYmI4ODExYjg0Mjk0ZiRjNmMyY2E2ZDA0MjI0OWFmMGNjN2Q3YTllOTYyNTc0Mw=="
```
+ 如果您在自己的硬件上运行Elasticsearch，请设置Filebeat可以找到Elasticsearch安装的主机和端口。例如：
```
output.elasticsearch:
  hosts: ["myEShost:9200"]
```

要将输出发送到Logstash，请改为配置Logstash输出。

### 配置Kibana端点
如果打算使用Filebeat随附的示例Kibana仪表板，请配置Kibana端点。
如果Kibana与Elasticsearch在同一主机上运行，​​则可以跳过此步骤。
```
setup.kibana:
  host: "mykibanahost:5601" 
```

## 在Elasticsearch中加载索引模板

## 配置Kibana dashboards

## 启动Filebeat
### mac and linux:
```
sudo chown root filebeat.yml 
sudo ./filebeat -e
```

### win:
```
PS C:\Program Files\Filebeat> Start-Service filebeat
```
默认情况下，Windows日志文件存储在中C:\ProgramData\filebeat\Logs。

命令行启动：
```
filebeat -e -c filebeat.yml
```

## 查看示例Kibana仪表板
## 常见日志格式的模块
## APT和YUM的存储库

# 设置并运行Filebeat
## 目录布局
## 秘密密钥库
## 命令参考
Filebeat提供了一个命令行界面，用于启动Filebeat和执行常见任务，例如测试配置文件和加载仪表板。
命令行还支持 用于控制全局行为的全局标志。

### 常用命令
+ export 将配置，索引模板，ILM策略或仪表板导出到标准输出。
+ help 显示任何命令的帮助。
+ keystore 管理秘密密钥库。
+ modules 管理已配置的模块。
+ run 运行Filebeat。如果您在不指定命令的情况下启动Filebeat，则默认情况下使用此命令。
+ setup 设置初始环境，包括索引模板，ILM策略和写入别名，Kibana仪表板（如果可用）以及机器学习作业（如果可用）。
+ test 测试配置。
+ version 显示有关当前版本的信息。

### export命令
将配置，索引模板，ILM策略或仪表板导出到标准输出。
可以使用此命令快速查看您的配置，查看索引模板和ILM策略的内容，或从Kibana导出仪表板。

#### 语法
```
filebeat export SUBCOMMAND [FLAGS]
```

#### 参数SUBCOMMAND
##### config
将当前配置导出到stdout。如果使用该-c标志，则此命令将导出在指定文件中定义的配置。

##### dashboard
导出仪表板。您可以使用此选项将仪表板存储在模块中的磁盘上并自动加载。例如，要将仪表板导出到JSON文件，请运行：
```
filebeat export dashboard --id="DASHBOARD_ID" > dashboard.json
```

##### template
将索引模板导出到stdout。您可以指定--es.version和 --index标志，以进一步定义导出的内容。
此外，您可以通过定义目录--dir，将模板导出到文件中，而不是stdout。

##### ilm-policy
将索引生命周期管理策略导出到stdout。您可以指定 --es.version和--dir，策略应作为文件导出到而不是导出到stdout。

#### 参数FLAGS
##### --es.version VERSION
与template结合使用时，导出与指定版本兼容的索引模板。与ilm-policy结合使用时，如果为ILM启用了指定的ES版本，则导出ILM策略。

##### -h, --help
显示export命令帮助。

##### --index BASE_NAME
与template结合使用时，设置要用于索引模板的基本名称。如果未指定此标志，则默认基本名称为 filebeat。

##### --dir DIRNAME
定义一个目录，模板和ILM策略应作为文件导出到该目录，而不是将它们打印到stdout。

##### --id DASHBOARD_ID
与dashboard结合使用时，指定仪表板ID。

#### 示列
```
filebeat export config
filebeat export template --es.version 7.5.1 --index myindexname
filebeat export dashboard --id="a7b35890-8baa-11e8-9676-ef67484126fb" > dashboard.json
```

### help命令
显示任何命令的帮助。如果未指定命令，则显示该run命令的帮助。

#### 语法
```
filebeat help COMMAND_NAME [FLAGS]
```

#### 参数COMMAND_NAME 
指定要显示帮助的命令的名称

#### 参数FLAGS 
##### -h, --help
显示help命令帮助。

#### 示列
```
filebeat help export
```

### keystore 命令
管理秘密密钥库。

#### 语法
```
filebeat keystore SUBCOMMAND [FLAGS]
```

#### 参数 SUBCOMMAND
##### add KEY
将指定的密钥添加到密钥库。使用该--force标志覆盖现有密钥。使用该--stdin标志将值传递给stdin。

##### create
创建一个密钥库来保存机密。使用该--force标志覆盖现有的密钥库。

##### list
列出密钥库中的密钥。

##### remove KEY
从密钥库中删除指定的密钥。

#### 参数FLAGS
##### --force
与add和create子命令一起有效。与一起使用时add，将覆盖指定的键。与结合使用时create，将覆盖密钥库。

##### --stdin
与配合使用时add，将stdin用作键值的来源。

##### -h, --help
显示keystore命令帮助。

#### 示列
```
filebeat keystore create
filebeat keystore add ES_PWD
filebeat keystore remove ES_PWD
filebeat keystore list
```

### modules命令
管理已配置的模块。可以使用此命令来启用和禁用modules.d目录中定义的特定模块配置。您使用此命令所做的更改将保留并用于以后的Filebeat运行。
要查看启用和禁用的模块，请运行list子命令。

#### 语法
```
filebeat modules SUBCOMMAND [FLAGS]
```

#### 参数SUBCOMMANDS
##### disable MODULE_LIST
禁用以空格分隔的列表中指定的模块。

##### enable MODULE_LIST
启用以空格分隔的列表中指定的模块。

##### list
列出当前启用和禁用的模块。

#### 参数FLAGS
##### -h, --help
显示export命令帮助。

#### 示列
```
filebeat modules list
filebeat modules enable apache2 auditd mysql
```

### run命令（缺省）
**运行Filebeat**。如果您在不指定命令的情况下启动Filebeat，则默认情况下使用此命令。

#### 语法
```
filebeat run [FLAGS]
Or:
filebeat [FLAGS]
```

#### 参数FLAGS
##### -N, --N
禁用出于测试目的的发布。此选项禁用除File输出之外的所有输出。

##### --cpuprofile FILE
将CPU配置文件数据写入指定的文件。此选项对于排除Filebeat故障很有用。

##### -h, --help
显示run命令帮助。

##### --httpprof [HOST]:PORT
启动http服务器进行概要分析。此选项对于故障排除和分析Filebeat很有用。

##### --memprofile FILE
将内存配置文件数据写入指定的输出文件。此选项对于排除Filebeat故障很有用。

##### --modules MODULE_LIST
指定要运行的模块的逗号分隔列表。例如：
```
filebeat run --modules nginx,mysql,system
```
无需每次运行Filebeat时都指定模块列表，而是可以使用modules命令启用和禁用特定模块。然后，当您运行Filebeat时，它将运行所有已启用的模块

##### --once
使用该--once标志时，Filebeat将启动所有已配置的采集器和输入，并运行每个输入，直到采集器关闭。
如果设置了 --once标志，则还应该进行设置，close_eof以便在到达文件末尾时关闭采集器。
默认情况下，采集器在close_inactive达到后关闭 。

#### 示列
```
filebeat run -e
Or:
filebeat -e
```

### setup命令
设置初始环境，包括索引模板，ILM策略和写别名，Kibana仪表板（如果有）以及机器学习作业（如果有）
+ 索引模板可确保在Elasticsearch中正确映射字段。如果启用了索引生命周期管理，则还可以确保将定义的ILM策略和写别名连接到与索引模板匹配的索引。ILM策略负责索引的生命周期，何时进行过渡，何时将索引从热阶段移至下一阶段等。
+ Kibana仪表板使您可以更轻松地在Kibana中可视化Filebeat数据。
+ 机器学习作业包含分析数据异常所需的配置信息和元数据。

此命令无需实际运行Filebeat并提取数据就可以设置环境。

#### 语法
```
filebeat setup [FLAGS]
```

#### 参数FLAGS
##### --dashboards
设置Kibana仪表板（如果有）。此选项从Filebeat包中加载仪表板。有关更多选项，例如加载自定义的仪表板，请参阅《Beats开发人员指南》中的“ 导入现有的Beat仪表板”。

##### -h, --help
显示setup命令帮助。

##### --machine-learning
仅设置机器学习作业配置。

##### --modules MODULE_LIST
指定以逗号分隔的模块列表。当filebeat.yml文件中没有定义模块时，使用此标志来避免错误。

##### --pipelines
为配置的文件集设置接收管道。Filebeat在filebeat.yml文件中寻找启用的模块。如果使用该 modules命令启用modules.d 目录中的模块，则还要指定--modules标志。

##### --index-management
设置与Elasticsearch索引管理相关的组件，包括模板，ILM策略和写入别名（如果受支持和配置）。

##### --template
[ 7.2 ] 在7.2中弃用。仅设置索引模板。建议改为使用--index-management。

##### --ilm-policy
[ 7.2 ] 在7.2中弃用。设置索引生命周期管理策略。建议改为使用--index-management。

#### 示列
```
filebeat setup --dashboards
filebeat setup --machine-learning
filebeat setup --pipelines
filebeat setup --pipelines --modules system,nginx,mysql 
filebeat setup --index-management
```

### test命令
测试配置。

#### 语法
```
filebeat test SUBCOMMAND [FLAGS]
```

#### 参数SUBCOMMANDS
##### config
测试配置设置。

##### output
测试Filebeat是否可以使用当前设置连接到输出。

#### 参数FLAGS
##### -h, --help
显示test命令帮助。

#### 示列
```
filebeat test config
```

### version命令
显示有关当前版本的信息。

#### 语法
```
filebeat version [FLAGS]
```

#### 参数FLAGS

##### -h, --help
显示version命令帮助。

### 全局参数 FLAGS
只要运行Filebeat，这些FLAGS参数就可用。

#### -E, --E "SETTING_NAME=VALUE"
覆盖特定的配置设置。您可以指定多个替代。例如：
```
filebeat -E "name=mybeat" -E "output.elasticsearch.hosts=['http://myhost:9200']"
```
此设置适用于当前运行的Filebeat进程。Filebeat配置文件未更改。

#### -M, --M "VAR_NAME=VALUE"
覆盖Filebeat模块的默认配置。您可以指定多个变量替代。例如：
```
filebeat -modules=nginx -M "nginx.access.var.paths=['/var/log/nginx/access.log*']" -M "nginx.access.var.pipeline=no_plugins"
```

#### -c, --c FILE
指定用于Filebeat的配置文件。您在此处指定的文件是相对于的path.config。如果-c未指定标志filebeat.yml，则使用默认配置文件。

#### -d, --d SELECTORS
为指定的选择器启用调试。对于选择器，可以指定以逗号分隔的组件列表，也可以用于-d "*"为所有组件启用调试。例如，-d "publish"显示所有与“发布”相关的消息。

#### -e, --e
登录到stderr并禁用syslog /文件输出。

#### --path.config
设置配置文件的路径。有关详细信息，请参见目录布局部分。

#### --path.data
设置数据文件的路径。有关详细信息，请参见目录布局部分。

#### --path.home
设置其他文件的路径。有关详细信息，请参见目录布局部分。

#### --path.logs
设置日志文件的路径。有关详细信息，请参见目录布局部分。

#### --strict.perms
对配置文件设置严格权限检查。默认值为 -strict.perms=true。有关更多信息，请参见 Beats Platform参考中的配置文件所有权和权限。

#### -v, --v
记录INFO级别的消息。

## 在Docker上运行Filebeat
## 在Kubernetes上运行Filebeat
## Filebeat和systemd
## 停止Filebeat

# 升级Filebeat

# Filebeat如何工作

# 配置Filebeat
## 指定运行模块
Filebeat 模块为您提供了一种快速处理常见日志格式的快速方法。
它们包含默认配置，Elasticsearch接收节点管道定义和Kibana仪表板，以帮助您实施和部署日志监视解决方案。

Filebeat提供了几种不同的方式来启用模块。您可以：
+ 在modules.d目录中启用模块配置
+ 运行Filebeat时启用模块
+ 在filebeat.yml文件中启用模块配置

启用模块时，还可以 指定变量设置以更改模块的默认行为，还可以指定 高级设置来覆盖输入设置。

在启用模块的情况下运行Filebeat之前，请确保还设置了环境以使用Kibana仪表板。

### 在modules.d目录中启用模块配置
modules.d目录包含Filebeat中所有可用模块的默认配置。
您可以通过运行 modules enable 或 modules disable 命令来启用或禁用特定的模块配置。

#### 启用apache和mysql
linux or mac:
```
./filebeat modules enable apache mysql
```

win:
```
PS > .\filebeat.exe modules enable apache mysql
```
然后，当您运行Filebeat时，它将加载modules.d目录中指定的相应模块配置（例如modules.d/apache.yml和 modules.d/mysql.yml）。

#### 查看已启用和已禁用模块的列表
linux or mac:
```
./filebeat modules list
```

win:
```
PS > .\filebeat.exe modules list
```

### 运行Filebeat时启用模块
要在命令行上运行Filebeat时启用特定模块，可以使用该--modules标志。
当您开始并希望在每次运行Filebeat时指定不同的模块和设置时，此方法效果很好。
命令行中指定的所有模块将与配置文件或modules.d 目录中启用的所有模块一起加载。
**如果存在冲突，则使用在命令行中指定的配置。**

#### 启用并运行nginx，mysql 和 system 模块
linux or mac:
```
./filebeat --modules nginx,mysql,system
```

win:
```
PS > .\filebeat.exe --modules nginx,mysql,system
```

### 在filebeat.yml文件中启用模块配置
如果可以，应使用modules.d目录中的配置文件。
但是，如果您已经从Filebeat的先前版本升级，并且不想将模块配置移到目录中，则直接在配置文件中启用模块是一种实用的方法modules.d。
您可以继续配置filebeat.yml文件中的模块，但是modules由于该命令需要modules.d布局，因此您将无法使用该命令来启用和禁用配置。

要在filebeat.yml配置文件中启用特定模块，可以将条目添加到filebeat.modules列表中。列表中的每个条目均以破折号（-）开头，后跟该模块的设置。

#### 在filebeat.yml文件配置nginx，mysql 和 system 模块
```
filebeat.modules:
- module: nginx
- module: mysql
- module: system
```
默认模块配置假定您要收集的日志位于操作系统预期的位置，并且模块的行为适合您的环境。

## 指定变量设置
每个模块和文件集都有变量，您可以设置这些变量来更改模块的默认行为，包括模块在其中查找日志文件的路径。

### 可以在配置中设置变量：
```
- module: nginx
  access:
    var.paths: ["/var/log/nginx/access.log*"] 
```

### 或从命令行设置变量：
linux or mac:
```
./filebeat -e -M "nginx.access.var.paths=[/usr/local/var/log/nginx/access.log*]"
```

win:
```
PS > 。\ filebeat 。exe - e - M “ nginx.access.var.paths = [c：/programdata/nginx/logs/*access.log*]” 
```

可以指定多个替代。每个替代都必须以开头-M。
如果将Filebeat作为服务运行，则不能从命令行设置路径。您必须var.paths在模块配置文件中设置该选项。

## 配置输入（inputs）
要手动配置Filebeat，请在 filebeat.yml文件 的 filebeat.inputs 部分中**指定输入列表** 。输入指定Filebeat如何查找和处理输入数据。

该列表是一个YAML数组，因此每个输入都以破折号（-）开头。您可以指定多个输入，并且可以多次指定相同的输入类型。

### 示列：
```
filebeat.inputs:
- type: log
  paths:
    - /var/log/system.log
    - /var/log/wifi.log
- type: log
  paths:
    - "/var/log/apache2/*"
  fields:
    apache: true
  fields_under_root: true
```

### 输入（input）类型
+ Log
+ Stdin
+ Container
+ Kafka
+ Redis
+ UDP
+ Docker
+ TCP
+ Syslog
+ s3
+ NetFlow
+ Google Pub/Sub

### Log 
使用log输入从日志文件中读取行。

要配置此输入，请指定一个基于glob的列表paths ，必须对其进行爬网以查找和获取日志行。

#### 配置示例
```
filebeat.inputs:
- type: log
  paths:
    - /var/log/messages
    - /var/log/*.log
```

您可以使用额外的**配置选项**（如fields， include_lines，exclude_lines，multiline等），从这些文件中获取的行。您指定的选项将应用于此输入收集的所有文件。

要将不同的配置设置应用于不同的文件，您需要定义多个输入节：
```
filebeat.inputs:
- type: log 
  paths:
    - /var/log/system.log
    - /var/log/wifi.log
- type: log 
  paths:
    - "/var/log/apache2/*"
  fields:
    apache: true
  fields_under_root: true
```

#### 通用配置
以下所有输入均支持以下配置选项。

##### enabled
使用该enabled选项可以启用和禁用输入。默认情况下，启用设置为true。

##### tags
Filebeat包含在tags每个已发布事件的字段中的标签列表。
使用标记，可以轻松地在Kibana中选择特定事件或在Logstash中应用条件过滤。
这些标签将被添加到常规配置中指定的标签列表中。
```
filebeat.inputs:
- type: log
  . . .
  tags: ["json"]
```

##### fields
您可以指定可选字段以将其他信息添加到输出中。例如，您可以添加可用于过滤日志数据的字段。
字段可以是标量值，数组，字典或它们的任何嵌套组合。
默认情况下，您在此处指定的字段将被分组fields到输出文档中的子词典下。
要将自定义字段存储为顶级字段，请将fields_under_root选项设置为true。
如果在常规配置中声明了重复字段，则其值将被此处声明的值覆盖。
```
filebeat.inputs:
- type: log
  . . .
  fields:
    app_id: query_engine_12
```

##### fields_under_root
如果将此选项设置为true，则自定义 字段将作为顶级字段存储在输出文档中，而不是分组在fields子词典下。
如果自定义字段名称与Filebeat添加的其他字段名称冲突，则自定义字段将覆盖其他字段。

##### processors
应用于输入数据的处理器列表。

##### pipeline
要为此输入生成的事件设置的接收节点管道ID。

##### keep_null
如果将此选项设置为true，则带有null值的字段将在输出文档中发布。默认情况下，keep_null设置为false。

##### index
如果存在，此格式化的字符串将覆盖来自此输入的事件的索引（对于elasticsearch输出），或者设置raw_index事件的元数据的字段（对于其他输出）。
该字符串只能引用代理名称和版本以及事件时间戳。用于访问动态字段，使用 output.elasticsearch.index或处理器。

#### 详细配置
log输入支持以下配置选项。

##### paths
获取的基于全局的路径的列表。
例如，将所有文件从子目录的预定水平取，可以用：/var/log/*/*.log。
这将从 /var/log 的子文件夹中提取所有 .log 文件。它不会从/var/log文件夹本身获取日志文件。
可以使用可选项 recursive_glob 设置以递归方式获取目录所有子目录中的所有文件。

Filebeat为在指定路径下找到的每个文件启动收集器。可以每行指定一个path。每行以破折号（-）开头。

##### recursive_glob.enabled
启用扩展 ** 为递归glob模式。启用此功能后，** 每个路径中最右边的部分将扩展为固定数量的glob模式。
例如：/foo/** 扩展到 /foo，/foo/* ，/foo/*/* 等。
如果启用，它将单个扩展 ** 为8级深度*模式。

默认情况下启用此功能。设置recursive_glob.enabled为false以禁用它。

##### encoding
用于读取包含国际字符的数据的文件编码。示列：
+ plain：纯ASCII编码
+ utf-8或utf8：UTF-8编码
+ gbk：简体中文字符

##### exclude_lines
正则表达式列表，以匹配您要Filebeat排除的行。
Filebeat会删除列表中与正则表达式匹配的所有行。
**默认情况下，不删除任何行。空行将被忽略。**

如果还指定了**多行**设置，则在用 exclude_lines 过滤各行之前，将每条多行消息合并为一行。

以下示例将Filebeat配置为删除任何以 “DBG” 开头的行。
```
filebeat.inputs:
- type: log
  ...
  exclude_lines: ['^DBG']
```

##### include_lines
正则表达式列表，以匹配您要Filebeat包括的行。
Filebeat仅导出与列表中的正则表达式匹配的行。
**默认情况下，所有行均被导出。空行将被忽略。**

如果还指定了**多行**设置，则在用 include_lines 过滤各行之前，将每条多行消息合并为一行。

如果同时定义了 include_lines 和 exclude_lines，则Filebeat 首先执行include_lines，然后执行exclude_lines。

以下示例将Filebeat配置为导出以ERR或开头的任何行WARN：
```
filebeat.inputs:
- type: log
  ...
  include_lines: ['^ERR', '^WARN']
```

以下示例导出所有包含的日志行sometext，但以DBG（调试消息）开头的行除外：
```
filebeat.inputs:
- type: log
  ...
  include_lines: ['sometext']
  exclude_lines: ['^DBG']
```

##### harvester_buffer_size
每个收集器在获取文件时使用的缓冲区大小（以字节为单位）。默认值为16384。

##### max_bytes
一条日志消息可以具有的最大字节数。max_bytes 之后的所有字节将被丢弃而不发送。
此设置对于多行日志消息特别有用，该消息可能会很大。默认值为10MB（10485760）。

##### json
这些选项使Filebeat可以解码结构为JSON消息的日志。
Filebeat逐行处理日志，因此，只有每行有一个JSON对象时，JSON解码才有效。

解码发生在行过滤和多行之前。如果设置了 message_key 选项，则可以将JSON解码与过滤和多行结合使用。
在将应用程序日志包装在JSON对象中的情况下（例如在Docker中发生这种情况），这可能会有所帮助。

配置示例：
```
json.keys_under_root: true
json.add_error_key: true
json.message_key: log
```

必须至少指定以下设置之一才能启用JSON解析模式：
+ keys_under_root
默认情况下，解码后的JSON放置在输出文档中的“ json”键下。如果启用此设置，则将密钥复制到输出文档的顶层。默认为false。
+ overwrite_keys
如果keys_under_root和启用此设置，则在发生冲突时，来自解码的JSON对象的值将覆盖Filebeat通常添加的字段（类型，源，偏移量等）。
+ add_error_key
如果启用了此设置，则在JSON解组错误或message_key在配置中定义a 但无法使用的情况下，Filebeat将添加“ error.message”和“ error.type：json”键。
+ message_key
可选的配置设置，它指定要在其上应用行过滤和多行设置的JSON密钥。如果指定了密钥，则该密钥必须位于JSON对象的顶层，并且与该密钥关联的值必须是字符串，否则将不会进行过滤或多行聚合。
+ document_id
选项配置设置，指定用于设置文档ID的JSON密钥。如果已配置，则该字段将从原始json文档中删除并存储在@metadata.id
+ ignore_decoding_error
可选配置设置，指定是否应记录JSON解码错误。如果设置为true，则不会记录错误。默认为false。

##### multiline
控制Filebeat如何处理跨多行的日志消息的选项。

Filebeat收集的文件可能包含跨越多行文本的消息。例如，多行消息在包含Java堆栈跟踪的文件中很常见。
为了正确处理这些多行事件，您需要multiline在filebeat.yml文件中配置设置以指定哪些行是单个事件的一部分。

下面的示例显示如何配置Filebeat以处理多行消息，其中消息的第一行以方括号（[）开头：
```
multiline.pattern: '^\['
multiline.negate: true
multiline.match: after
```

multiline可选配置项：
+ multiline.pattern
指定要匹配的正则表达式模式。请注意，Filebeat支持的正则表达式模式与Logstash支持的模式有些不同。
+ multiline.negate
定义是否否定模式。默认值为false。
+ multiline.match
指定Filebeat如何将匹配的行组合到事件中。设置为after或before。
+ multiline.flush_pattern
指定一个正则表达式，其中将从内存中刷新当前多行，结束多行消息。
+ multiline.max_lines
可以合并为一个事件的最大行数。如果多行消息包含多个max_lines，则所有其他行都将被丢弃。默认值为500。
+ multiline.timeout
在指定的超时后，即使未找到新的模式来启动新事件，Filebeat也会发送多行事件。默认值为5秒。

##### exclude_files
与您希望Filebeat忽略的文件匹配的正则表达式列表。默认情况下，不排除任何文件。

以下示例将Filebeat配置为忽略所有具有gz扩展名的文件：
```
filebeat.inputs:
- type: log
  ...
  exclude_files: ['\.gz$']
```

##### ignore_older
如果启用此选项，Filebeat将忽略在指定时间范围之前修改的所有文件。
ignore_older如果长时间保留日志文件，则配置特别有用。
例如，如果要启动Filebeat，但只想发送最新的文件和上周的文件，则可以配置此选项。

可以使用2h（2小时）和5m（5分钟）之类的时间字符串。默认值为0，这将禁用设置。注释掉配置与将其设置为0具有相同的效果。

##### close_*
close_*配置选项用于之后的某一标准或时间以关闭采集器。
关闭采集器意味着关闭文件处理程序。如果在关闭采集器之后更新文件，则文件将在scan_frequency经过后再次被拾取。

##### close_inactive
启用此选项后，如果在指定时间内未收获文件，则Filebeat将关闭文件句柄。
当采集器读取了最后一条日志行时，定义周期的计数器开始计数。
它不是基于文件的修改时间。如果关闭的文件再次更改，则将启动新的采集器，并且在scan_frequency经过之后将获取最新的更改 。

##### close_renamed
启用此选项后，Filebeat在重命名文件时关闭文件采集器。例如，在旋转文件时会发生这种情况。
默认情况下，采集器保持打开状态并继续读取文件，因为采集器不依赖于文件名。

如果启用了close_renamed选项，并且文件的重命名或移动方式与为其指定的文件模式不再匹配，则不会再次提取该文件。

##### close_removed
启用此选项后，Filebeat将在删除文件时关闭采集器。
通常，仅当文件在所指定的时间内处于非活动状态后才应将其删除close_inactive。
但是，如果文件过早删除而您未启用close_removed，则Filebeat会将文件保持打开状态以确保采集器已经完成。
如果此设置导致文件由于过早从磁盘中删除而无法完全读取，请禁用此选项。

##### close_eof
仅当您了解数据丢失是潜在的副作用时，才使用此选项。

启用此选项后，一旦到达文件末尾，Filebeat就会关闭文件。
当您的文件仅写入一次且不时更新时，此功能很有用。
例如，当您将每个日志事件写入新文件时，就会发生这种情况。默认情况下禁用此选项。

##### close_timeout
启用此选项后，Filebeat会为每个采集器提供预定义的生存期。
无论阅读器在文件中的何处，经过close_timeout一段时间后，阅读都会停止。
当您只想在文件上花费预定义的时间时，此选项对于较旧的日志文件很有用。
虽然close_timeout将在预定义的超时后关闭文件，但是如果文件仍在更新中，
则Filebeat将根据定义的再次启动新的采集器scan_frequency。
并且此采集器的close_timeout将以超时倒数再次开始。

##### clean_*
clean_*选项用于清除注册表文件中的状态条目。这些设置有助于减小注册表文件的大小，并可以防止潜在的inode重用问题。

##### clean_inactive
仅当您了解数据丢失是潜在的副作用时，才使用此选项。

启用此选项后，经过指定的非活动时间后，Filebeat会删除文件的状态。
仅当Filebeat已忽略该文件（文件早于ignore_older）时，才能删除状态 。
该 clean_inactive 设置必须大于 ignore_older + scan_frequency，要确保在仍在收集文件的同时不删除任​​何状态。
否则，该设置可能导致Filebeat不断重新发送全部内容，因为 clean_inactive删除了Filebeat仍检测到的文件的状态。
如果文件已更新或再次出现，则会从头开始读取文件。

该clean_inactive配置选项是有用的，以减少注册表文件的大小，特别是如果每天都在产生大量的新文件。

##### clean_removed
启用此选项后，如果在磁盘上找不到以最后一个已知名称命名的文件，则Filebeat将从注册表中清除文件。
这意味着在收割机完成后重命名的文件也将被删除。默认情况下启用此选项。

如果共享驱动器在短时间内消失并再次出现，则将从头开始再次读取所有文件，因为状态已从注册表文件中删除。
在这种情况下，建议您禁用该clean_removed 选项。

如果还禁用，则必须禁用此选项close_removed。

##### scan_frequency
Filebeat多久检查一次指定用于收集的路径中的新文件。
例如，如果您指定一个glob如/var/log/*，则使用所指定的频率扫描目录中的文件 scan_frequency。
指定1可以尽可能频繁地扫描目录，而不会导致Filebeat频繁扫描。我们不建议设置此值<1s。

如果您需要近乎实时地发送日志行，请不要设置太低， scan_frequency而要进行调整，
close_inactive以使文件处理程序保持打开状态并不断轮询文件。

默认设置为10秒。

##### scan.sort
此功能是试验性的，在将来的版本中可能会完全更改或删除。
Elastic会尽力解决所有问题，但是实验性功能不受官方GA功能的支持SLA约束。

如果您为此设置指定了空字符串以外的其他值，则可以使用确定使用升序还是降序scan.order。
可能的值为modtime和filename。要按文件修改时间排序，请使用modtime，否则请使用filename。将此选项留空可将其禁用。

如果为此设置指定一个值，则可以scan.order用来配置文件是按升序还是降序扫描。

默认设置为禁用。

##### scan.order
此功能是试验性的，在将来的版本中可能会完全更改或删除。
Elastic会尽力解决所有问题，但是实验性功能不受官方GA功能的支持SLA约束。

scan.sort设置为无时，指定使用升序还是降序。可能的值为asc或desc。

默认设置为asc。

##### tail_files

##### symlinks

##### backoff
##### max_backoff
##### backoff_factor
##### harvester_limit

## 管理多行消息
## 指定常规设置
## 加载外部配置文件
## 配置内部队列
## 配置输出
通过在filebeat.yml配置文件的 Outputs 部分中设置选项，将Filebeat配置为写入特定输出。**只能定义一个输出**。

Filebeat 支持一下输出：
+ Elasticsearch
+ Logstash
+ Kafka
+ Redis
+ File
+ Console
+ Elastic Cloud

### Kafka
Kafka输出将事件发送到Apache Kafka。

#### 配置示例
```
output.kafka:
  #initial brokers for reading cluster metadata
  hosts: ["kafka1:9092", "kafka2:9092", "kafka3:9092"]

  #message topic selection + partitioning
  topic: '%{[fields.log_topic]}'
  partition.round_robin:
    reachable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
```

#### 兼容性
此输出适用于0.11和2.1.0之间的所有Kafka版本。较旧的版本也可以使用，但不受支持。

## 配置索引生命周期管理
## 负载均衡输出主机
## 指定SSL设置
## 过滤并增强导出的数据
## 使用摄取节点解析数据
## 利用geoIP信息丰富事件
## 配置项目路径
## 配置Kibana端点
## 加载Kibana仪表板
## 加载Elasticsearch索引模板
## 配置日志记录
## 在配置中使用环境变量
## 自动发现
## YAML提示和陷阱
## 正则表达式支持
## HTTP端点
## filebeat.reference.yml


# Beats central management
# Modules
# Exported fields
# Monitoring Filebeat
# Securing Filebeat
# Troubleshooting
# Contributing to Beats

```

```