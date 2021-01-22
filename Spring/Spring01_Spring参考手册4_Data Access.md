# 参考资料
> [Spring参考文档](https://docs.spring.io/spring-framework/docs/current/reference/html/)
> [Data Access](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#spring-data-tier)

--------------------------------------------------
参考文档的这一部分涉及数据访问以及数据访问层与业务或服务层之间的交互。

详细介绍了Spring全面的事务管理支持，然后全面介绍了Spring框架所集成的各种数据访问框架和技术。

# 事物管理
全面的事务支持是使用Spring Framework的最令人信服的原因。

Spring框架为事务管理提供了一致的抽象，具有以下优点：
+ 跨不同事务API的一致编程模型，例如Java事务API（Java Transaction API、JTA），JDBC，Hibernate和Java Persistence API（JPA）。
+ 支持声明式事务管理。
+ 与诸如JTA之类的复杂事务API相比，用于程序化事务管理的API更简单。
+ 与Spring的数据访问抽象的出色集成。

以下各节描述了Spring Framework的事务功能和技术：
+ Spring框架的事务支持模型的优点描述了为什么您将使用Spring框架的事务抽象而不是EJB容器管理的事务（CMT）或选择通过诸如Hibernate之类的专有API驱动本地事务的原因。
+ 了解Spring Framework事务抽象 概述了核心类，并描述了如何DataSource 从各种来源配置和获取实例。
+ 将资源与事务同步描述了应用程序代码如何确保正确创建，重用和清理资源。
+ 声明式事务管理描述了对声明式事务管理的支持。
+ 程序化事务管理涵盖对程序化（即，显式编码）事务管理的支持。
+ 事务绑定事件描述了如何在事务中使用应用程序事件。

本章还讨论了最佳实践， 应用程序服务器集成以及常见问题的解决方案。

## Spring的事务模型的优点
传统上，Java EE开发人员在事务管理中有两种选择：全局或本地事务，两者都有很大的局限性。

### 全局事物
全局事务使您可以使用多个事务资源，通常是关系数据库和消息队列。
应用服务器通过JTA管理全局事务，该JTA是繁琐的API（部分是由于其异常模型）。
此外，UserTransaction通常需要从JNDI派生JTA ，这意味着您还需要使用JNDI才能使用JTA。
全局事务的使用限制了应用程序代码的任何潜在重用，因为JTA通常仅在应用程序服务器环境中可用。

以前，使用全局事务的首选方法是通过EJB CMT（容器管理的事务）。
CMT是声明式事务管理的一种形式（与程序性事务管理不同）。
尽管使用EJB本身需要使用JNDI，但是EJB CMT消除了与事务相关的JNDI查找的需要。
它消除了编写Java代码来控制事务的大部分（但不是全部）需求。重大缺点是CMT与JTA和应用程序服务器环境相关联。
此外，它是唯一可用的，如果一个人选择使用EJB实现业务逻辑（或至少一个事务EJB门面后面）。
通常，EJB的负面影响是如此之大，以至于这不是一个有吸引力的主张，尤其是面对声明式事务管理的强制选择时。

### 本地事物
本地事务是特定于资源的，例如与JDBC连接关联的事务。
本地事务可能更易于使用，但有一个明显的缺点：它们不能跨多个事务资源工作。
例如，使用JDBC连接管理事务的代码不能在全局JTA事务中运行。由于应用程序服务器不参与事务管理，因此它无法帮助确保多个资源之间的正确性。
另一个缺点是本地事务侵入了编程模型。

### Spring框架的一致编程模型
Spring解决了全局事物和本地事物的弊端。

它使应用程序开发人员可以在任何环境中使用一致的编程模型，只需编写一次代码，它就可以从不同环境中的不同事务管理策略中受益。

Spring框架提供了声明式和程序化事务管理。大多数用户喜欢声明式事务管理，在大多数情况下我们建议这样做。
通过程序化事务管理，开发人员可以使用Spring Framework事务抽象，该抽象可以在任何基础事务基础架构上运行。
使用首选的声明性模型，开发人员通常只编写很少或没有编写与事务管理相关的代码，因此，它们不依赖于Spring Framework事务API或任何其他事务API。

## Spring框架事务抽象
Spring事务抽象的关键是事务策略的概念。

### TransactionManager
事务策略由定义TransactionManager，其中
org.springframework.transaction.PlatformTransactionManager接口用于命令式交易管理的，
org.springframework.transaction.ReactiveTransactionManager接口用于用于反应式交易管理的。

以下清单显示了PlatformTransactionManagerAPI的定义：
```
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

尽管可以从应用程序代码中以编程方式使用它，但它主要是一个服务提供接口（service provider interface 、SPI）。
因为 PlatformTransactionManager 是接口，所以可以根据需要轻松地对其进行模拟或存根。它与诸如JNDI之类的查找策略无关。
PlatformTransactionManager实现的定义与Spring Framework IoC容器中的任何其他对象（或bean）一样。
即使使用JTA，Spring框架事务也成为有价值的抽象。与直接使用JTA相比，您可以更轻松地测试事务代码。

任何PlatformTransactionManager接口的方法都可以抛出TransactionException异常（它扩展了java.lang.RuntimeException）。
事务基础架构故障几乎总是致命的。在极少数情况下，应用程序代码实际上可以从事务失败中恢复，应用程序开发人员仍然可以选择catch and handle TransactionException。

PlatformTransactionManager的getTransaction(..)方法，根据TransactionDefinition参数，返回一个TransactionStatus类对象 。
如果当前调用堆栈中存在匹配的事务，则返回的结果TransactionStatus可能表示新事务，也可能表示现有事务。
后一种情况的含义是，与Java EE事务上下文一样，TransactionStatus与执行线程相关联。


从Spring Framework 5.2开始，Spring还为使用反应式类型或Kotlin协程的反应式应用程序提供了事务管理抽象。
以下清单显示了由 org.springframework.transaction.ReactiveTransactionManager 定义的事物策略：
```
public interface ReactiveTransactionManager extends TransactionManager {

    Mono<ReactiveTransaction> getReactiveTransaction(TransactionDefinition definition) throws TransactionException;

    Mono<Void> commit(ReactiveTransaction status) throws TransactionException;

    Mono<Void> rollback(ReactiveTransaction status) throws TransactionException;
}
```
反应式事务管理器主要是服务提供商接口（SPI），尽管您可以从应用程序代码中以编程方式使用它。
因为ReactiveTransactionManager是接口，所以可以根据需要轻松地对其进行模拟或存根。

### TransactionDefinition
TransactionDefinition接口指定：
+ 传播：通常，事务范围内的所有代码都在该事务中运行。但是，如果在已存在事务上下文的情况下运行事务方法，则可以指定行为。例如，代码可以在现有事务中继续运行（常见情况），或者可以暂停现有事务并创建新事务。
+ 隔离度：此事务与其他事务的工作隔离的程度。例如，该交易能否看到其他交易的未提交写入。
+ 超时：超时之前该事务运行了多长时间，并被基础事务基础结构自动回滚。
+ 只读状态：当代码读取但不修改数据时，可以使用只读事务。在某些情况下，例如使用Hibernate时，只读事务可能是有用的优化。

这些设置反映了标准的交易概念。

### TransactionStatus
该TransactionStatus接口为事务代码提供了一种控制事务执行和查询事务状态的简单方法。
这些概念应该很熟悉，因为它们对于所有事务API都是通用的。

以下清单显示了该 TransactionStatus接口：
```
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {

    @Override
    boolean isNewTransaction();

    boolean hasSavepoint();

    @Override
    void setRollbackOnly();

    @Override
    boolean isRollbackOnly();

    void flush();

    @Override
    boolean isCompleted();
}
```

### TransactionManager定义
无论您在Spring中选择声明式还是程序化事务管理，定义正确的TransactionManager实现都是绝对必要的。
通常，您可以通过依赖注入来定义此实现。
TransactionManager实现通常需要了解其工作环境：JDBC，JTA，Hibernate等。

以下示例显示了如何定义本地PlatformTransactionManager实现（在这种情况下，使用纯JDBC）。
可以通过创建类似于以下内容的bean来定义JDBC DataSource：
```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```

然后，相关的PlatformTransactionManagerbean定义将引用该 DataSource 定义。它应类似于以下示例：
```
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

如果您在Java EE容器中使用JTA，则可以使用DataSource通过JNDI与Spring的容器一起获得的容器JtaTransactionManager。
以下示例显示了JTA和JNDI查找版本的外观：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/jee
        https://www.springframework.org/schema/jee/spring-jee.xsd">

    <jee:jndi-lookup id="dataSource" jndi-name="jdbc/jpetstore"/>

    <bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager" />

    <!-- other <bean/> definitions here -->

</beans>
```

在JtaTransactionManager并不需要了解DataSource（或任何其他特定资源），因为它使用容器的全局事务管理。

在所有Spring事务设置中，无需更改应用程序代码。您可以仅通过更改配置来更改事务的管理方式，即使更改意味着从本地事务转移到全局事务，反之亦然。

### Hibernate事物设置
可以使用Hibernate本地事务，如以下示例所示。
在这种情况下，您需要定义一个Hibernate LocalSessionFactoryBean，您的应用程序代码可使用该Hibernate获取HibernateSession实例。

txManager在这种情况下，bean是HibernateTransactionManager类型。
在作为相同的方式DataSourceTransactionManager需要在一个参考DataSource时，HibernateTransactionManager需要将一个参考SessionFactory。

以下示例声明sessionFactory和txManagerbean：
```
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingResources">
        <list>
            <value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
        </list>
    </property>
    <property name="hibernateProperties">
        <value>
            hibernate.dialect=${hibernate.dialect}
        </value>
    </property>
</bean>

<bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

如果使用Hibernate和Java EE容器管理的JTA事务，则应使用与JtaTransactionManager前面的JDBC JTA示例相同的示例，如以下示例所示。另外，建议使Hibernate通过其事务协调器以及可能的连接释放模式配置来了解JTA：
```
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingResources">
        <list>
            <value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
        </list>
    </property>
    <property name="hibernateProperties">
        <value>
            hibernate.dialect=${hibernate.dialect}
            hibernate.transaction.coordinator_class=jta
            hibernate.connection.handling_mode=DELAYED_ACQUISITION_AND_RELEASE_AFTER_STATEMENT
        </value>
    </property>
</bean>

<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

或者，也可以将JtaTransactionManager传递给LocalSessionFactoryBean，实现相同的设置：
```
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingResources">
        <list>
            <value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
        </list>
    </property>
    <property name="hibernateProperties">
        <value>
            hibernate.dialect=${hibernate.dialect}
        </value>
    </property>
    <property name="jtaTransactionManager" ref="txManager"/>
</bean>

<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

## 将资源与事务同步
现在应该清楚如何创建不同的事务管理器，以及如何将它们链接到需要与事务同步的相关资源（例如，DataSourceTransactionManager 到JDBC DataSource，HibernateTransactionManagerHibernateSessionFactory等）。

### 高级同步方法
首选方法是使用Spring基于最高级别模板的持久性集成API或将本机ORM API与具有事务感知功能的工厂bean或代理一起使用，以管理本机资源工厂。

这些支持事务的解决方案在内部处理资源的创建和重用，清理，资源的可选事务同步以及异常映射。

因此，用户数据访问代码不必解决这些任务，而可以完全专注于非样板持久性逻辑。

通常，您使用本机ORM API或通过使用模板方法来进行JDBC访问JdbcTemplate。

### 低级同步方法
诸如DataSourceUtils（对于JDBC），EntityManagerFactoryUtils（对于JPA）， SessionFactoryUtils（对于Hibernate）之类的类处于较低级别。
当您希望应用程序代码直接处理本机持久性API的资源类型时，可以使用这些类来确保获得正确的Spring Framework管理的实例，（可选）同步事务以及处理过程中发生的异常。正确映射到一致的API。

例如，在JDBC的情况下，可以使用Spring的类DataSourceorg.springframework.jdbc.datasource.DataSourceUtils，而不是使用 传统的JDBC方法在上调用getConnection()方法，如下所示：
```
Connection conn = DataSourceUtils.getConnection(dataSource);
```

如果现有事务已具有与其同步（链接）的连接，则返回该实例。否则，方法调用将触发创建新连接，该连接（可选）同步到任何现有事务，并可供该同一事务中的后续重用使用。如前所述，任何内容 SQLException都包装在Spring Framework中CannotGetJdbcConnectionException，Spring Framework是未经检查的DataAccessException类型的层次结构之一。这种方法为您提供的信息多于从轻松获得的信息，SQLException并确保了跨数据库甚至跨不同持久性技术的可移植性。

这种方法在没有Spring事务管理的情况下也可以使用（事务同步是可选的），因此无论是否使用Spring进行事务管理，都可以使用它。

当然，一旦使用了Spring的JDBC支持，JPA支持或Hibernate支持，您通常就不愿使用DataSourceUtils或其他帮助程序类，因为与直接使用相关的API相比，通过Spring抽象进行工作会更快乐。例如，如果您使用SpringJdbcTemplate或 jdbc.object程序包来简化JDBC的使用，则正确的连接检索将在后台进行，并且您无需编写任何特殊代码。

### TransactionAwareDataSourceProxy
TransactionAwareDataSourceProxy该类的最低级别。这是target的代理DataSource，它包装了目标DataSource以增加对Spring管理的事务的了解。在这方面，它类似于DataSourceJava EE服务器提供的事务性JNDI 。

您几乎永远不需要或不想使用此类，除非必须调用现有代码并传递标准的JDBCDataSource接口实现。在这种情况下，该代码可能可用，但参与了Spring管理的事务。您可以使用前面提到的高级抽象来编写新代码。

## 声明式事物管理
大多数Spring Framework用户选择声明式事务管理。此选项对应用程序代码的影响最小，因此与无创轻量级容器的理念最一致。

Spring面向方面的编程（AOP）使Spring框架的声明式事务管理成为可能。但是，由于事务方面的代码随Spring Framework发行版一起提供并且可以以样板方式使用，因此通常不必理解AOP概念即可有效地使用此代码。
+ Spring Framework的声明式事务管理与EJB CMT相似，因为您可以指定事务行为（或缺少事务行为），直至单个方法级别。setRollbackOnly()如有必要，您可以在事务上下文中进行呼叫。两种类型的事务管理之间的区别是：
+ 与绑定到JTA的EJB CMT不同，Spring框架的声明式事务管理可在任何环境中工作。它可以通过使用JDBC，JPA或Hibernate通过调整配置文件来处理JTA事务或本地事务。
+ 您可以将Spring Framework声明式事务管理应用于任何类，而不仅限于诸如EJB之类的特殊类。
+ Spring框架提供了声明性 回滚规则，这是没有EJB等效项的功能。提供了对回滚规则的编程和声明性支持。
+ Spring Framework允许您使用AOP自定义事务行为。例如，在事务回滚的情况下，您可以插入自定义行为。您还可以添加任意建议以及事务建议。使用EJB CMT，您不能影响容器的事务管理，除非使用 setRollbackOnly()。
+ Spring框架不像高端应用程序服务器那样支持跨远程调用传播事务上下文。如果需要此功能，建议您使用EJB。但是，在使用这种功能之前，请仔细考虑，因为通常情况下，您不希望事务跨越远程调用。

回滚规则的概念很重要。他们让您指定哪些异常（和可抛出对象）应引起自动回滚。您可以在配置中而不是在Java代码中以声明方式指定。所以，尽管你仍然可以调用setRollbackOnly()上的TransactionStatus对象回滚当前事务回，经常可以指定一个规则，MyApplicationException必须始终导致回滚。此选项的主要优点是业务对象不依赖于事务基础结构。例如，他们通常不需要导入Spring事务API或其他Spring API。

尽管EJB容器的默认行为会在系统异常（通常是运行时异常）时自动回滚事务，但是EJB CMT不会在应用程序异常（即以外的已检查异常java.rmi.RemoteException）下自动回滚事务。尽管Spring声明式事务管理的默认行为遵循EJB约定（仅针对未检查的异常会自动回滚），但自定义此行为通常很有用。

### 声明式事务实现原理
仅告诉您使用注释对类进行@Transactional注释，添加@EnableTransactionManagement到配置中并希望您了解其全部工作原理是不够的。
为了提供更深入的理解，本节介绍了在与事务相关的问题的上下文中，Spring框架的声明式事务基础结构的内部工作方式。

关于Spring框架的声明式事务支持，要把握的最重要的概念是，该支持是通过AOP代理启用的 ，并且事务建议由元数据（当前基于XML或基于注释）驱动。AOP与事务元数据的组合产生了一个AOP代理，该代理TransactionInterceptor结合使用一个适当的TransactionManager实现来驱动方法调用周围的事务。

Spring框架TransactionInterceptor为命令式和反应式编程模型提供事务管理。拦截器通过检查方法返回类型来检测所需的事务管理风格。
返回反应式类型（例如Publisher或Kotlin Flow（或其子类型））的方法符合反应式事务管理的条件。所有其他返回类型包括void使用代码路径进行命令式事务管理。

事务管理风格影响需要哪个事务管理器。
命令式交易需要使用PlatformTransactionManager，而反应式交易则使用 ReactiveTransactionManager实现。

@Transactional通常与由管理的线程绑定事务一起使用 PlatformTransactionManager，将事务暴露给当前执行线程内的所有数据访问操作。
注意：这不会传播到方法中新启动的线程。

由管理的反应式事务ReactiveTransactionManager使用Reactor上下文而不是线程本地属性。
所有参与的数据访问操作都需要在同一反应性管道中的同一Reactor上下文中执行。

### 声明式事务实现示例
考虑以下接口及其附带的实现。
本示例使用 Foo和Bar类作为占位符，以便您可以专注于事务使用而不必关注特定的域模型。
出于本示例的目的，DefaultFooService类UnsupportedOperationException 在每个已实现方法的主体中引发实例的事实是很好的。
该行为使您可以查看正在创建的事务，然后回滚以响应 UnsupportedOperationException实例。

以下清单显示了该FooService 接口：
```
// the service interface that we want to make transactional

package x.y.service;

public interface FooService {

    Foo getFoo(String fooName);

    Foo getFoo(String fooName, String barName);

    void insertFoo(Foo foo);

    void updateFoo(Foo foo);

}
```

以下示例显示了上述接口的实现：
```
package x.y.service;

public class DefaultFooService implements FooService {

    @Override
    public Foo getFoo(String fooName) {
        // ...
    }

    @Override
    public Foo getFoo(String fooName, String barName) {
        // ...
    }

    @Override
    public void insertFoo(Foo foo) {
        // ...
    }

    @Override
    public void updateFoo(Foo foo) {
        // ...
    }
}
```

假设FooService接口的前两个方法getFoo(String)和 getFoo(String, String)必须在具有只读语义的事务的上下文中运行，而其他方法insertFoo(Foo)和updateFoo(Foo)必须在具有读写语义的事务的上下文中运行。
以下几节将详细说明以下配置：
```
<!-- from the file 'context.xml' -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the service object that we want to make transactional -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- the transactional semantics... -->
        <tx:attributes>
            <!-- all methods starting with 'get' are read-only -->
            <tx:method name="get*" read-only="true"/>
            <!-- other methods use the default transaction settings (see below) -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- ensure that the above transactional advice runs for any execution of an operation defined by the FooService interface -->
    <aop:config>
        <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    </aop:config>

    <!-- don't forget the DataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <!-- similarly, don't forget the TransactionManager -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- other <bean/> definitions here -->

</beans>
```
前面的配置，假定要使服务对象（fooService）具有事务性。要应用的事务语义封装在<tx:advice/>定义中。
<tx:advice/>标签定义为“所有以get开头的方法都将在只读事务的上下文中运行，而所有其他方法都将以默认事务语义运行”。
<tx:advice/>标签的 transaction-manager 属性设置TransactionManager为将要驱动事务的bean的名称（在本例中为 txManagerbean）。

<aop:config/>定义可确保txAdviceBean定义的事务建议 在程序中的适当位置运行。
首先，您定义一个切入点，该切入点与FooService接口（fooServiceOperation）中定义的任何操作的执行相匹配。
然后txAdvice，使用顾问将切入点与关联。
结果表明，在执行时fooServiceOperation，txAdvice运行定义的建议。

<aop:pointcut/>元素内定义的表达式是AspectJ切入点表达式。

一个普遍的要求是使整个服务层具有事务性。最好的方法是更改​​切入点表达式以匹配服务层中的任何操作。以下示例显示了如何执行此操作：
```
<aop:config>
    <aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
</aop:config>
```

### !!!声明式事务回滚
本节介绍如何以简单的声明方式控制事务的回滚。

向Spring框架的事务基础结构指示要回滚事务的建议方法是，在事务上下文正在执行的代码中抛出一个Exception。
Spring框架的事务基础结构代码Exception会在气泡堆积到调用堆栈时捕获任何未处理的内容，并确定是否将事务标记为回滚。

在其默认配置中，Spring的事务基础结构代码仅在 runtime，unchecke 异常 情况下将事务标记为回滚。
当抛出的异常是 RuntimeException的实例或子类时会导致回滚（Error实例也会导致回滚）。
当抛出的异常是 Checked异常 不会导致默认配置中的回滚。

可以配置将哪些Exception类型标记为回滚事务，包括已检查的异常。以下XML代码段演示了如何为选中的，特定于应用程序的Exception类型配置回滚：
```
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
    <tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

如果不希望在引发异常时回滚事务，则还可以指定“无回滚规则”。下面的示例告诉Spring框架的事务基础结构即使在InstrumentNotFoundException未处理的情况下也要提交伴随的事务：
```
<tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="updateStock" no-rollback-for="InstrumentNotFoundException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

当Spring Framework的事务基础结构捕获到异常并查询配置的回滚规则以确定是否将事务标记为回滚时，最强的匹配规则获胜。因此，在以下配置的情况下，除InstrumentNotFoundException结果以外的任何异常都会导致附带事务的回滚：
```
<tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="*" rollback-for="Throwable" no-rollback-for="InstrumentNotFoundException"/>
    </tx:attributes>
</tx:advice>
```

您还可以通过编程方式指示所需的回滚。尽管很简单，但是此过程具有很大的侵入性，并将您的代码紧密耦合到Spring Framework的事务基础结构。以下示例显示如何以编程方式指示所需的回滚：
```
public void resolvePosition() {
    try {
        // some business logic...
    } catch (NoProductInStockException ex) {
        // trigger rollback programmatically
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

### 为不同的Bean配置不同的事务语义
考虑以下场景：您有许多服务层对象，并且您希望对每个对象应用完全不同的事务配置。
您可以通过定义不同的做<aop:advisor/>用不同的元素pointcut和 advice-ref属性值。

作为比较，首先假设所有服务层类都在根x.y.service包中定义。
要使所有在该包（或子包）中定义的类实例的bean的名称以Service默认事务配置为结尾，可以编写以下代码：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:config>
        <aop:pointcut id="serviceOperation" expression="execution(* x.y.service..*Service.*(..))"/>
        <aop:advisor pointcut-ref="serviceOperation" advice-ref="txAdvice"/>
    </aop:config>

    <!-- these two beans will be transactional... -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>
    <bean id="barService" class="x.y.service.extras.SimpleBarService"/>

    <!-- ... and these two beans won't -->
    <bean id="anotherService" class="org.xyz.SomeService"/> <!-- (not in the right package) -->
    <bean id="barManager" class="x.y.service.SimpleBarManager"/> <!-- (doesn't end in 'Service') -->

    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- other transaction infrastructure beans such as a TransactionManager omitted... -->

</beans>
```

以下示例说明如何使用完全不同的事务设置配置两个不同的Bean：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:config>
        <aop:pointcut id="defaultServiceOperation" expression="execution(* x.y.service.*Service.*(..))"/>
        <aop:advisor pointcut-ref="defaultServiceOperation" advice-ref="defaultTxAdvice"/>
		
        <aop:pointcut id="noTxServiceOperation" expression="execution(* x.y.service.ddl.DefaultDdlManager.*(..))"/>
        <aop:advisor pointcut-ref="noTxServiceOperation" advice-ref="noTxAdvice"/>
    </aop:config>

    <!-- this bean will be transactional (see the 'defaultServiceOperation' pointcut) -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- this bean will also be transactional, but with totally different transactional settings -->
    <bean id="anotherFooService" class="x.y.service.ddl.DefaultDdlManager"/>

    <tx:advice id="defaultTxAdvice">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <tx:advice id="noTxAdvice">
        <tx:attributes>
            <tx:method name="*" propagation="NEVER"/>
        </tx:attributes>
    </tx:advice>

    <!-- other transaction infrastructure beans such as a TransactionManager omitted... -->

</beans>
```


### <tx：advice />设置
### 使用@Transactional
### 交易传播
### 交易事务咨询
### 在AspectJ中使用@Transactional

## Programmatic Transaction Management
## Choosing Between Programmatic and Declarative Transaction Management
## Transaction-bound Events
## Application server-specific integration
## Solutions to Common Problems
## Further Resources

# DAO Support
# Data Access with JDBC
# Data Access with R2DBC
# Retrieving Auto-generated Keys
# Object Relational Mapping (ORM) Data Access
# Marshalling XML by Using Object-XML Mappers
# Appendix