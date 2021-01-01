# 参考资料
+ [Dubbo官网](http://dubbo.apache.org/zh-cn/)
+ [Dubbo文档](http://dubbo.apache.org/zh-cn/docs/user/quick-start.html)

# Dubbo简介
## 背景
随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

![dubbo-architecture-roadmap](image/dubbo-architecture-roadmap.jpg)

### 单一应用架构 (All in One) (ORM)
当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。
此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

### 垂直应用架构 (Vertical Application) (MVC)
当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，提升效率的方法之一是将应用拆成互不相干的几个应用，以提升效率。
此时，用于加速前端页面开发的Web框架(MVC)是关键。

### 分布式服务架构 (Distributed service) (RPC)
当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。
此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

### 流动计算架构 (Elastic Computing) (SOA)
当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。
此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

## 需求
![dubbo-service-governance](image/dubbo-service-governance.jpg)

在大规模服务化之前，应用可能只是通过 RMI 或 Hessian 等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过 F5 等硬件进行负载均衡。

当服务越来越多时，服务 URL 配置管理变得非常困难，F5 硬件负载均衡器的单点压力也越来越大。 此时需要一个服务注册中心，动态地注册和发现服务，使服务的位置透明。
并通过在消费方获取服务提供方地址列表，实现软负载均衡和 Failover，降低对 F5 硬件负载均衡器的依赖，也能减少部分成本。

当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。 
这时，需要自动画出应用间的依赖关系图，以帮助架构师理清关系。

接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？ 
为了解决这些问题，第一步，要将服务现在每天的调用量，响应时间，都统计出来，作为容量规划的参考指标。
其次，要可以动态调整权重，在线上，将某台机器的权重一直加大，并在加大的过程中记录响应时间的变化，直到响应时间到达阈值，记录此时的访问量，再以此访问量乘以机器数反推总容量。

以上是 Dubbo 最基本的几个需求。

## 架构
![dubbo-architecture](image/dubbo-architecture.jpg)

**节点角色说明：**

|节点		|角色说明							|
|--	|--	|
|Provider	|暴露服务的服务提供方					|
|Consumer	|调用远程服务的服务消费方				|
|Registry	|服务注册与发现的注册中心				|
|Monitor	|统计服务的调用次数和调用时间的监控中心	|
|Container	|服务运行容器							|

**调用关系说明：**
1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。

### 连通性
+ 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
+ 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
+ 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
+ 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
+ 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
+ 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
+ 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
+ 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

### 健壮性
+ 监控中心宕掉不影响使用，只是丢失部分采样数据
+ 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
+ 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
+ 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
+ 服务提供者无状态，任意一台宕掉后，不影响使用
+ 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

### 伸缩性
+ 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
+ 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

### 升级性
当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力。下图是未来可能的一种架构：

![dubbo-architecture-future](image/dubbo-architecture-future.jpg)

|节点		|角色说明								|
|--	|--	|
|Deployer	|自动部署服务的本地代理					|
|Repository	|仓库用于存储服务应用发布包				|
|Scheduler	|调度中心基于访问压力自动增减服务提供者		|
|Admin		|统一管理控制台							|
|Registry	|服务注册与发现的注册中心					|
|Monitor	|统计服务的调用次数和调用时间的监控中心		|

## 用法
### 本地服务 Spring 配置
local.xml:
```
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” />
<bean id=“xxxAction” class=“com.xxx.XxxAction”>
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

### 远程服务 Spring 配置
在本地服务的基础上，只需做简单配置，即可完成远程化：

将上面的 local.xml 配置拆分成两份，将服务定义部分放在服务提供方 remote-provider.xml，将服务引用部分放在服务消费方 remote-consumer.xml。
并在提供方增加暴露服务配置 <dubbo:service>，在消费方增加引用服务配置 <dubbo:reference>。

remote-provider.xml:
```
<!-- 和本地服务一样实现远程服务 -->
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> 
<!-- 增加暴露远程服务配置 -->
<dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” /> 
```

remote-consumer.xml:
```
<!-- 增加引用远程服务配置 -->
<dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
<!-- 和本地服务一样使用远程服务 -->
<bean id=“xxxAction” class=“com.xxx.XxxAction”> 
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

# 快速开始
Dubbo 采用全 Spring 配置方式，透明化接入应用，对应用没有任何 API 侵入，只需用 Spring 加载 Dubbo 的配置即可，Dubbo 基于 Spring 的 **Schema 扩展** 进行加载。

如果不想使用 Spring 配置，可以通过 **API 的方式** 进行调用。

## 服务提供者

### 定义服务接口
DemoService.java：
```
package org.apache.dubbo.demo;

public interface DemoService {
    String sayHello(String name);
}
```
该接口需单独打包，在服务提供方和消费方共享。

### 在服务提供方实现接口
DemoServiceImpl.java：
```
package org.apache.dubbo.demo.provider;
 
import org.apache.dubbo.demo.DemoService;
 
public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```
对服务消费方隐藏实现。

### 用 Spring 配置声明暴露服务
provider.xml：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

### 加载 Spring 配置
Provider.java：
```
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://10.20.160.198/wiki/display/dubbo/provider.xml"});
        context.start();
        System.in.read(); // 按任意键退出
    }
}
```

## 服务消费者

### 通过 Spring 配置引用远程服务
consumer.xml：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
</beans>
```

### 加载Spring配置，并调用远程服务
Consumer.java：
```
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.apache.dubbo.demo.DemoService;

public class Consumer {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://10.20.160.198/wiki/display/dubbo/consumer.xml"});
        context.start();
        DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
        String hello = demoService.sayHello("world"); // 执行远程方法
        System.out.println( hello ); // 显示调用结果
    }
}
```
也可以使用 IoC 注入。

# 依赖
## 必须依赖
JDK 1.6+

## 缺省依赖
通过 mvn dependency:tree > dep.log 命令分析，Dubbo 缺省依赖以下三方库：
```
[INFO] +- com.alibaba:dubbo:jar:2.5.9-SNAPSHOT:compile
[INFO] |  +- org.springframework:spring-context:jar:4.3.10.RELEASE:compile
[INFO] |  +- org.javassist:javassist:jar:3.21.0-GA:compile
[INFO] |  \- org.jboss.netty:netty:jar:3.2.5.Final:compile
```

这里所有依赖都是换照 Dubbo 缺省配置选的，这些缺省值是基于稳定性和性能考虑的。
+ javassist.jar : 如果 <dubbo:provider proxy="jdk" /> 或 <dubbo:consumer proxy="jdk" />，以及 <dubbo:application compiler="jdk" />，则不需要。
+ spring-context.jar : 如果用 ServiceConfig 和 ReferenceConfig 的 API 调用，则不需要。
+ netty.jar : 如果 <dubbo:protocol server="mina"/> 或 <dubbo:protocol server="grizzly"/>，则换成 mina.jar 或 grizzly.jar。如果 <protocol name="rmi"/>，则不需要。

## 可选依赖
以下依赖，在主动配置使用相应实现策略时用到，需自行加入依赖。
+ netty-all 4.0.35.Final
+ mina: 1.1.7
+ grizzly: 2.1.4
+ httpclient: 4.5.3
+ hessian_lite: 3.2.1-fixed
+ fastjson: 1.2.31
+ zookeeper: 3.4.9
+ jedis: 2.9.0
+ xmemcached: 1.3.6
+ hessian: 4.0.38
+ jetty: 6.1.26
+ hibernate-validator: 5.4.1.Final
+ zkclient: 0.2
+ curator: 2.12.0
+ cxf: 3.0.14
+ thrift: 0.8.0
+ servlet: 3.0
+ validation-api: 1.1.0.GA
+ jcache: 1.0.0
+ javax.el: 3.0.1-b08
+ kryo: 4.0.1
+ kryo-serializers: 0.42
+ fst: 2.48-jdk-6
+ resteasy: 3.0.19.Final
+ tomcat-embed-core: 8.0.11
+ slf4j: 1.7.25
+ log4j: 1.2.16

# 成熟度

## 功能成熟度
|Feature	|Maturity	|Strength	|Problem	|Advise		|User																	|
|--	|--	|--	|--	|--	|--	|
|并发控制	|Tested	|并发控制	|			|试用		|																		|
|连接控制	|Tested	|连接数控制	|			|试用		|																		|
|直连提供者	|Tested	|点对点直连服务提供方，用于测试	|			|测试环境使用|Alibaba																|
|分组聚合	|Tested	|分组聚合返回值，用于菜单聚合等服务	|特殊场景使用|可用于生产环境|																		|
|参数验证	|Tested	|参数验证，JSR303验证框架集成	|对性能有影响|试用		|LaiWang																|
|结果缓存	|Tested	|结果缓存，用于加速请求	|			|试用		|																		|
|泛化引用	|Stable	|泛化调用，无需业务接口类进行远程调用，用于测试平台，开放网关桥接等	|			|可用于生产环境|Alibaba																|
|泛化实现	|Stable	|泛化实现，无需业务接口类实现任意接口，用于Mock平台	|			|可用于生产环境|Alibaba																|
|回声测试	|Tested	|回声测试	|			|试用		|																		|
|隐式传参	|Stable	|附加参数	|			|可用于生产环境|																		|
|异步调用	|Tested	|不可靠异步调用	|			|试用		|																		|
|本地调用	|Tested	|本地调用	|			|试用		|																		|
|参数回调	|Tested	|参数回调	|特殊场景使用|试用		|Registry																|
|事件通知	|Tested	|事件通知，在远程调用执行前后触发	|			|试用		|																		|
|本地存根	|Stable	|在客户端执行部分逻辑	|			|可用于生产环境|Alibaba																|
|本地伪装	|Stable	|伪造返回结果，可在失败时执行，或直接执行，用于服务降级	|需注册中心支持|可用于生产环境|Alibaba																|
|延迟暴露	|Stable	|延迟暴露服务，用于等待应用加载warmup数据，或等待spring加载完成	|			|可用于生产环境|Alibaba																|
|延迟连接	|Tested	|延迟建立连接，调用时建立	|			|试用		|Registry																|
|粘滞连接	|Tested	|粘滞连接，总是向同一个提供方发起请求，除非此提供方挂掉，再切换到另一台	|			|试用		|Registry																|
|令牌验证	|Tested	|令牌验证，用于服务授权	|需注册中心支持|试用		|																		|
|路由规则	|Tested	|动态决定调用关系	|需注册中心支持|试用		|																		|
|配置规则	|Tested	|动态下发配置，实现功能的开关	|需注册中心支持|试用		|																		|
|访问日志	|Tested	|访问日志，用于记录调用信息	|本地存储，影响性能，受磁盘大小限制|试用		|																		|
|分布式事务	|Research	|JTA/XA三阶段提交事务	|不稳定	|不可用	|	|

## 策略成熟度
### 注册中心
|Feature	|Maturity	|Strength	|Problem			|Advise		|User																				|
|--	|--	|--	|--	|--	|--	|
|Zookeeper注册中心	|Stable	|支持基于网络的集群方式，有广泛周边开源产品，建议使用dubbo-2.3.3以上版本（推荐使用）	|依赖于Zookeeper的稳定性|可用于生产环境|																					|
|Redis注册中心	|Stable	|支持基于客户端双写的集群方式，性能高	|要求服务器时间同步，用于检查心跳过期脏数据|可用于生产环境|																					|
|Multicast注册中心	|Tested	|去中心化，不需要安装注册中心	|依赖于网络拓扑和路由，跨机房有风险|小规模应用或开发测试环境|																					|
|Simple注册中心		|Tested		|Dogfooding，注册中心本身也是一个标准的RPC服务										|没有集群支持，可能单点故障	|试用	|		|

### 监控中心
|Feature	|Maturity	|Strength		|Problem	|Advise					|User												|
|--	|--	|--	|--	|--	|--	|
|Simple监控中心	|Stable		|支持JFreeChart统计报表	|没有集群支持，可能单点故障，但故障后不影响RPC运行	|可用于生产环境	|		|

### 通信协议
|Feature	|Maturity	|Strength		|Problem	|Advise					|User												|
|--	|--	|--	|--	|--	|--	|
|Dubbo协议	|Stable	|采用NIO复用单一长连接，并使用线程池并发处理请求，减少握手和加大并发效率，性能较好（推荐使用）	|在大文件传输时，单一连接会成为瓶颈|可用于生产环境|Alibaba																						|
|Rmi协议	|Stable	|可与原生RMI互操作，基于TCP协议	|偶尔会连接失败，需重建Stub|可用于生产环境|Alibaba																						|
|Hessian协议|Stable	|可与原生Hessian互操作，基于HTTP协议			|需hessian.jar支持，http短连接的开销大		|	可用于生产环境	|		|

### Transporter

### Serialization

### ProxyFactory

### Cluster

### LoadBalance

### 路由规则

### Container

# 配置

## XML配置
provider.xml
```
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

consumer.xml示例
```
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

所有标签都支持自定义参数，用于不同扩展点实现的特殊配置，如：
```
<dubbo:protocol name="jms">
    <dubbo:parameter key="queue" value="your_queue" />
</dubbo:protocol>
```
或
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">  
    
	<dubbo:protocol name="jms" p:queue="your_queue" />  
</beans>
```
2.1.0 开始支持，注意声明：xmlns:p="http://www.springframework.org/schema/p"。

### 配置之间的关系
![dubbo-config](image/dubbo-config.jpg)

|标签					|用途			|解释																							|
|--	|--	|--	|
|<dubbo:application/>	|应用配置		|用于配置当前应用信息，不管该应用是提供者还是消费者												|
|<dubbo:module/>		|模块配置		|用于配置当前模块信息，可选																		|
|<dubbo:registry/>		|注册中心配置	|用于配置连接注册中心相关信息																		|
|<dubbo:monitor/>		|监控中心配置	|用于配置连接监控中心相关信息，可选										|
|<dubbo:provider/>		|提供方配置		|当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选							|
|<dubbo:protocol/>		|协议配置		|用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受	
|<dubbo:service/>		|服务配置		|用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心		|
|<dubbo:consumer/>		|消费方配置		|当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选											|
|<dubbo:reference/> 	|引用配置		|用于创建一个远程服务代理，一个引用可以指向多个注册中心												|
|<dubbo:method/>		|方法配置		|用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息										|
|<dubbo:argument/>		|参数配置		|用于指定方法参数配置																			|

### 不同粒度配置的覆盖关系
以 timeout 为例，下图显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：
+ 方法级优先，接口级次之，全局配置再次之。
+ 如果级别一样，则消费方优先，提供方次之。

![dubbo-config-override](image/dubbo-config-override.jpg)
（建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置）。

理论上 ReferenceConfig 中除了interface这一项，其他所有配置项都可以缺省不配置，框架会自动使用ConsumerConfig，ServiceConfig, ProviderConfig等提供的缺省配置。

引用缺省是延迟初始化的，只有引用被注入到其它 Bean，或被 getBean() 获取，才会初始化。如果需要饥饿加载，即没有人引用也立即生成动态代理，可以配置：
```
<dubbo:reference ... init="true" />
```

## 属性配置
如果应用足够简单，例如，不需要多注册中心或多协议，并且需要在spring容器中共享配置，可以直接使用 dubbo.properties作为默认配置。

Dubbo可以自动加载classpath根目录下的dubbo.properties，但是你同样可以使用JVM参数来指定路径：-Ddubbo.properties.file=xxx.properties。

### 映射规则
可以将xml的tag名和属性名组合起来，用‘.’分隔。每行一个属性。
+ dubbo.application.name=foo 相当于 <dubbo:application name="foo" />
+ dubbo.registry.address=10.20.153.10:9090 相当于 <dubbo:registry address="10.20.153.10:9090" />

如果在xml配置中有超过一个的tag，那么你可以使用‘id’进行区分。如果你不指定id，它将作用于所有tag。
+ dubbo.protocol.rmi.port=1099 相当于 <dubbo:protocol id="rmi" name="rmi" port="1099" />
+ dubbo.registry.china.address=10.20.153.10:9090 相当于 <dubbo:registry id="china" address="10.20.153.10:9090" />

如下，是一个典型的dubbo.properties配置样例。
```
dubbo.application.name=foo
dubbo.application.owner=bar
dubbo.registry.address=10.20.153.10:9090
```

### 重写与优先级
![dubbo-properties-override](image/dubbo-properties-override.jpg)

优先级从高到低：
+ JVM -D参数，当你部署或者启动应用时，它可以轻易地重写配置，比如，改变dubbo协议端口；
+ XML, XML中的当前配置会重写dubbo.properties中的；
+ Properties，默认配置，仅仅作用于以上两者没有配置时。

如果在classpath下有超过一个dubbo.properties文件，比如，两个jar包都各自包含了dubbo.properties，dubbo将随机选择一个加载，并且打印错误日志。

如果 id没有在protocol中配置，将使用name作为默认属性。

## API配置
API 属性与配置项一对一，比如：
ApplicationConfig.setName("xxx") 对应 <dubbo:application name="xxx" /> 

### 服务提供者
```
import org.apache.dubbo.rpc.config.ApplicationConfig;
import org.apache.dubbo.rpc.config.RegistryConfig;
import org.apache.dubbo.rpc.config.ProviderConfig;
import org.apache.dubbo.rpc.config.ServiceConfig;
import com.xxx.XxxService;
import com.xxx.XxxServiceImpl;
 
// 服务实现
XxxService xxxService = new XxxServiceImpl();
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("xxx");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 服务提供者协议配置
ProtocolConfig protocol = new ProtocolConfig();
protocol.setName("dubbo");
protocol.setPort(12345);
protocol.setThreads(200);
 
// 注意：ServiceConfig为重对象，内部封装了与注册中心的连接，以及开启服务端口
 
// 服务提供者暴露服务配置
ServiceConfig<XxxService> service = new ServiceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接，请自行缓存，否则可能造成内存和连接泄漏
service.setApplication(application);
service.setRegistry(registry); // 多个注册中心可以用setRegistries()
service.setProtocol(protocol); // 多个协议可以用setProtocols()
service.setInterface(XxxService.class);
service.setRef(xxxService);
service.setVersion("1.0.0");
 
// 暴露及注册服务
service.export();
```

### 服务消费者
```
import org.apache.dubbo.rpc.config.ApplicationConfig;
import org.apache.dubbo.rpc.config.RegistryConfig;
import org.apache.dubbo.rpc.config.ConsumerConfig;
import org.apache.dubbo.rpc.config.ReferenceConfig;
import com.xxx.XxxService;
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("yyy");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 注意：ReferenceConfig为重对象，内部封装了与注册中心的连接，以及与服务提供方的连接
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
reference.setApplication(application);
reference.setRegistry(registry); // 多个注册中心可以用setRegistries()
reference.setInterface(XxxService.class);
reference.setVersion("1.0.0");
 
// 和本地bean一样使用xxxService
XxxService xxxService = reference.get(); // 注意：此代理对象内部封装了所有通讯细节，对象较重，请缓存复用
```

### 方法级设置
```
// 方法级配置
List<MethodConfig> methods = new ArrayList<MethodConfig>();
MethodConfig method = new MethodConfig();
method.setName("createXxx");
method.setTimeout(10000);
method.setRetries(0);
methods.add(method);
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
...
reference.setMethods(methods); // 设置方法级配置
```

### 点对点直连
```
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
// 如果点对点直连，可以用reference.setUrl()指定目标地址，设置url后将绕过注册中心，
// 其中，协议对应provider.setProtocol()的值，端口对应provider.setPort()的值，
// 路径对应service.setPath()的值，如果未设置path，缺省path为接口名
reference.setUrl("dubbo://10.20.130.230:20880/com.xxx.XxxService"); 
```

## 注解配置
需要 2.6.3 及以上版本支持。

## 动态配置中心

## 配置加载流程

## 自动加载环境变量

# 示列
## 启动时检查
## 集群容错
## 负载均衡
## 线程模型
## 直连提供者
## 只订阅
## 只注册
## 静态服务
## 多协议
## 多注册中心
## 服务分组
## 多版本
## 分组聚合
## 参数验证
## 结果缓存
## 泛化引用
## 泛化实现
## 回声测试
## 上下文信息
## 隐式参数
## Consumer异步调用
## Provider异步执行
## 本地调用
## 参数回调
## 事件通知
## 本地存根
## 本地伪装
## 延迟暴露
## 并发控制
## 连接控制
## 延迟连接
## 粘滞连接
## 令牌验证
## 路由规则
## 配置规则
## 服务降级
## 优雅停机
## 主机绑定
## 日志适配
## 访问日志
## 服务容器
## Reference Config 缓存
## 分布式事务
## 线程栈自动dump
## Netty4
## Kryo和FST序列化
## 简化注册中心URL