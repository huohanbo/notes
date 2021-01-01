# 简介
Spring Data MongoDB项目提供与MongoDB文档数据库的集成。
Spring Data MongoDB的关键功能区域是一个POJO中心模型，用于与MongoDB DBCollection交互并轻松编写Repository样式数据访问层。

# 特性
- Spring配置支持使用基于Java的@Configuration类或Mongo驱动程序实例和副本集的XML命名空间。c
- MongoTemplate助手类，可提高执行常见Mongo操作的效率。包括文档和POJO之间的集成对象映射。c
- 异常转换为Spring的可移植数据访问异常层次结构c
- 功能丰富的对象映射与Spring的转换服务集成c
- 基于注释的映射元数据，但可扩展以支持其他元数据格式c
- 持久性和映射生命周期事件c
- 使用MongoReader / MongoWriter抽象的低级映射c
- 基于Java的查询，标准和更新DSLc
- 自动实现Repository接口，包括对自定义查询方法的支持。c
- QueryDSL集成以支持类型安全查询。地理空间整合c
- Map-Reduce集成
- JMX管理和监控
- CDI对存储库的支持
- GridFS支持

# 前言
## 了解Spring
[官网](https://docs.spring.io/spring/docs/5.1.9.RELEASE/spring-framework-reference/index.html)

Spring Data使用Spring框架的核心功能，包括：IoC容器、类型转换系统、表达语言、JMX集成、DAO异常层次结构。

## 了解MongoDB
[官网](https://www.mongodb.org/)
[文档](https://docs.mongodb.org/manual/)

# 要求
Spring Data MongoDB 2.x二进制文件需要JDK 8.0及以上版本以及Spring Framework 5.1.9.RELEASE及以上版本。

就文档存储而言，至少需要2.6版本的MongoDB。