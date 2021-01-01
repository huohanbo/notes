# 简介
Spring Data的使命是为数据访问提供熟悉且一致的基于Spring的编程模型，同时仍保留底层数据存储的特​​殊特性。
它使数据访问技术，关系数据库和非关系数据库，map-reduce框架和基于云的数据服务变得简单易用。这是一个伞形项目，其中包含许多特定于给定数据库的子项目。
这些项目是通过与这些激动人心的技术背后的许多公司和开发人员合作开发的。

# 特性
- 强大的存储库和自定义对象映射抽象
- 从存储库方法名称派生动态查询
- 实现域基类提供基本属性
- 支持透明审核（创建，最后更改）
- 可以集成自定义存储库代码
- 通过JavaConfig和自定义XML命名空间轻松实现Spring集成
- 与Spring MVC控制器的高级集成
- 跨存储持久性的实验支持

# 主要模块
Spring Data Commons - 支持每个Spring Data模块的Core Spring概念。

Spring Data JDBC - 对JDBC的Spring Data存储库支持。

Spring Data JDBC Ext - 支持标准JDBC的数据库特定扩展，包括对Oracle RAC快速连接故障转移的支持，AQ JMS支持以及对使用高级数据类型的支持。

Spring Data JPA - JPA的 Spring Data存储库支持。

Spring Data KeyValue- Map基于库和SPI轻松建立键值存储一个Spring数据模块。

Spring Data LDAP - 对Spring LDAP的 Spring Data存储库支持。

Spring Data MongoDB - 基于Spring的对象文档支持和MongoDB的存储库。

Spring Data Redis - 从Spring应用程序轻松配置和访问Redis。

Spring Data REST - 将Spring Data存储库导出为超媒体驱动的RESTful资源。

Spring Data for Apache Cassandra - 轻松配置和访问Apache Cassandra或大规模，高可用性，面向数据的Spring应用程序。

Spring Data for Apache Geode - 轻松配置和访问Apache Geode，以实现高度一致，低延迟，面向数据的Spring应用程序。

Spring Data for Apache Solr - 为面向搜索的Spring应用程序轻松配置和访问Apache Solr。

Pivotal GemFire的Spring数据 - 轻松配置和访问Pivotal GemFire，实现高度一致，低延迟/高吞吐量，面向数据的Spring应用程序。

# 社区模块
Spring Data Aerospike - Aerospike的弹簧数据模块。

Spring Data ArangoDB - ArangoDB的 Spring Data模块。

Spring Data Couchbase - Couchbase的 Spring Data模块。

Spring Data Azure Cosmos DB - 用于Microsoft Azure Cosmos DB的Spring Data模块。

Spring Data Cloud Datastore - 用于Google Datastore的Spring数据模块。

Spring Data Cloud Spanner - 适用于Google Spanner的Spring数据模块。

Spring Data DynamoDB - DynamoDB的 Spring数据模块。

Spring Data Elasticsearch - Elasticsearch的 Spring Data模块。

Spring Data Hazelcast - 为Hazelcast提供Spring Data存储库支持。

Spring Data Jest - 基于Jest REST客户端的Elasticsearch的Spring Data模块。

Spring Data Neo4j - Neo4j的基于Spring的对象图支持和存储库。

Spring Data Vault - 构建于Spring Data KeyValue之上的Vault存储库。