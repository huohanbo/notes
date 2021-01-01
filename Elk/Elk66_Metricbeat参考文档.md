# 参考资料
+ [Metricbeat Reference](https://www.elastic.co/guide/en/beats/metricbeat/current/index.html)

# Metricbeat概述
Metricbeat是一种轻量级的托运人，您可以将其安装在服务器上，以定期从操作系统和服务器上运行的服务收集指标。
Metricbeat会收集它收集的度量标准和统计信息，并将其运送到您指定的输出中，例如 Elasticsearch 或 Logstash。

Metricbeat通过从服务器上运行的系统和服务收集指标来帮助您监视服务器，例如：
+ Apache
+ HAProxy
+ MongoDB
+ MySQL
+ Nginx
+ PostgreSQL
+ Redis
+ System
+ Zookeeper

Metricbeat可以将收集的指标直接插入Elasticsearch或将其发送到Logstash，Redis或Kafka。

# Metricbeat入门
要开始使用自己的Metricbeat设置，请安装和配置以下相关产品：
+ 用于存储和索引数据的Elasticsearch。
+ 用于UI的Kibana。
+ 用于解析和增强数据的Logstash（可选）。

## 安装Metricbeat
应该将Metricbeat安装在尽可能靠近要监视的服务的位置。
例如，如果您有四台运行MySQL的服务器，则建议您在每台服务器上运行Metricbeat。
这允许Metricbeat从本地主机访问您的服务，并且不会引起任何其他网络流量，也不会阻止Metricbeat在出现网络问题时收集指标。
来自多个Metricbeat实例的指标将在Elasticsearch服务器上合并。

### linux or mac:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.5.1-linux-x86_64.tar.gz
tar xzvf metricbeat-7.5.1-linux-x86_64.tar.gz
```

### win:
1. 从[下载页面](https://www.elastic.co/downloads/beats/metricbeat)下载Metricbeat Windows zip文件 。
2. 将zip文件的内容提取到中C:\Program Files。
3. 将metricbeat-<version>-windows目录重命名为Metricbeat。
4. 以管理员身份打开PowerShell提示符（右键单击PowerShell图标，然后选择“以管理员身份运行”）。
5. 在PowerShell提示符下，运行以下命令以将Metricbeat安装为Windows服务：
```
PS > cd 'C:\Program Files\Metricbeat'
PS C:\Program Files\Metricbeat> .\install-service-metricbeat.ps1
```

## 配置Metricbeat
要配置Metricbeat，请编辑配置文件。默认配置文件称为 metricbeat.yml。
还有一个完整的示例配置文件metricbeat.reference.yml，该文件显示了所有未弃用的选项。

配置Metricbeat时，需要指定 要运行的模块。Metricbeat使用模块来收集指标。
每个模块都定义了从特定服务（例如Redis或MySQL）收集数据的基本逻辑。
一个模块由获取和构造数据的指标集组成。

### 启用要运行的模块
启用要运行的模块。如果您接受默认配置而不启用其他模块，则Metricbeat仅收集系统指标。
您可以启用modules.d目录中定义的默认模块配置 （推荐），也可以将模块配置添加到 metricbeat.yml文件中。
该modules.d目录包含所有可用Metricbeat模块的默认配置。

以下示例在目录中启用apache和mysql配置 modules.d：

mac and linux:
```
./metricbeat modules enable apache mysql
```
win:
```
PS > .\metricbeat.exe modules enable apache mysql
```

### 配置输出
Metricbeat支持多种 输出，但是通常您将事件直接发送到Elasticsearch或Logstash以进行其他处理。

要将输出直接发送到Elasticsearch（不使用Logstash），请设置Elasticsearch安装的位置：
+ 如果您 在Elastic Cloud上运行我们的 托管Elasticsearch Service，请指定您的Cloud ID。例如：
```
cloud.id: "staging:dXMtZWFzdC0xLmF3cy5mb3VuZC5pbyRjZWM2ZjI2MWE3NGJmMjRjZTMzYmI4ODExYjg0Mjk0ZiRjNmMyY2E2ZDA0MjI0OWFmMGNjN2Q3YTllOTYyNTc0Mw=="
```
+ 如果您在自己的硬件上运行Elasticsearch，请设置Metricbeat可以找到Elasticsearch安装的主机和端口。例如：
```
setup.kibana:
  host: "mykibanahost:5601" 
```

要将输出发送到Logstash，请 改为配置Logstash输出。

### 配置Kibana端点
如果您打算使用Metricbeat随附的示例Kibana仪表板，请配置Kibana端点。
如果Kibana与Elasticsearch在同一主机上运行，​​则可以跳过此步骤。
```
setup.kibana:
  host: "mykibanahost:5601" 
```

## 在Elasticsearch中加载索引模板

## 设置Kibana dashboards

## 启动Metricbeat
mac and linux:
```
sudo chown root metricbeat.yml 
sudo chown root modules.d/system.yml 
sudo ./metricbeat -e
```

win:
```
PS C:\Program Files\Metricbeat> Start-Service metricbeat
```

### 测试Metricbeat是否正常启动
使用如下命令：
```
curl -XGET 'http://localhost:9200/metricbeat-*/_search?pretty'
```
确保localhost:9200用Elasticsearch实例的地址替换。
在Windows上，如果没有安装cURL，只需将浏览器指向URL。

## 查看示例Kibana dashboards

## APT和YUM的存储库

# 设置和运行Metricbeat
## 目录布局
## 命令参考
Metricbeat提供了一个命令行界面，用于启动Metricbeat和执行常见任务，例如测试配置文件和加载仪表板。
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

## 在Docker上运行Metricbeat
## 在Kubernetes上运行Metricbeat
## Metricbeat和systemd