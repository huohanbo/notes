# 参考资料
> [Spring参考文档](https://docs.spring.io/spring-framework/docs/current/reference/html/)
> [Data Access](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#spring-data-tier)

--------------------------------------------------
参考文档的这一部分涉及数据访问以及数据访问层与业务或服务层之间的交互。

详细介绍了Spring全面的事务管理支持，然后全面介绍了Spring框架所集成的各种数据访问框架和技术。

# Transaction Management 事物管理
全面的事务支持是使用Spring Framework的最令人信服的原因。

Spring框架为事务管理提供了一致的抽象，具有以下优点：
+ 跨不同事务API的一致编程模型，例如Java事务API（Java Transaction API、JTA），JDBC，Hibernate和Java Persistence API（JPA）。
+ 支持声明式事务管理。
+ 与诸如JTA之类的复杂事务API相比，用于程序化事务管理的API更简单。
+ 与Spring的数据访问抽象的出色集成。

以下各节描述了Spring Framework的事务功能和技术：
+ Spring框架的事务支持模型的优点描述了为什么将使用Spring框架的事务抽象而不是EJB容器管理的事务（CMT）或选择通过诸如Hibernate之类的专有API驱动本地事务的原因。
+ 了解Spring Framework事务抽象 概述了核心类，并描述了如何DataSource 从各种来源配置和获取实例。
+ 将资源与事务同步描述了应用程序代码如何确保正确创建，重用和清理资源。
+ 声明式事务管理描述了对声明式事务管理的支持。
+ 程序化事务管理涵盖对程序化（即，显式编码）事务管理的支持。
+ 事务绑定事件描述了如何在事务中使用应用程序事件。

本章还讨论了最佳实践， 应用程序服务器集成以及常见问题的解决方案。

## Spring事务模型的优点
传统上，Java EE开发人员在事务管理中有两种选择：全局或本地事务，两者都有很大的局限性。

### 全局事物
全局事务使可以使用多个事务资源，通常是关系数据库和消息队列。
应用服务器通过JTA管理全局事务，该JTA是繁琐的API（部分是由于其异常模型）。
此外，UserTransaction通常需要从JNDI派生JTA ，这意味着还需要使用JNDI才能使用JTA。
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

## Spring事务抽象
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
即使使用JTA，Spring框架事务也成为有价值的抽象。与直接使用JTA相比，可以更轻松地测试事务代码。

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
反应式事务管理器主要是服务提供商接口（SPI），尽管可以从应用程序代码中以编程方式使用它。
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

### 定义TransactionManager
无论在Spring中选择声明式还是程序化事务管理，定义正确的TransactionManager实现都是绝对必要的。
通常，可以通过依赖注入来定义此实现。
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

如果在Java EE容器中使用JTA，则可以使用DataSource通过JNDI与Spring的容器一起获得的容器JtaTransactionManager。
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

在所有Spring事务设置中，无需更改应用程序代码。可以仅通过更改配置来更改事务的管理方式，即使更改意味着从本地事务转移到全局事务，反之亦然。

### Hibernate事物设置
可以使用Hibernate本地事务，如以下示例所示。
在这种情况下，需要定义一个Hibernate LocalSessionFactoryBean，的应用程序代码可使用该Hibernate获取HibernateSession实例。

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

通常，使用本机ORM API或通过使用模板方法来进行JDBC访问JdbcTemplate。

### 低级同步方法
诸如DataSourceUtils（对于JDBC），EntityManagerFactoryUtils（对于JPA）， SessionFactoryUtils（对于Hibernate）之类的类处于较低级别。
当希望应用程序代码直接处理本机持久性API的资源类型时，可以使用这些类来确保获得正确的Spring Framework管理的实例，（可选）同步事务以及处理过程中发生的异常。正确映射到一致的API。

例如，在JDBC的情况下，可以使用Spring的类DataSourceorg.springframework.jdbc.datasource.DataSourceUtils，而不是使用 传统的JDBC方法在上调用getConnection()方法，如下所示：
```
Connection conn = DataSourceUtils.getConnection(dataSource);
```

如果现有事务已具有与其同步（链接）的连接，则返回该实例。否则，方法调用将触发创建新连接，该连接（可选）同步到任何现有事务，并可供该同一事务中的后续重用使用。如前所述，任何内容 SQLException都包装在Spring Framework中CannotGetJdbcConnectionException，Spring Framework是未经检查的DataAccessException类型的层次结构之一。这种方法为提供的信息多于从轻松获得的信息，SQLException并确保了跨数据库甚至跨不同持久性技术的可移植性。

这种方法在没有Spring事务管理的情况下也可以使用（事务同步是可选的），因此无论是否使用Spring进行事务管理，都可以使用它。

当然，一旦使用了Spring的JDBC支持，JPA支持或Hibernate支持，通常就不愿使用DataSourceUtils或其他帮助程序类，因为与直接使用相关的API相比，通过Spring抽象进行工作会更快乐。例如，如果使用SpringJdbcTemplate或 jdbc.object程序包来简化JDBC的使用，则正确的连接检索将在后台进行，并且无需编写任何特殊代码。

### TransactionAwareDataSourceProxy
TransactionAwareDataSourceProxy该类的最低级别。这是target的代理DataSource，它包装了目标DataSource以增加对Spring管理的事务的了解。在这方面，它类似于DataSourceJava EE服务器提供的事务性JNDI 。

几乎永远不需要或不想使用此类，除非必须调用现有代码并传递标准的JDBCDataSource接口实现。在这种情况下，该代码可能可用，但参与了Spring管理的事务。可以使用前面提到的高级抽象来编写新代码。

## !!!声明式事物管理
大多数Spring Framework用户选择声明式事务管理。此选项对应用程序代码的影响最小，因此与无创轻量级容器的理念最一致。

Spring面向方面的编程（AOP）使Spring框架的声明式事务管理成为可能。但是，由于事务方面的代码随Spring Framework发行版一起提供并且可以以样板方式使用，因此通常不必理解AOP概念即可有效地使用此代码。
+ Spring Framework的声明式事务管理与EJB CMT相似，因为可以指定事务行为（或缺少事务行为），直至单个方法级别。setRollbackOnly()如有必要，可以在事务上下文中进行呼叫。两种类型的事务管理之间的区别是：
+ 与绑定到JTA的EJB CMT不同，Spring框架的声明式事务管理可在任何环境中工作。它可以通过使用JDBC，JPA或Hibernate通过调整配置文件来处理JTA事务或本地事务。
+ 可以将Spring Framework声明式事务管理应用于任何类，而不仅限于诸如EJB之类的特殊类。
+ Spring框架提供了声明性 回滚规则，这是没有EJB等效项的功能。提供了对回滚规则的编程和声明性支持。
+ Spring Framework允许使用AOP自定义事务行为。例如，在事务回滚的情况下，可以插入自定义行为。还可以添加任意建议以及事务建议。使用EJB CMT，不能影响容器的事务管理，除非使用 setRollbackOnly()。
+ Spring框架不像高端应用程序服务器那样支持跨远程调用传播事务上下文。如果需要此功能，建议使用EJB。但是，在使用这种功能之前，请仔细考虑，因为通常情况下，不希望事务跨越远程调用。

回滚规则的概念很重要。他们让指定哪些异常（和可抛出对象）应引起自动回滚。可以在配置中而不是在Java代码中以声明方式指定。所以，尽管你仍然可以调用setRollbackOnly()上的TransactionStatus对象回滚当前事务回，经常可以指定一个规则，MyApplicationException必须始终导致回滚。此选项的主要优点是业务对象不依赖于事务基础结构。例如，他们通常不需要导入Spring事务API或其他Spring API。

尽管EJB容器的默认行为会在系统异常（通常是运行时异常）时自动回滚事务，但是EJB CMT不会在应用程序异常（即以外的已检查异常java.rmi.RemoteException）下自动回滚事务。尽管Spring声明式事务管理的默认行为遵循EJB约定（仅针对未检查的异常会自动回滚），但自定义此行为通常很有用。

### 声明式事务实现原理
仅仅了解用```@Transaction```注解注解类、将```@EnableTransactionManagement```添加到配置中是不够的。

关于Spring的声明性事务支持，需要掌握的最重要的概念是这种支持是通过AOP代理启用的，并且事务性建议是由元数据(当前基于XML或注解)驱动的。
AOP与事务性元数据的组合产生了一个AOP代理，它结合使用```TransactionInterceptor```和适当的```TransactionManager```实现来驱动方法调用周围的事务。

Spring的```TransactionInterceptor```为命令式和反应式编程模型提供事务管理。
拦截器通过检查方法返回类型来检测所需的事务管理风格。
返回反应式类型(如Publisher或Kotlin Flow(或其子类型))的方法符合反应式事务管理的条件。
所有其他返回类型(包括void)都使用代码路径进行命令性事务管理。

事务管理风格会影响需要哪个事务管理器。命令性事务需要PlatformTransactionManager，而反应性事务使用ReactiveTransactionManager实现。
由```PlatformTransactionManager```管理的线程绑定事务，将事务公开给当前执行线程内的所有数据访问操作。注意：**这不会传播到方法中新启动的线程**。
由```ReactiveTransactionManager```管理的反应式事务使用反应器上下文，而不是线程本地属性。因此，所有参与的数据访问操作都需要在同一反应性管道中的同一反应器上下文中执行。

### !!!声明式事务实现示例
请考虑以下接口及其附带实现。

```DefaultFooService```类在每个已实现方法的主体中抛出```UnsupportedOperationException```实例这一事实是好的。该行为允许看到正在创建事务，然后回滚以响应UnsupportedOperationException实例。

下面的清单显示了FooService接口：
```
public interface FooService {
    Foo getFoo(String fooName);
    Foo getFoo(String fooName, String barName);
    void insertFoo(Foo foo);
    void updateFoo(Foo foo);
}
```

以下示例显示了上述接口的实现：
```
public class DefaultFooService implements FooService {
    @Override
    public Foo getFoo(String fooName) {
    }
    @Override
    public Foo getFoo(String fooName, String barName) {
    }
    @Override
    public void insertFoo(Foo foo) {
    }
    @Override
    public void updateFoo(Foo foo) {
    }
}
```

假设```FooService```接口的前两个方法```getFoo(String)```和```getFoo(string，string)```必须在具有只读语义的事务上下文中运行，而其他方法```inserttFoo(Foo)```和```updateFoo(Foo)```必须在具有读写语义的事务上下文中运行。
下面几段将详细介绍以下配置（context.xml）：
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

    <!-- DataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <!-- TransactionManager -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 事物建议 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- 事务语义 -->
        <tx:attributes>
            <!-- 以get开头的方法使用只读事物 -->
            <tx:method name="get*" read-only="true"/>
            <!-- 其他方法使用默认事物 -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- 确保上面的事物建议针对FooService接口定义的操作的任何执行运行 -->
    <aop:config>
        <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    </aop:config>

    <!-- 业务对象 -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>
</beans>
```
在前面的配置中：

```<tx:advice/>```标签定义事务语义。
```<tx:advice/>```的 ```transaction-manager``` 属性要设置为驱动事务的```TransactionManager```的bean名称(txManager)。
如果要连接的```TransactionManager```的bean名称为```transactionManager```，则可以省略```transaction-manager```属性。
如果要连接的```TransactionManager```的bean名称为其他名称，则必须显式定义```transaction-manager```属性。

```<aop：config/>```定义确保```txAdviceBean```定义的事务性通知在程序中的适当位置运行。
首先，定义一个与FooService接口(FooServiceOperation)中定义的任何操作的执行相匹配的切入点
然后使用Advisor将切入点与txAdvice相关联。结果表明，在执行fooServiceOperation时，将运行txAdility定义的通知。

```<aop:pointcut/>```定义的表达式是AspectJ切入点表达式。

**一个普遍的要求是使整个服务层具有事务性**。最好的方法是更改​​切入点表达式以匹配服务层中的任何操作。
以下示例显示了如何执行此操作：
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

还可以通过编程方式指示所需的回滚。尽管很简单，但是此过程具有很大的侵入性，并将的代码紧密耦合到Spring Framework的事务基础结构。以下示例显示如何以编程方式指示所需的回滚：
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
考虑以下场景：有许多服务层对象，并且希望对每个对象应用完全不同的事务配置。
可以通过定义不同的做```<aop:advisor/>```用不同的元素```pointcut```和```advice-ref```属性值。

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

### !!!<tx:advice/>设置
本节总结了可以使用```<tx:advice/>```标记指定的各种事务设置。
默认的```<tx:advice/>```设置为：
+ 传播特性为```REQUIRED```。
+ 隔离级别为```DEFAULT```。
+ 事务是```read-write```。
+ 事务超时默认为底层事务系统的默认超时，如果不支持超时则为none。
+ 任何```RuntimeException```都会触发回滚，而任何```checked Exception```都不会触发回滚。

可以更改这些默认设置。
下表总结了```<tx:advice/>```和```<tx:attributes/>```标签中的```<tx:method/>```标签的各种属性：

|属性	|是否必需	|默认	|描述	|
|--	|--	|--	|--	|
|name	|是	|	|方法名称。通配符(*)可用于将相同的事务属性设置与许多方法(例如，GET*、HANDLE*、ON*EVENT等)相关联。	|
|propagation	|否	|REQUIRED	|事务传播行为	|
|isolation	|否	|DEFAULT	|事务隔离级别。仅适用于REQUIRED或REQUIRED_NEW的传播设置。	|
|timeout	|否	|-1	|事务超时(秒)。仅适用于REQUIRED或REQUIES_NEW传播。		|
|read-only	|否	|false	|读写事务与只读事务。仅适用于REQUIRED或REQUIRED_NEW。	|
|rollback-for	|否	|	|以逗号分隔的触发回滚的异常实例列表。例如，com.foo.MyBusinessException、ServletException。	|
|no-rollback-for	|否	|	|以逗号分隔的不触发回滚的异常实例列表。例如，com.foo.MyBusinessException、ServletException。	|

### !!!@Transactional
除了基于XML的声明性事务配置方法之外，还可以使用基于注解的方法。
直接在Java源代码中声明事务语义使声明更接近受影响的代码。

@Transaction注解使用示列：
```
@Transactional
public class DefaultFooService implements FooService {

    Foo getFoo(String fooName) {
    }

    Foo getFoo(String fooName, String barName) {
    }

    void insertFoo(Foo foo) {
    }

    void updateFoo(Foo foo) {
    }
}
```
注解在类级别使用，默认表示**声明类(及其子类)的所有方法**。此外，每个方法都可以单独进行注解。

通过```@Configuration```类中的```@EnableTransactionManagement```注解使Bean实例成为事务性的。

在XML配置中，```<tx:annotation-driven/>```标签提供了类似的功能：
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

    <!-- DataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <!-- TransactionManager -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 启用基于注解的事务行为配置。仍然需要TransactionManager -->
    <tx:annotation-driven transaction-manager="txManager"/>

</beans>
```

当使用代理时，只能将```@Transaction```注解应用于具有```public```可见性的方法。
如果注解protected、private或package可见的方法，则不会显示已配置的事务设置。如果需要注解非```public```方法，请考虑使用AspectJ。

代理模式下(默认设置)，只有通过代理传入的外部方法调用才会被拦截，即自调用(实际上是目标对象中的一个方法调用目标对象的另一个方法)在运行时不会导致实际的事务，即使被调用的方法被标记为@Transaction。

代理必须完全初始化才能提供预期的行为，因此不应该在初始化代码中依赖此功能(即@PostConstruct)。

```@Transaction```注释可以应用于接口、接口上的方法、类或类上的公共方法。
仅仅使用```@Transaction```注释并不足以激活事务行为，```@Transaction```注释只是一些运行时基础设施可以使用的元数据，这些运行时基础设施支持```@Transaction```，并且可以使用元数据来配置具有事务行为的适当bean。

**注释驱动事务设置**

|XML属性	|Annotation属性	|默认值	|说明	|
|--	|--	|--	|--	|
|transaction-manager	|N/A	|transactionManager	|使用的事务管理器的名称	|
|mode	|mode	|proxy	|默认模式(代理模式)使用Spring的AOP框架处理要代理的带注释的bean(遵循代理语义，如前所述，仅适用于通过代理传入的方法调用)。替代模式(AspectJ)用Spring的AspectJ事务方面编织受影响的类，修改目标类字节码以应用于任何类型的方法调用。AspectJ编织要求类路径中有Spring-aspects.jar，并且启用了加载时编织(或编译时编织)。	|
|proxy-target-class	|proxyTargetClass	|false	|仅适用于代理模式。控制为使用@Transaction批注批注的类创建哪种类型的事务代理。如果proxy-target-class属性设置为true，则会创建基于类的代理。如果proxy-target-class为false或该属性被省略，则创建标准的基于JDK接口的代理。	|
|order	|order	|Ordered.LOWEST_PRECEDENCE	|定义应用于使用@Transaction注释的Bean的事务通知的顺序。没有指定的顺序意味着AOP子系统决定通知的顺序。	|

**在评估方法的事务设置时，派生最多的位置优先。**
在下面的示例中，DefaultFooService类在类级别使用只读事务的设置进行批注，但同一类中updateFoo(Foo)方法上的@Transaction批注优先于在类级别定义的事务设置。
```
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

    public Foo getFoo(String fooName) {
        // ...
    }

    // these settings have precedence for this method
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
    public void updateFoo(Foo foo) {
        // ...
    }
}
```

#### !!!@Transactional设置
@Transaction注释是元数据，它指定接口、类或方法必须具有事务性语义。

默认的@Transaction设置如下：
+ 传播特性为```PROPACTION_REQUIRED```。
+ 隔离级别为```ISOLATION_DEFAULT```。
+ 事务为```read-write```。
+ 事务超时默认为底层事务系统的默认超时，如果不支持超时，则默认为无。
+ 任何```RuntimeException```都会触发回滚，而任何```checked Exception```都不会。

可以更改这些默认设置。下表总结了@Transaction批注的各种属性：

|属性	|类型	|说明	|
|--	|--	|--	|
|value	|String	|要使用的事务管理器	|
|propagation	|enum: Propagation	|事物传播特性	|
|isolation	|enum: Isolation	|事物隔离级别。仅适用于传播特性为REQUIRED或REQUIRED_NEW的事物	|
|timeout	|int(以秒为粒度单位)	|事物超时时间。仅适用于传播特性为REQUIRED或REQUIRED_NEW的事物	|
|readOnly	|boolean	|读写事务与只读事务。仅适用于传播特性为REQUIRED或REQUIRED_NEW的事物	|
|rollbackFor	|Class对象的数组，它必须从Throwable派生。	|引起回滚的异常类的数组	|
|noRollbackFor	|Class对象的数组，它必须从Throwable派生。	|不能引起回滚的异常类的数组	|
|rollbackForClassName	|类名的数组。这些类必须从Throwable派生。	|引起回滚的异常类名称的数组	|
|noRollbackForClassName	|类名的数组。这些类必须从Throwable派生。	|不能引起回滚的异常类名称的数组	|
|label	|字符串数组，用于向事务添加富有表现力的描述。	|标签可以由事务管理器进行评估，以将特定于实现的行为与实际事务相关联。	|

#### 使用多个事务管理器
大多数Spring应用程序只需要一个事务管理器，但在某些情况下，可能希望在单个应用程序中有多个独立的事务管理器。
使用```@Transaction```注释的```value```或```transactionManager```属性来选择性地指定要使用的TransactionManager的标识。
可以是事务管理器Bean的Bean名称或限定符值。

例如，使用限定符表示法，可以在应用程序上下文中将以下Java代码与以下事务管理器Bean声明相结合：
```
public class TransactionalService {

    @Transactional("order")
    public void setSomething(String name) { ... }

    @Transactional("account")
    public void doSomething() { ... }

    @Transactional("reactive-account")
    public Mono<Void> doSomethingReactive() { ... }
}
```

下面的清单显示了```TransactionManager```Bean声明：
```
<bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	...
	<qualifier value="order"/>
</bean>

<bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	...
	<qualifier value="account"/>
</bean>

<bean id="transactionManager3" class="org.springframework.data.r2dbc.connectionfactory.R2dbcTransactionManager">
	...
	<qualifier value="reactive-account"/>
</bean>
```

#### 自定义合成注解
如果发现在许多不同的方法上重复使用@Transaction相同的属性，Spring的元注释支持允许为特定用例定义自定义组合注释。
例如，考虑以下注释定义：
```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "order", label = "causal-consistency")
public @interface OrderTx {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "account", label = "retryable")
public @interface AccountTx {
}
```

基于此可以重写上一节中的示例，如下所示：
```
public class TransactionalService {

    @OrderTx
    public void setSomething(String name) {
        // ...
    }

    @AccountTx
    public void doSomething() {
        // ...
    }
}
```

### 事物转播特性
本节描述Spring中事务传播的一些语义。
请注意，本节不是对事务传播本身的介绍。相反，它详细介绍了Spring中有关事务传播的一些语义。
在Spring管理的事务中，请注意**物理事务和逻辑事务**之间的差异，以及传播特性如何处理这种差异。

#### PROPAGATION_REQUIRED
```PROPACTION_REQUIRED```强制执行物理事务。

如果还不存在事务，则在本地为当前作用域执行，或者参与为更大作用域定义的现有“外部”事务。
这是同一线程内的公共调用堆栈安排中的一个很好的缺省设置。

默认情况下，参与事务加入外部作用域的特征，以静默方式忽略本地隔离级别、超时值或只读标志(如果有)。
如果希望在参与具有不同隔离级别的现有事务时拒绝隔离级别声明，请考虑将事务管理器上的validateExistingTransaction标志切换为true。此非宽松模式还拒绝只读不匹配。

#### PROPACTION_REQUIRED_NEW
```PROPACTION_REQUIRED_NEW```始终对每个受影响的事务作用域使用独立的物理事务，从不参与外部作用域的现有事务。
在这样的安排中，底层资源事务是不同的，因此可以独立地提交或回滚，其中外部事务不受内部事务的回滚状态的影响，并且内部事务的锁在其完成后立即被释放。
这种独立的内部事务还可以声明其自己的隔离级别、超时和只读设置，而不继承外部事务的特征。

#### PROPAGATION_NESTED
```PROPACTION_NESTED```使用具有多个保存点的单个物理事务，它可以回滚到这些保存点。
这样的部分回滚允许内部事务作用域触发其作用域的回滚，而外部事务能够继续物理事务，尽管有些操作已经回滚。
此设置通常映射到JDBC保存点，因此它仅适用于JDBC资源事务。

### 为事物运行提供切面增强
假设既要运行事务性操作，又要运行一些基本的切面增强。
要如何在```<tx:annotation-driven/>```的上下文中实现这一点？

调用updateFoo(Foo)方法时，希望看到以下操作：
+ 配置的分析切面启动。
+ 事务运行。
+ 目标对象上的方法运行。
+ 事务提交。
+ 分析方面报告整个事务性方法调用的确切持续时间。

下面的代码显示了前面讨论的简单分析方面：
```
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;
import org.springframework.core.Ordered;

public class SimpleProfiler implements Ordered {

    private int order;

    // 允许我们控制增强的顺序
    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    // 环绕增强
    public Object profile(ProceedingJoinPoint call) throws Throwable {
        Object returnValue;
        StopWatch clock = new StopWatch(getClass().getName());
        try {
            clock.start(call.toShortString());
            returnValue = call.proceed();
        } finally {
            clock.stop();
            System.out.println(clock.prettyPrint());
        }
        return returnValue;
    }
}
```

以下配置创建了一个fooService bean，该bean以所需的顺序对其应用了分析和事务方面：
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

    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- 切面 -->
    <bean id="profiler" class="x.y.SimpleProfiler">
        <!-- 在事务增强之前运行(order较小) -->
        <property name="order" value="1"/>
    </bean>

    <tx:annotation-driven transaction-manager="txManager" order="200"/>

    <aop:config>
            <!-- this advice runs around the transactional advice -->
            <aop:aspect id="profilingAspect" ref="profiler">
                <aop:pointcut id="serviceMethodWithReturnValue"
                        expression="execution(!void x.y..*Service.*(..))"/>
                <aop:around method="profile" pointcut-ref="serviceMethodWithReturnValue"/>
            </aop:aspect>
    </aop:config>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```

下面的示例创建与前两个示例相同的设置，但使用纯XML声明性方法：
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

    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- 分析增强-->
    <bean id="profiler" class="x.y.SimpleProfiler">
        <property name="order" value="1"/>
    </bean>

    <aop:config>
        <aop:pointcut id="entryPointMethod" expression="execution(* x.y..*Service.*(..))"/>
        <!-- runs after the profiling advice (c.f. the order attribute) -->

        <aop:advisor advice-ref="txAdvice" pointcut-ref="entryPointMethod" order="2"/>
        <!-- order value is higher than the profiling aspect -->

        <aop:aspect id="profilingAspect" ref="profiler">
            <aop:pointcut id="serviceMethodWithReturnValue"
                    expression="execution(!void x.y..*Service.*(..))"/>
            <aop:around method="profile" pointcut-ref="serviceMethodWithReturnValue"/>
        </aop:aspect>

    </aop:config>

    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- other <bean/> definitions such as a DataSource and a TransactionManager here -->

</beans>
```

### 在AspectJ中使用@Transactional
以下示例显示如何创建事务管理器并配置```AnnotationTransactionAspect```以使用它：
```
// 构造适当的事务管理器
DataSourceTransactionManager txManager = new DataSourceTransactionManager(getDataSource());

// 配置AnnotationTransactionAspect以使用它；这必须在执行任何事务性方法之前完成
AnnotationTransactionAspect.aspectOf().setTransactionManager(txManager);
```

## 编程式事务管理
Spring提供了两种编程事务管理方法：
+ 使用TransactionTemplate或TransactionalOperator。
+ 使用TransactionManager的实现。

Spring通常推荐TransactionTemplate用于命令流中的程序性事务管理，而TransactionalOperator用于反应性代码。
第二种方法类似于使用JTA UserTransaction API，尽管异常处理不那么麻烦。

### TransactionTemplate
TransactionTemplate采用与其他Spring模板(如JdbcTemplate)相同的方法。
它使用回调方法，并生成意图驱动的代码，因为的代码只关注想要做的事情。

必须在事务上下文中运行并显式使用TransactionTemplate的应用程序代码类似于下一个示例。
可以编写TransactionCallback实现(通常表示为匿名内部类)，其中包含在事务上下文中运行所需的代码。
然后，可以将自定义TransactionCallback的实例传递给Execute(..)。
以下示例显示如何执行此操作：
```
public class SimpleService implements Service {

    // 此实例中的所有方法共享单个TransactionTemplate
    private final TransactionTemplate transactionTemplate;

    // 使用构造函数注入来提供PlatformTransactionManager
    public SimpleService(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public Object someServiceMethod() {
        return transactionTemplate.execute(new TransactionCallback() {
            // 此方法中的代码在事务性上下文中运行
            public Object doInTransaction(TransactionStatus status) {
                updateOperation1();
                return resultOfUpdateOperation2();
            }
        });
    }
}
```

如果没有返回值，可以将TransactionCallbackWithoutResult类与匿名类一起使用，如下所示：
```
transactionTemplate.execute(new TransactionCallbackWithoutResult() {
    protected void doInTransactionWithoutResult(TransactionStatus status) {
        updateOperation1();
        updateOperation2();
    }
});
```

回调中的代码可以通过对提供的TransactionStatus对象调用setRollbackOnly()方法回滚事务，如下所示：
```
transactionTemplate.execute(new TransactionCallbackWithoutResult() {

    protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
            updateOperation1();
            updateOperation2();
        } catch (SomeBusinessException ex) {
            status.setRollbackOnly();
        }
    }
});
```

### TransactionOperator
TransactionOperator遵循与其他反应式操作符类似的操作符设计。它使用回调方法(将应用程序代码从必须执行样板获取和释放事务资源的工作中解放出来)，并生成意图驱动的代码，因为的代码只关注想要做的事情。

必须在事务上下文中运行并显式使用TransactionOperator的应用程序代码类似于下面的示例：
```
public class SimpleService implements Service {

    // 此实例中的所有方法共享单个TransactionOperator
    private final TransactionalOperator transactionalOperator;

    // 使用构造函数注入来提供ReactiveTransactionManager
    public SimpleService(ReactiveTransactionManager transactionManager) {
        this.transactionOperator = TransactionalOperator.create(transactionManager);
    }

    public Mono<Object> someServiceMethod() {

        // 此方法中的代码在事务性上下文中运行

        Mono<Object> update = updateOperation1();

        return update.then(resultOfUpdateOperation2).as(transactionalOperator::transactional);
    }
}
```

### TransactionManager
#### PlatformTransactionManager
对于命令性事务，可以直接使用org.springframework.transaction.PlatformTransactionManager来管理事务。
为此，请通过Bean引用将使用的PlatformTransactionManager的实现传递给Bean。
然后，通过使用TransactionDefinition和TransactionStatus对象，可以启动事务、回滚和提交。
以下示例显示如何执行此操作：
```
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
// explicitly setting the transaction name is something that can be done only programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

TransactionStatus status = txManager.getTransaction(def);
try {
    // put your business logic here
}
catch (MyException ex) {
    txManager.rollback(status);
    throw ex;
}
txManager.commit(status);
```

#### ReactiveTransactionManager
在处理反应性事务时，可以直接使用org.springframework.transaction.ReactiveTransactionManager来管理的事务。
为此，请通过Bean引用将使用的Reactive TransactionManager的实现传递给的Bean。
然后，通过使用TransactionDefinition和Reactive Transaction对象，可以启动事务、回滚和提交。
以下示例显示如何执行此操作：
```
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
// explicitly setting the transaction name is something that can be done only programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

Mono<ReactiveTransaction> reactiveTx = txManager.getReactiveTransaction(def);

reactiveTx.flatMap(status -> {

    Mono<Object> tx = ...; // put your business logic here

    return tx.then(txManager.commit(status))
            .onErrorResume(ex -> txManager.rollback(status).then(Mono.error(ex)));
});
```

## 在声明式事务管理和编程式事务管理之间进行选择
只有当有少量的事务性操作时，编程式事务管理通常才是一个好主意。
例如，如果的Web应用程序只需要某些更新操作的事务，则可能不希望使用Spring或任何其他技术来设置事务代理。
在这种情况下，使用TransactionTemplate可能是一种很好的方法。能够显式设置事务名称也是只能通过使用事务管理的编程方法来实现的。

另一方面，如果的应用程序有许多事务性操作，声明性事务管理通常是值得的。
它将事务管理排除在业务逻辑之外，并且配置起来并不困难。
当使用Spring Framework而不是EJB CMT时，声明性事务管理的配置成本大大降低。

## 事务绑定事件
从Spring4.2开始，事件的侦听器可以绑定到事务的某个阶段。
典型的示例是在事务成功完成时处理事件。这样做可以在当前事务的结果对侦听器真正重要时更灵活地使用事件。

可以使用@EventListener注释注册常规事件侦听器。
如果需要将其绑定到事务，请使用@TransactionalEventListener。执行此操作时，默认情况下，侦听器将绑定到事务的提交阶段。

下一个示例显示了这个概念。假设一个组件发布了一个由订单创建的事件，并且我们希望定义一个侦听器，该侦听器应该只在发布它的事务成功提交后才处理该事件。下面的示例设置这样的事件侦听器：
```
@Component
public class MyComponent {

    @TransactionalEventListener
    public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) {
        // ...
    }
}
```
@TransactionalEventListener注释公开了一个阶段属性，该属性允许自定义侦听器应该绑定到的事务的阶段。
有效阶段是BEFORE_COMMIT、AFTER_COMMIT(默认值)、AFTER_ROLLBACK以及AFTER_COMPLETION，后者聚合事务完成(无论是提交还是回滚)。

如果没有正在运行的事务，则根本不会调用侦听器，因为我们无法遵守所需的语义。
但是，可以通过将批注的FallbackExecution属性设置为true来覆盖该行为。


## 特定于应用程序服务器的集成
略

## 常见问题的解决方案
略

## 其他资源
略

# DAO Support
Spring中的数据访问对象(```Data Access Object```，DAO)支持旨在使以一致的方式轻松使用数据访问技术(如JDBC、Hibernate或JPA)。
这使可以相当容易地在上述持久性技术之间进行切换，并且还允许编写代码而无需担心捕获特定于每种技术的异常。

## 异常层次结构
Spring提供了从特定于技术的异常(如SQLException)到其自己的异常类层次结构(将DataAccessException作为根异常)的方便转换。这些异常包装了原始异常，这样就永远不会丢失任何有关可能出错的信息的风险。

除了JDBC异常，Spring还可以包装特定于JPA和Hibernate的异常，将它们转换为一组集中的运行时异常。这使可以只在适当的层中处理大多数不可恢复的持久性异常，而不需要在DAO中使用烦人的样板捕捉抛出块和异常声明。(不过，仍然可以在需要的任何地方捕获和处理异常。)。如上所述，JDBC异常(包括特定于数据库的方言)也会转换为相同的层次结构，这意味着可以在一致的编程模型中使用JDBC执行一些操作。

前面的讨论适用于Spring对各种ORM框架的支持中的各种模板类。如果使用基于拦截器的类，应用程序必须关注HibernateExceptions和PersistenceExceptions本身的处理，最好通过委托ConvertHibernateAccessException(..)或ConvertJpaAccessException(..)方法，分别调用SessionFactoryUtils。这些方法将异常转换为与org.springframework work.ao异常层次结构中的异常兼容的异常。由于PersistenceExceptions未被检查，它们也可能被抛出(不过，在异常方面牺牲了泛型DAO抽象)。

## !!!@Repository
保证数据访问对象(DAO)或存储库提供异常转换的最好方法是使用```@Repository```注释。
该注释还允许组件扫描支持查找和配置DAO和存储库，而不必为它们提供XML配置条目。
以下示例显示如何使用@Repository注释：
```
@Repository 
public class SomeMovieFinder implements MovieFinder {
    // ...
}
```

根据使用的持久性技术，任何DAO或存储库实现都需要访问持久性资源。例如，基于JDBC的存储库需要访问JDBC数据源，而基于JPA的存储库需要访问EntityManager。
要实现这一点，最简单的方法是通过使用@Autwire、@Inject、@Resource或@PersistenceContext注释之一注入此资源依赖项。

以下示例适用于JPA存储库：
```
@Repository
public class JpaMovieFinder implements MovieFinder {

    @PersistenceContext
    private EntityManager entityManager;

    // ...
}
```

如果使用经典的Hibernate API，则可以注入SessionFactory，如下例所示：
```
@Repository
public class HibernateMovieFinder implements MovieFinder {

    private SessionFactory sessionFactory;

    @Autowired
    public void setSessionFactory(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    // ...
}
```

这个例子是典型的JDBC支持。可以将DataSource注入到初始化方法或构造函数中，在其中将使用此DataSource创建一个JdbcTemplate和其他数据访问支持类(如SimpleJdbcCall和其他类)。下面的示例自动绑定一个数据源：
```
@Repository
public class JdbcMovieFinder implements MovieFinder {

    private JdbcTemplate jdbcTemplate;

    @Autowired
    public void init(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // ...
}
```

# Data Access with JDBC 使用JDBC进行数据访问
Spring Framework JDBC抽象提供的价值可能最好地通过下表中概述的操作序列来显示。
该表显示了Spring负责哪些操作，哪些操作是的责任。

|模块	|Spring	|开发者	|
|--	|--	|--	|
|定义连接参数	|	|V	|
|打开连接	|V	|	|
|指定SQL语句	|	|V	|
|声明参数并提供参数值	|	|V	|
|准备并运行该语句	|V	|	|
|设置循环以迭代结果	|V	|	|
|完成每个迭代的工作	|	|V	|
|处理任何异常	|V	|	|
|处理事物	|V	|	|
|关闭连接、语句和结果集	|V	|	|

## 选择JDBC数据库访问方式
除了JdbcTemplate的三种风格之外，一种新的SimpleJdbcInsert和SimpleJdbcCall方法优化了数据库元数据，RDBMS对象样式采用了与JDO查询设计类似的更面向对象的方法。
一旦开始使用这些方法中的一种，仍然可以混合匹配以包含来自不同方法的功能。
所有方法都需要与JDBC 2.0兼容的驱动程序，一些高级功能需要JDBC 3.0驱动程序。

### JdbcTemplate
```JdbcTemplate```是经典且最流行的Spring JDBC方法。这种“最低级别”的方法和所有其他方法都在幕后使用JdbcTemplate。

### NamedParameterJdbcTemplate
```NamedParameterJdbcTemplate```包装了```JdbcTemplate```以提供命名参数，而不是传统的JDBC的```?```占位符。当一条SQL语句有多个参数时，这种方法提供了更好的文档和易用性。

### SimpleJdbcInsert和SimpleJdbcCall
```SimpleJdbcInsert```和```SimpleJdbcCall```优化数据库元数据以限制必要的配置量。这种方法简化了编码，因此只需要提供表或过程的名称，并提供与列名匹配的参数映射。只有当数据库提供足够的元数据时，这才起作用。如果数据库不提供此元数据，则必须提供参数的显式配置。

### RDBMS对象
```RDBMS对象```(包括```MappingSqlQuery```、```SqlUpdate```和```StoredProcedure```)要求在数据访问层初始化期间创建可重用和线程安全的对象。此方法仿效JDO查询，定义查询字符串、声明参数并编译查询。完成此操作后，请执行```execute(​)```，```update(​)```和```FindObject(​)```方法可以使用各种参数值多次调用。

## JDBC包结构
Spring Framework的JDBC抽象框架由四个不同的包组成：
+ core
+ datasource
+ object
+ support

## !!!JDBC核心类的使用
### !!!JdbcTemplate
JdbcTemplate是JDBC核心包中的核心类。
它处理资源的创建和释放，从而帮助避免常见错误，如忘记关闭连接。
它执行核心JDBC工作流的基本任务(如语句创建和执行)，让应用程序代码提供SQL和提取结果

JdbcTemplate类：
+ 运行SQL查询。
+ 更新语句和存储过程调用。
+ 对ResultSet实例执行迭代并提取返回的参数值。
+ 捕获JDBC异常并将其转换为org.springframework.dao包中定义的通用的、信息量更大的异常层次结构。

当在代码中使用JdbcTemplate时，只需要实现回调接口，从而为它们提供一个明确定义的约定。
给定JdbcTemplate类提供的连接，PreparedStatementCreator回调接口创建一条准备好的语句，提供SQL和任何必要的参数。
CallableStatementCreator接口也是如此，它创建可调用的语句。RowCallbackHandler接口从ResultSet的每一行提取值。

#### Querying(查询)
获取关系中的行数：
```
int rowCount = this.jdbcTemplate.queryForObject("select count(*) from t_actor", Integer.class);
```

使用绑定变量：
```
int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject(
        "select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
```

查找字符串：
```
String lastName = this.jdbcTemplate.queryForObject(
        "select last_name from t_actor where id = ?",
        String.class, 1212L);
```

查找并填充单个域对象：
```
Actor actor = jdbcTemplate.queryForObject(
        "select first_name, last_name from t_actor where id = ?",
        (resultSet, rowNum) -> {
            Actor newActor = new Actor();
            newActor.setFirstName(resultSet.getString("first_name"));
            newActor.setLastName(resultSet.getString("last_name"));
            return newActor;
        },
        1212L);
```

查找并填充域对象列表：
```
List<Actor> actors = this.jdbcTemplate.query(
        "select first_name, last_name from t_actor",
        (resultSet, rowNum) -> {
            Actor actor = new Actor();
            actor.setFirstName(resultSet.getString("first_name"));
            actor.setLastName(resultSet.getString("last_name"));
            return actor;
        });
```

如果最后两段代码实际存在于同一应用程序中，那么删除两个```RowMapper```lambda表达式中的重复项并将其提取到单个字段中将是有意义的，然后DAO方法可以根据需要引用该字段。例如，编写前面的代码片段可能更好，如下所示：
```
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
    Actor actor = new Actor();
    actor.setFirstName(resultSet.getString("first_name"));
    actor.setLastName(resultSet.getString("last_name"));
    return actor;
};

public List<Actor> findAllActors() {
    return this.jdbcTemplate.query( "select first_name, last_name from t_actor", actorRowMapper);
}
```

#### Updating (插入、更新和删除)
插入一个新记录：
```
this.jdbcTemplate.update(
        "insert into t_actor (first_name, last_name) values (?, ?)",
        "Leonor", "Watling");
```

更新一条现有记录：
```
This.jdbcTemplate.update(。
“UPDATE t_ACTOR SET LAST_NAME=？其中id=？”，
“班卓琴”，5276L)；
```

删除记录：
```
this.jdbcTemplate.update(
        "delete from t_actor where id = ?",
        Long.valueOf(actorId));
```

#### 其他操作
可以使用execute()方法来运行任意SQL。
该方法还可以用于DDL语句。它被接受回调接口、绑定变量数组等的变量严重重载。
下面的示例创建一个表：
```
this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

下面的示例调用一个存储过程：
```
this.jdbcTemplate.update(
        "call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
        Long.valueOf(unionId));
```

#### 最佳实践
JdbcTemplate类的实例就是线程安全的，这意味着可以配置JdbcTemplate的单个实例，然后安全地将该共享引用注入多个DAO(或存储库)。JdbcTemplate是有状态的，因为它维护对数据源的引用，但此状态不是会话状态。

使用JdbcTemplate类时的常见做法是在Spring配置文件中配置一个DataSource，然后将该共享DataSource bean依赖项注入到DAO类中。JdbcTemplate是在数据源的setter中创建的。这将导致类似以下内容的DAO：
```
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

以下示例显示了相应的XML配置：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="corporateEventDao" class="com.example.JdbcCorporateEventDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

显式配置的另一种选择是使用组件扫描和注释支持进行依赖注入。
在这种情况下，可以用@Repository注释类(这使它成为组件扫描的候选对象)，并用@Autwire注释DataSource setter方法。以下示例显示如何执行此操作：
```
@Repository 
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    @Autowired 
    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource); 
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

以下示例显示了相应的XML配置：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Scans within the base package of the application for @Component classes to configure as beans -->
    <context:component-scan base-package="org.springframework.docs.test" />

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```


无论选择使用(或不使用)上述哪种模板初始化样式，很少需要在每次运行SQL时创建JdbcTemplate类的新实例。
一旦配置好，JdbcTemplate实例就是线程安全的。如果的应用程序访问多个数据库，可能需要多个JdbcTemplate实例，这需要多个数据源，随后需要多个配置不同的JdbcTemplate实例。

### !!!NamedParameterJdbcTemplate
```NamedParameterJdbcTemplate```类添加了对使用命名参数编程JDBC语句的支持，而不是仅使用经典占位符(?)编程JDBC语句。
```NamedParameterJdbcTemplate```类包装JdbcTemplate，并委托包装的JdbcTemplate执行大部分工作。

以下示例显示如何使用NamedParameterJdbcTemplate：
```
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {
	
    String sql = "select count(*) from T_ACTOR where first_name = :first_name";
    SqlParameterSource namedParameters = new MapSqlParameterSource("first_name", firstName);
    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```
请注意，在分配给SQL变量的值和插入namedParameters变量(类型为MapSqlParameterSource)的相应值中使用了命名参数表示法。
或者，可以使用基于映射的样式将命名参数及其相应值传递给NamedParameterJdbcTemplate实例。

下例显示了基于Map的样式的使用：
```
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

    String sql = "select count(*) from T_ACTOR where first_name = :first_name";
    Map<String, String> namedParameters = Collections.singletonMap("first_name", firstName);
    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters,  Integer.class);
}
```

与NamedParameterJdbcTemplate(存在于同一个Java包中)相关的一个很好的特性是SqlParameterSource接口。
SqlParameterSource是NamedParameterJdbcTemplate的命名参数值的源。
MapSqlParameterSource类是一个简单的实现，它是java.util.Map周围的适配器，其中键是参数名称，值是参数值。
另一个实现是BeanPropertySqlParameterSource类，该类包装任意的JavaBean，并使用包装的JavaBean的属性作为命名参数值的来源。

下面的示例使用NamedParameterJdbcTemplate返回上一个示例中显示的类的成员计数：
```
public class Actor {

    private Long id;
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return this.firstName;
    }

    public String getLastName() {
        return this.lastName;
    }

    public Long getId() {
        return this.id;
    }

    // setters omitted...

}
```
```
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActors(Actor exampleActor) {

    // notice how the named parameters match the properties of the above 'Actor' class
    String sql = "select count(*) from T_ACTOR where first_name = :firstName and last_name = :lastName";

    SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(exampleActor);

    return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

NamedParameterJdbcTemplate类包装了一个经典的JdbcTemplate模板。
如果需要访问包装的JdbcTemplate实例来访问仅存在于JdbcTemplate类中的功能，则可以使用getJdbcOperations()方法通过JdbcOperations接口访问包装的JdbcTemplate。

### SQLExceptionTranslator
```SQLExceptionTranslator```是一个由类实现的接口，这些类可以在SQLExceptions和Spring自己的org.springframework work.dao.DataAccessException之间进行转换，后者在数据访问策略方面是不可知的。实现可以是通用的(例如，使用JDBC的SQLState代码)，也可以是专有的(例如，使用Oracle错误代码)，以获得更高的精度。

SQLErrorCodeSQLExceptionTranslator是默认使用的SQLExceptionTranslator的实现。
此实施使用特定的供应商代码。它比SQLState实现更精确。错误代码转换基于名为SQLErrorCodes的JavaBean类型类中保存的代码。
此类由SQLErrorCodesFactory创建和填充，顾名思义，SQLErrorCodesFactory是根据名为sql-error-codes.xml的配置文件的内容创建SQLErrorCodes的工厂。此文件由供应商代码填充，并基于从DatabaseMetaData获取的DatabaseProductName。将使用正在使用的实际数据库的代码。

SQLErrorCodeSQLExceptionTranslator按以下顺序应用匹配规则：
+ 由子类实现的任何自定义转换。通常，使用提供的具体SQLErrorCodeSQLExceptionTranslator，因此此规则不适用。只有当实际提供了一个子类实现时，它才适用。
+ 作为SQLErrorCodes类的customSqlExceptionTranslator属性提供的SQLExceptionTranslator接口的任何自定义实现。
+ 将在CustomSQLErrorCodesTranslate类的实例列表(为SQLErrorCodes类的customTranslations属性提供)中搜索匹配项。
+ 应用了错误代码匹配。
+ 使用备用转换器。SQLExceptionSubclassTranslator是默认的后备转换器。如果此转换不可用，则下一个备用转换程序是SQLStateSQLExceptionTranslator。

可以扩展SQLErrorCodeSQLExceptionTranslator，如下例所示：
```
public class CustomSQLErrorCodesTranslator extends SQLErrorCodeSQLExceptionTranslator {

    protected DataAccessException customTranslate(String task, String sql, SQLException sqlEx) {
        if (sqlEx.getErrorCode() == -12345) {
            return new DeadlockLoserDataAccessException(task, sqlEx);
        }
        return null;
    }
}
```

在前面的示例中，将转换特定错误代码(-12345)，而将其他错误留给默认转换器实现进行转换。
要使用此自定义转换器，必须通过方法setExceptionTranslator将其传递给JdbcTemplate，并且必须在需要此转换器的所有数据访问处理中使用此JdbcTemplate。
以下示例显示如何使用此自定义翻译程序：
```
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {

    // create a JdbcTemplate and set data source
    this.jdbcTemplate = new JdbcTemplate();
    this.jdbcTemplate.setDataSource(dataSource);

    // create a custom translator and set the DataSource for the default translation lookup
    CustomSQLErrorCodesTranslator tr = new CustomSQLErrorCodesTranslator();
    tr.setDataSource(dataSource);
    this.jdbcTemplate.setExceptionTranslator(tr);

}

public void updateShippingCharge(long orderId, long pct) {
    // use the prepared JdbcTemplate for this update
    this.jdbcTemplate.update("update orders" +
        " set shipping_charge = shipping_charge * ? / 100" +
        " where id = ?", pct, orderId);
}
```

### Running Statements
运行SQL语句只需要很少的代码。需要一个DataSource和一个JdbcTemplate，包括JdbcTemplate提供的方便方法。

下面的示例显示了创建新表的最小但功能齐全的类需要包括的内容，使用```execute()```：
```
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAStatement {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void doExecute() {
        this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
    }
}
```


### Running Queries
某些查询方法返回单个值。
要从一行检索计数或特定值，请使用```queryForObject()```方法。后者将返回的JDBC```Type```转换为作为参数传入的Java类。
如果类型转换无效，则引发InvalidDataAccessApiUsageException。

下面的示例包含两个查询方法，一个用于int，另一个用于查询字符串：
```
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class RunAQuery {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int getCount() {
        return this.jdbcTemplate.queryForObject("select count(*) from mytable", Integer.class);
    }

    public String getName() {
        return this.jdbcTemplate.queryForObject("select name from mytable", String.class);
    }
}
```

除了单个结果查询方法之外，还有几个方法返回一个列表，其中包含查询返回的每一行的条目。
最通用的方法是```queryForList()```，它返回一个列表，其中每个元素都是一个Map，使用列名作为键，每列包含一个条目。
如果在前面的示例中添加一个方法来检索所有行的列表，则可能如下所示：
```
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<Map<String, Object>> getList() {
    return this.jdbcTemplate.queryForList("select * from mytable");
}
```
返回的列表如下所示：
```
[{name=Bob, id=1}, {name=Mary, id=2}]
```

### Updating the Database
下面的示例更新特定主键的列，适用```update()```：
```
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAnUpdate {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void setName(int id, String name) {
        this.jdbcTemplate.update("update mytable set name = ? where id = ?", name, id);
    }
}
```

在前面的示例中，SQL语句具有行参数的占位符。
可以将参数值作为varargs传递，也可以作为对象数组传递。
因此，应该显式地将基元包装在基元包装类中，或者应该使用自动装箱。

### Retrieving Auto-generated Keys 检索自增主键
```update()```方法支持检索数据库生成的主键。此支持是JDBC 3.0标准的一部分。
该方法将```PreparedStatementCreator```作为其第一个参数，这是指定所需INSERT语句的方式。
另一个参数是```KeyHolder```，它包含从更新成功返回时生成的自增主键。
没有标准的单一方法来创建适当的```PreparedStatement```。

以下示例可以在Oracle上运行，但可能不能在其他平台上运行：
```
final String INSERT_SQL = "insert into my_test (name) values(?)";
final String name = "Rob";

KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(connection -> {
    PreparedStatement ps = connection.prepareStatement(INSERT_SQL, new String[] { "id" });
    ps.setString(1, name);
    return ps;
}, keyHolder);

// 现在可以通过 keyHolder.getKey() 获取主键
```

## !!!数据库连接管理
### !!!DataSource
Spring通过```DataSource```获得到数据库的连接。
```DataSource```是JDBC规范的一部分，是通用连接工厂。它允许容器或框架对应用程序代码隐藏连接池和事务管理问题。

当使用Spring的JDBC层时，可以从JNDI获取数据源，也可以使用第三方提供的连接池实现来配置自己的数据源。
对于传统的选择是带有Bean样式的DataSource类的Apache Commons DBCP和C3P0；
对于现代的JDBC连接池，可以考虑使用HikariCP的构建器样式的API。

注意，DriverManagerDataSource和SimpleDriverDataSource类应该仅用于测试目的，这些变体不提供池，并且在发出多个连接请求时性能很差。

要配置```DriverManagerDataSource```，请执行以下操作：
+ 获得与```DriverManagerDataSource```的连接，就像通常获得JDBC连接一样。
+ 指定JDBC驱动程序的完全限定类名，以便```DriverManager```可以加载驱动程序类。
+ 提供一个因JDBC驱动程序而异的URL。
+ 提供用户名和密码以连接到数据库。

以下示例显示如何在Java中配置```DriverManagerDataSource```：
```
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
dataSource.setUsername("sa");
dataSource.setPassword("");
```

以下示例显示了相应的XML配置：
```
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

以下示例显示```DBCP配置```：
```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

以下示例显示了C3P0配置：
```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
    <property name="driverClass" value="${jdbc.driverClassName}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

### DataSourceUtils
```DataSourceUtils```类是一个方便而强大的工具类，它提供静态方法来从JNDI获取连接，并在必要时关闭连接。
例如，它支持与```DataSourceTransactionManager```的线程绑定连接。

### SmartDataSource
```SmartDataSource```接口应该由能够提供到关系数据库的连接的类来实现。
它扩展了```DataSource```接口，让使用它的类查询在给定操作后是否应该关闭连接。
当需要重用连接时，这种用法非常有效。

### AbstractDataSource
```AbstractDataSource```是Spring的```DataSource```实现的抽象基类。它实现了所有```DataSource```实现通用的代码。
如果编写自己的```DataSource```实现，则应该扩展```AbstractDataSource```类。

### SingleConnectionDataSource
```SingleConnectionDataSource```类是```SmartDataSource```接口的实现，该接口封装了一个单连接，每次使用后不会关闭。这不支持多线程。

如果任何客户端代码在假定池连接的情况下调用Close，则应将```suppressClose```属性设置为true。此设置返回包装物理连接的关闭抑制代理。请注意，不能再将其强制转换为本机Oracle连接或类似对象。

```SingleConnectionDataSource```主要是一个测试类。与简单的JNDI环境相结合，它通常支持在应用程序服务器之外轻松测试代码。与```DriverManagerDataSource```不同，它始终重复使用相同的连接，避免过度创建物理连接。

### DriverManagerDataSource
```DriverManagerDataSource```类是标准```DataSource```接口的实现，该接口通过bean属性配置普通JDBC驱动程序，并每次返回一个新连接。

该实现对于Java EE容器外部的测试和独立环境非常有用，可以作为Spring IOC容器中的DataSource bean，也可以与简单的JNDI环境结合使用。
池连接调用```Connection.close()```关闭连接，因此任何支持DataSource的持久性代码都应该可以工作。
使用JavaBean风格的连接池(如Commons-DBCP)非常简单，即使在测试环境中，使用这样的连接池几乎总是比使用DriverManagerDataSource更可取。

### TransactionAwareDataSourceProxy
TransactionAwareDataSourceProxy是目标数据源的代理。
代理包装目标数据源，以增加对Spring管理事务的感知。在这方面，它类似于由JavaEE服务器提供的事务性JNDI数据源。

### DataSourceTransactionManager
DataSourceTransactionManager类是单个JDBC数据源的PlatformTransactionManager实现。
它将指定数据源的JDBC连接绑定到当前执行的线程，从而潜在地允许每个数据源有一个线程连接。



## !!!JDBC批处理操作
如果批处理对同一预准备语句的多个调用，大多数JDBC驱动程序会提高性能。
通过将更新分组为批处理，可以限制数据库的往返次数。

### 基本批处理操作
要完成JdbcTemplate批处理，需要实现特殊接口```BatchPreparedStatementSetter```的两个方法，并将该实现作为```batchUpdate```方法调用中的第二个参数传入。
可以使用```getBatchSize```方法提供当前批次的大小。
可以使用```setValues```方法设置预准备语句的参数值。
此方法的调用次数与在```getBatchSize```调用中指定的次数相同。

下面的示例根据列表中的条目更新t_action表，并将整个列表用作批处理：
```
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[] batchUpdate(final List<Actor> actors) {
        return this.jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                new BatchPreparedStatementSetter() {
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        Actor actor = actors.get(i);
                        ps.setString(1, actor.getFirstName());
                        ps.setString(2, actor.getLastName());
                        ps.setLong(3, actor.getId().longValue());
                    }
                    public int getBatchSize() {
                        return actors.size();
                    }
                });
    }

    // ... additional methods
}
```
如果处理更新流或从文件读取，可能有一个首选的批处理大小，但最后一批可能没有该数量的条目。
在这种情况下，可以使用```InterruptibleBatchPreparedStatementSetter```接口，该接口允许在输入源耗尽后中断批处理。
```isBatchExhausted```方法允许发出批次结束的信号。

### 使用对象列表的批处理操作
```JdbcTemplate```和```NamedParameterJdbcTemplate```都提供了另一种提供批量更新的方法。
可以将调用中的所有参数值作为列表提供，而不是实现特殊的批处理接口。该框架循环遍历这些值，并使用内部预准备语句设置器。
根据是否使用命名参数，API会有所不同。对于命名参数，需要提供```SqlParameterSource```数组，批处理的每个成员都有一个条目。
可以使用```SqlParameterSourceUtils.createBatch```方便的方法来创建此数组，传入一组bean样式的对象、字符串键Map实例或两者的组合。

以下示例显示了使用命名参数的批量更新：
```
public class JdbcActorDao implements ActorDao {

    private NamedParameterTemplate namedParameterJdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
    }

    public int[] batchUpdate(List<Actor> actors) {
        return this.namedParameterJdbcTemplate.batchUpdate(
                "update t_actor set first_name = :firstName, last_name = :lastName where id = :id",
                SqlParameterSourceUtils.createBatch(actors));
    }

    // ... additional methods
}
```

对于使用经典的```?```占位符SQL语句，则传入一个包含具有更新值的对象数组的列表。
对于SQL语句中的每个占位符，此对象数组必须有一个条目，并且它们的顺序必须与在SQL语句中定义的顺序相同。
下面的示例与前面的示例相同，只是它使用了经典的```?```占位符：
```
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[] batchUpdate(final List<Actor> actors) {
        List<Object[]> batch = new ArrayList<Object[]>();
        for (Actor actor : actors) {
            Object[] values = new Object[] {
                    actor.getFirstName(), actor.getLastName(), actor.getId()};
            batch.add(values);
        }
        return this.jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                batch);
    }

    // ... additional methods
}
```

我们前面描述的所有批处理更新方法都返回一个int数组，其中包含每个批处理条目受影响的行数。
此计数由JDBC驱动程序报告。
如果计数不可用，JDBC驱动程序将返回值-2。

### 使用多批次的批处理操作
前面的批处理更新示例处理的批太大，以至于您想要将它们拆分成几个较小的批。
您可以使用前面提到的方法通过多次调用```batchUpdate```方法来实现这一点，但是现在有了一个更方便的方法。
除SQL语句外，此方法还采用一个包含参数的对象集合、要为每个批处理进行的更新次数，以及一个```ParameterizedPreparedStatementSetter```来设置准备好的语句的参数值。
该框架遍历提供的值，并将更新调用分成指定大小的批。

以下示例显示批处理大小为100的批处理更新：
```
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int[][] batchUpdate(final Collection<Actor> actors) {
        int[][] updateCounts = jdbcTemplate.batchUpdate(
                "update t_actor set first_name = ?, last_name = ? where id = ?",
                actors,
                100,
                (PreparedStatement ps, Actor actor) -> {
                    ps.setString(1, actor.getFirstName());
                    ps.setString(2, actor.getLastName());
                    ps.setLong(3, actor.getId().longValue());
                });
        return updateCounts;
    }

    // ... additional methods
}
```
此调用的批处理更新方法返回一个整型数组数组，其中包含每个批次的数组条目，以及每个更新受影响的行数组。
顶级数组的长度表示批处理运行的数量，第二级数组的长度表示该批处理中的更新次数。
每个批次中的更新次数应为为所有批次提供的批次大小(最后一个批次可能较少)，具体取决于提供的更新对象总数。
每条UPDATE语句的更新计数是JDBC驱动程序报告的更新计数。如果计数不可用，JDBC驱动程序将返回值-2。

## 使用SimpleJdbc类简化JDBC操作
```SimpleJdbcInsert```和```SimpleJdbcCall```类利用可通过JDBC驱动程序检索的数据库元数据提供简化的配置。
这意味着您需要预先配置的较少，尽管如果您希望在代码中提供所有详细信息，则可以覆盖或关闭元数据处理。

### 使用SimpleJdbcInsert插入数据
我们首先查看具有最少配置选项的SimpleJdbcInsert类。
您应该在数据访问层的初始化方法中实例化SimpleJdbcInsert。对于本例，初始化方法是setDataSource方法。
您不需要将SimpleJdbcInsert类划分为子类。相反，您可以使用```withTableName```方法创建一个新实例并设置表名。
该类的配置方法遵循Fluid样式，该样式返回SimpleJdbcInsert的实例，允许您链接所有配置方法。

以下示例仅使用一种配置方法：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource).withTableName("t_actor");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(3);
        parameters.put("id", actor.getId());
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        insertActor.execute(parameters);
    }
}
```
这里使用的Execute方法将普通的java.util.Map作为其唯一参数。这里需要注意的重要一点是，Map使用的键必须与数据库中定义的表的列名相匹配。这是因为我们读取元数据来构造实际的INSERT语句。

### 使用SimpleJdbcInsert检索自增主键
下一个示例使用与上一个示例相同的INSERT，但是它没有传入id，而是检索自动生成的键并将其设置在新的Actor对象上。
在创建SimpleJdbcInsert时，除了指定表名外，还使用```usingGeneratedKeyColumns```方法指定生成的键列的名称。

下面的清单显示了它的工作原理：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(2);
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }
}
```

使用第二种方法运行INSERT时，主要不同之处在于不将id添加到Map，而是调用```executeAndReturnKey```方法。
这将返回一个java.lang.Number对象，您可以使用该对象创建域类中使用的数字类型的实例。在这里，不能依赖所有数据库来返回特定的Java类。Number是您可以依赖的基类。
如果有多个自动生成的列，或者生成的值不是数字，则可以使用从```executeAndReturnKeyHolder```方法返回的KeyHolder。

### 指定SimpleJdbcInsert的列
可以通过使用usingColumns方法指定列名列表来限制插入的列，如下面的示例所示：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingColumns("first_name", "last_name")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(2);
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```

### 使用SqlParameterSource提供参数值
使用Map提供参数值很好，但它不是最方便使用的类。
Spring提供了两个```SqlParameterSource```接口的实现，您可以改为使用它们。

第一个是```BeanPropertySqlParameterSource```，如果您有一个与JavaBean兼容的包含您的值的类，那么它是一个非常方便的类。它使用相应的getter方法来提取参数值。
以下示例显示如何使用BeanPropertySqlParameterSource：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        SqlParameterSource parameters = new BeanPropertySqlParameterSource(actor);
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}	
```

另一个是```MapSqlParameterSource```，它类似于Map，但提供了一个更方便的可以链接的AddValue方法。
以下示例显示如何使用它：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.insertActor = new SimpleJdbcInsert(dataSource)
                .withTableName("t_actor")
                .usingGeneratedKeyColumns("id");
    }

    public void add(Actor actor) {
        SqlParameterSource parameters = new MapSqlParameterSource()
                .addValue("first_name", actor.getFirstName())
                .addValue("last_name", actor.getLastName());
        Number newId = insertActor.executeAndReturnKey(parameters);
        actor.setId(newId.longValue());
    }

    // ... additional methods
}
```
如您所见，配置是相同的。只有执行代码必须更改才能使用这些替代输入类。

### 使用SimpleJdbcCall调用存储过程
SimpleJdbcCall类使用数据库中的元数据来查找In和Out参数的名称，因此您不必显式声明它们。
如果您愿意这样做，或者如果您的参数(如ARRAY或STRUCT)没有自动映射到Java类，则可以声明参数。

第一个示例显示了一个简单的过程，该过程仅从MySQL数据库返回VARCHAR和DATE格式的标量值。
示例过程读取指定的执行元条目，并以out参数的形式返回First_Name、Last_Name和Birth_Date列。
下面的清单显示了第一个示例：
```
CREATE PROCEDURE read_actor (
    IN in_id INTEGER,
    OUT out_first_name VARCHAR(100),
    OUT out_last_name VARCHAR(100),
    OUT out_birth_date DATE)
BEGIN
    SELECT first_name, last_name, birth_date
    INTO out_first_name, out_last_name, out_birth_date
    FROM t_actor where id = in_id;
END;
```
In_id参数包含您正在查找的执行元的ID。OUT参数返回从表中读取的数据。

您可以用与声明SimpleJdbcInsert类似的方式声明SimpleJdbcCall。
您应该在数据访问层的初始化方法中实例化和配置该类。与StoredProcedure类相比，您不需要创建子类，也不需要声明可以在数据库元数据中查找的参数。
下面的SimpleJdbcCall配置示例使用前面的存储过程(除数据源之外，唯一的配置选项是存储过程的名称)：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        this.procReadActor = new SimpleJdbcCall(dataSource)
                .withProcedureName("read_actor");
    }

    public Actor readActor(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        Map out = procReadActor.execute(in);
        Actor actor = new Actor();
        actor.setId(id);
        actor.setFirstName((String) out.get("out_first_name"));
        actor.setLastName((String) out.get("out_last_name"));
        actor.setBirthDate((Date) out.get("out_birth_date"));
        return actor;
    }

    // ... additional methods
}
```

### 显式声明要用于SimpleJdbcCall的参数
在本章早些时候，我们描述了如何从元数据推导参数，但如果愿意，您可以显式声明参数。
为此，可以使用```decreParameters```方法创建和配置```SimpleJdbcCall```，该方法接受数量可变的```SqlParameter```对象作为输入。

您可以选择显式声明一个、部分或所有参数。在未显式声明参数的情况下，仍使用参数元数据。
若要绕过对潜在参数的元数据查找的所有处理，而只使用已声明的参数，可以调用```withoutProcedureColumnMetaDataAccess ```方法作为声明的一部分。假设您为数据库函数声明了两个或多个不同的调用签名。
在本例中，您可以调用```useInParameterNames```来指定要包含在给定签名中的IN参数名列表。

下面的示例显示一个完全声明的过程调用，并使用上一个示例中的信息：
```
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor")
                .withoutProcedureColumnMetaDataAccess()
                .useInParameterNames("in_id")
                .declareParameters(
                        new SqlParameter("in_id", Types.NUMERIC),
                        new SqlOutParameter("out_first_name", Types.VARCHAR),
                        new SqlOutParameter("out_last_name", Types.VARCHAR),
                        new SqlOutParameter("out_birth_date", Types.DATE)
                );
    }

    // ... additional methods
}
```

### 如何定义SqlParameters
要为SimpleJdbc类和RDBMS操作类定义参数，可以使用SqlParameter或其一个子类。
为此，通常需要在构造函数中指定参数名称和SQL类型。SQL类型是使用java.sql.Types常量指定的。

在本章的前面部分，我们看到了类似以下内容的声明：
```
new SqlParameter("in_id", Types.NUMERIC),
new SqlOutParameter("out_first_name", Types.VARCHAR),
```
SqlParameter的第一行声明一个IN参数。通过使用SqlQuery及其子类(在了解SqlQuery中介绍)，可以将IN参数用于存储过程调用和查询。

第二行(带有SqlOutParameter)声明要在存储过程调用中使用的OUT参数。还有一个SqlInOutParameter用于InOut参数(向过程提供IN值并且也返回值的参数)。


### 使用SimpleJdbcCall调用存储函数
略

### 从SimpleJdbcCall返回ResultSet或Ref游标
略

## 将JDBC操作建模为Java对象
程序包```org.springframework.jdbc.object```包含允许您以更面向对象的方式访问数据库的类。
可以运行查询并将结果作为包含业务对象的列表返回，该列表将关系列数据映射到业务对象的属性。
还可以运行存储过程以及运行UPDATE、DELETE和INSERT语句。

### 了解SqlQuery
### 使用MappingSqlQuery
### 使用SqlUpdate
### 使用StoredProcedure

## 参数和数据处理的常见问题
参数和数据值的常见问题存在于Spring Framework的JDBC支持提供的不同方法中。

### 提供参数的SQL类型信息
### 处理BLOB和CLOB对象
### 传入IN子句的值列表
### 处理存储过程调用的复杂类型

## !!!嵌入式数据库支持
程序包```org.springframework.jdbc.datasource.embedded```提供对嵌入式Java数据库引擎的支持。
Spring已经提供了对HSQL、H2和Derby的支持，还可以使用可扩展API来插入新的嵌入式数据库类型和数据源实现。

### 为什么要使用嵌入式数据库
嵌入式数据库在项目的开发阶段非常有用，因为它具有轻量级的特性。
好处包括易于配置、快速启动、可测试性以及在开发过程中快速发展SQL的能力。

### 使用XML配置创建嵌入式数据库
如果希望将嵌入式数据库实例作为Spring```ApplicationContext```中的bean公开，则可以在```spring-jdbc```命名空间中使用```embedded-database```标记：
```
<jdbc:embedded-database id="dataSource" generate-name="true">
    <jdbc:script location="classpath:schema.sql"/>
    <jdbc:script location="classpath:test-data.sql"/>
</jdbc:embedded-database>
```
前面的配置创建了一个嵌入式HSQL数据库，该数据库由类路径根目录下的```schema.sql```和```test-data.sql```资源中的SQL填充。此外，最佳做法是为嵌入式数据库分配一个唯一生成的名称。嵌入的数据库作为```javax.sql.DataSource```类型的bean对Spring容器可用，然后可以根据需要注入到数据访问对象中。

### 使用编程方式创建嵌入式数据库
EmbeddedDatabaseBuilder类为以编程方式构建嵌入式数据库提供了API。
当需要在独立环境或独立集成测试中创建嵌入式数据库时，可以使用此选项，如下例所示：
```
EmbeddedDatabase db = new EmbeddedDatabaseBuilder()
        .generateUniqueName(true)
        .setType(H2)
        .setScriptEncoding("UTF-8")
        .ignoreFailedDrops(true)
        .addScript("schema.sql")
        .addScripts("user_data.sql", "country_data.sql")
        .build();

// perform actions against the db (EmbeddedDatabase extends javax.sql.DataSource)

db.shutdown()
```

您还可以使用EmbeddedDatabaseBuilder通过Java配置创建嵌入式数据库，如下例所示：
```
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .generateUniqueName(true)
                .setType(H2)
                .setScriptEncoding("UTF-8")
                .ignoreFailedDrops(true)
                .addScript("schema.sql")
                .addScripts("user_data.sql", "country_data.sql")
                .build();
    }
}
```

### 选择嵌入式数据库类型
本节介绍如何从Spring支持的三个嵌入式数据库中选择一个。

#### HSQL
Spring支持HSQL1.8.0及以上版本。
如果没有显式指定类型，则HSQL是默认的嵌入式数据库。
要显式指定HSQL，请将嵌入式数据库标记的type属性设置为hsql。
如果使用构建器API，请调用setType(EmbeddedDatabaseType)方法，传入EmbeddedDatabaseType.HSQL。

#### H2
Spring支持H2数据库。
要启用h2，请将嵌入式数据库标记的type属性设置为h2。
如果使用构建器API，请调用setType(EmbeddedDatabaseType)方法，传入EmbeddedDatabaseType.H2。

#### Derby
Spring支持Apache Derby 10.5及更高版本。
要启用Derby，请将Embedded-database标记的type属性设置为Derby。
如果使用构建器API，请调用setType(EmbeddedDatabaseType)方法，传入EmbeddedDatabaseType.DERBY。

### 使用嵌入式数据库测试数据访问逻辑
嵌入式数据库提供了一种测试数据访问代码的轻量级方法。
下一个示例是使用嵌入式数据库的数据访问集成测试模板。当嵌入式数据库不需要跨测试类重用时，使用这样的模板对于一次性很有用。
但是，如果您希望创建在测试套件中共享的嵌入式数据库，请考虑使用Spring TestContext框架并将嵌入式数据库配置为Spring ApplicationContext中的Bean，如Creating an Embedded Database by Using Spring XML and Creating an Embedded Database Programming中所述。
下面的清单显示了测试模板：
```
public class DataAccessIntegrationTestTemplate {

    private EmbeddedDatabase db;

    @BeforeEach
    public void setUp() {
        // creates an HSQL in-memory database populated from default scripts
        // classpath:schema.sql and classpath:data.sql
        db = new EmbeddedDatabaseBuilder()
                .generateUniqueName(true)
                .addDefaultScripts()
                .build();
    }

    @Test
    public void testDataAccess() {
        JdbcTemplate template = new JdbcTemplate(db);
        template.query( /* ... */ );
    }

    @AfterEach
    public void tearDown() {
        db.shutdown();
    }

}
```

### 为嵌入式数据库生成唯一名称
如果开发团队的测试套件无意中尝试重新创建同一数据库的其他实例，则开发团队经常会遇到嵌入式数据库的错误。如果XML配置文件或@Configuration类负责创建嵌入式数据库，然后在同一测试套件(即，在同一个 - 进程内)内的多个测试场景中重用相应的配置，这很容易实现。例如，针对嵌入式数据库的集成测试，这些数据库的ApplicationContext配置仅在哪些bean定义概要文件处于活动状态方面有所不同。

此类错误的根本原因是Spring的EmbeddedDatabaseFactory(由```<jdbc：Embedded-database>```XML名称空间元素和EmbeddedDatabaseBuilder for Java配置在内部使用)将嵌入式数据库的名称设置为testdb(如果没有指定的话)。对于```<jdbc：bedded-database>```，嵌入式数据库通常被分配一个与bean的id相同的名称(通常类似于dataSource)。因此，后续创建嵌入式数据库的尝试不会产生新数据库。相反，相同的JDBC连接URL被重用，并且尝试创建新的嵌入式数据库实际上指向从相同配置创建的现有嵌入式数据库。

为了解决这个常见问题，Spring Framework4.2支持为嵌入式数据库生成唯一名称。要启用生成的名称，请使用以下选项之一。
+ EmbeddedDatabaseFactory.setGenerateUniqueDatabaseName()
+ EmbeddedDatabaseBuilder.generateUniqueName()
+ <jdbc:embedded-database generate-name="true" …​ >

### 扩展嵌入式数据库
您可以通过两种方式扩展Spring JDBC嵌入式数据库支持：
+ 实现EmbeddedDatabaseConfigurer以支持新的嵌入式数据库类型。
+ 实现DataSourceFactory以支持新的数据源实现，例如管理嵌入式数据库连接的连接池。

## 初始化数据源
包```org.springframework.jdbc.datasource.init```为初始化现有的数据源提供支持。
嵌入式数据库支持为应用程序创建和初始化数据源提供了一种选择。但是，有时可能需要初始化在某个服务器上运行的实例。

### 通过XML初始化数据源
如果要初始化数据库，并且可以提供对```DataSource```Bean的引用，则可以在```spring-jdbc```命名空间中使用```initialize-database```标记：
```
<jdbc:initialize-database data-source="dataSource">
    <jdbc:script location="classpath:com/foo/sql/db-schema.sql"/>
    <jdbc:script location="classpath:com/foo/sql/db-test-data.sql"/>
</jdbc:initialize-database>
```
前面的示例针对数据库运行两个指定的脚本。
第一个脚本创建一个模式，第二个脚本用测试数据集填充表。
脚本位置也可以是带有通配符的模式，其通配符采用用于Spring中资源的常用Ant样式(例如，classpath*：/com/foo/**/sql/*-data.sql)。
如果使用模式，脚本将按照其URL或文件名的词汇顺序运行。

数据库初始值设定项的默认行为是无条件运行提供的脚本。
例如，如果您对已有测试数据的数据库运行脚本，这可能不是您想要的。
通过遵循先创建表，然后插入数据的常见模式(如前所述)，可以降低意外删除数据的可能性。如果表已经存在，则第一步失败。

但是，为了更好地控制现有数据的创建和删除，XML命名空间提供了一些附加选项。
第一个是打开和关闭初始化的标志。您可以根据环境进行设置(例如从系统属性或环境bean中提取布尔值)。
下面的示例从系统属性中获取一个值：
```
<jdbc:initialize-database data-source="dataSource"
    enabled="#{systemProperties.INITIALIZE_DATABASE}"> 
    <jdbc:script location="..."/>
</jdbc:initialize-database>
```

控制现有数据的第二个选项是提高故障容忍度。为此，您可以控制初始化器忽略其从脚本运行的SQL中的某些错误的能力，如下面的示例所示：
```
<jdbc:initialize-database data-source="dataSource" ignore-failures="DROPS">
    <jdbc:script location="..."/>
</jdbc:initialize-database>
```

# Data Access with R2DBC 使用R2DBC进行数据访问
R2DBC("Reactive Relational Database Connectivity")是一个由社区驱动的规范工作，旨在使用反应式模式标准化对SQL数据库的访问。

略

# Retrieving Auto-generated Keys 检索自增主键
略

# !!!Object Relational Mapping (ORM) Data Access 对象关系映射(ORM)数据访问
本节介绍使用对象关系映射(ORM)时的数据访问。

## 基于Spring的ORM
Spring框架支持与Java持久性API(Java Persistence API, JPA)集成，并支持用于资源管理、数据访问对象(DAO)实现和事务策略的本机Hibernate。
例如，对于Hibernate，有几个方便的IoC特性提供了一流的支持，这些特性解决了许多典型的Hibernate集成问题。
您可以通过依赖注入为OR(object relational 对象关系)映射工具配置所有支持的功能。
它们可以参与Spring的资源和事务管理，并且遵循Spring的泛型事务和DAO异常层次结构。
推荐的集成风格是针对普通```Hibernate```或者```JPA APIs```编写DAO代码。

在创建数据访问应用程序时，Spring为您选择的ORM层添加了重大增强功能。
您可以根据需要利用尽可能多的集成支持，并且应该将此集成工作与在内部构建类似基础设施的成本和风险进行比较。
无论采用何种技术，您都可以像使用库一样使用大部分ORM支持，因为一切都设计为一组可重用的JavaBean。
Spring IOC容器中的ORM简化了配置和部署。因此，本节中的大多数示例都显示了Spring容器中的配置。

使用Spring框架创建ORM DAO的好处包括：
**测试更简单**：Spring的IoC方法使得交换Hibernate SessionFactory实例、JDBC DataSource实例、事务管理器和映射对象实现(如果需要)的实现和配置位置变得很容易。这反过来又使得隔离测试每段与持久性相关的代码变得容易得多。
**通用的数据访问异常**：Spring可以包装ORM工具中的异常，将它们从专有的(可能已检查的)异常转换为通用的运行时DataAccessException层次结构。这个特性允许您仅在适当的层中处理大多数不可恢复的持久性异常，而无需恼人的样板捕捉、抛出和异常声明。您仍然可以根据需要捕获和处理异常。请记住，JDBC异常(包括特定于DB的方言)也会转换为相同的层次结构，这意味着您可以在一致的编程模型中使用JDBC执行某些操作。
**常规资源管理**：Spring应用程序上下文可以处理Hibernate SessionFactory实例、JPA EntityManagerFactory实例、JDBC DataSource实例和其他相关资源的位置和配置。这使得这些值易于管理和更改。Spring提供了对持久化资源的高效、简单和安全的处理。例如，使用Hibernate的相关代码通常需要使用相同的Hibernate会话来确保效率和正确的事务处理。Spring通过Hibernate SessionFactory公开当前会话，使得创建会话并将其透明地绑定到当前线程变得容易。因此，Spring为任何本地或JTA事务环境解决了许多典型Hibernate使用的长期问题。
**集成事务管理**：您可以通过@Transaction注释或通过在XML配置文件中显式配置事务AOP建议，使用声明性的、面向方面的编程(AOP)风格的方法拦截器包装ORM代码。在这两种情况下，都会为您处理事务语义和异常处理(回滚等)。正如资源和事务管理中所讨论的，您还可以交换各种事务管理器，而不会影响与ORM相关的代码。例如，您可以在本地事务和JTA之间进行交换，在这两种场景中都可以使用相同的完整服务(如声明性事务)。此外，与JDBC相关的代码可以与您用来执行ORM的代码以事务方式完全集成。这对于不适合ORM的数据访问(如批处理和BLOB流)很有用，但仍需要与ORM操作共享公共事务。

## ORM集成注意事项
SpringORM集成的主要目标是清晰的应用程序分层(使用任何数据访问和事务技术)，并实现应用程序对象 - 的松散耦合，不再依赖于数据访问或事务策略的业务服务，不再需要硬编码的资源查找，不再需要难以替换的单例，不再需要自定义服务注册中心。我们的目标是采用一种简单而一致的方法来连接应用程序对象，使它们尽可能地保持可重用性并不依赖于容器。所有单独的数据访问功能都可以单独使用，但它们与Spring的应用程序上下文概念很好地集成在一起，提供了基于XML的配置和对不需要支持Spring的普通JavaBean实例的交叉引用。在典型的Spring应用程序中，许多重要的对象都是JavaBean：数据访问模板、数据访问对象、事务管理器、使用数据访问对象的业务服务和事务管理器、Web视图解析器、使用业务服务的Web控制器等等。

### 资源和事务管理
典型的业务应用程序中充斥着重复的资源管理代码。
许多项目试图发明自己的解决方案，有时会为了编程方便而牺牲对故障的正确处理。
Spring提倡正确处理资源的简单解决方案，即通过JDBC中的模板实现IoC，并为ORM技术应用AOP拦截器。

基础设施提供适当的资源处理和特定API异常到未检查的基础设施异常层次结构的适当转换。
Spring引入了DAO异常层次结构，适用于任何数据访问策略。
对于直接JDBC，上一节中提到的JdbcTemplate类提供连接处理和SQLException到DataAccessException层次结构的正确转换，包括将特定于数据库的SQL错误代码转换为有意义的异常类。
对于ORM技术，请参阅下一节，了解如何获得相同的异常转换好处。

在事务管理方面，JdbcTemplate类与Spring事务支持挂钩，并通过各自的Spring事务管理器同时支持JTA和JDBC事务。
对于受支持的ORM技术，Spring通过Hibernate和JPA事务管理器以及JTA支持来提供Hibernate和JPA支持。

### 异常转换
在DAO中使用Hibernate或JPA时，必须决定如何处理持久性技术的本机异常类。
DAO抛出HibernateException或PersistenceException的子类，具体取决于技术。这些异常都是运行时异常，不需要声明或捕获。
您可能还必须处理IllegalArgumentException和IllegalStateException。
这意味着调用者只能将异常视为通常致命的异常，除非他们希望依赖持久性技术自己的异常结构。
如果不将调用方与实现策略捆绑在一起，则无法捕获特定原因(如乐观锁定失败)。
对于强烈基于ORM的应用程序或不需要任何特殊异常处理(或两者兼而有之)的应用程序，这种权衡可能是可以接受的。
但是，Spring允许通过@Repository注释透明地应用异常转换。

以下示例(一个用于Java配置，一个用于XML配置)显示了如何执行此操作：
```
@Repository
public class ProductDaoImpl implements ProductDao {

    // class body here...

}
```

```
<beans>

    <!-- Exception translation bean post processor -->
    <bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

    <bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

后处理器自动查找所有异常翻译器(PersistenceExceptionTranslator接口的实现)，并通知所有标记有@Repository注释的bean，以便发现的翻译器可以截获并对抛出的异常应用适当的翻译。

总之，您可以基于普通持久性技术的API和注释实现DAO，同时仍然受益于Spring管理的事务、依赖项注入和透明异常转换(如果需要)到Spring的自定义异常层次结构。

## Hibernate
我们从介绍Spring环境中的Hibernate5开始，用它来演示Spring集成OR映射器的方法。
本节详细介绍了许多问题，并展示了DAO实现和事务划分的不同变体。
这些模式中的大多数可以直接转换为所有其他受支持的ORM工具。

### Spring容器中的SessionFactory设置
为了避免将应用程序对象绑定到硬编码的资源查找，您可以在Spring容器中将资源(如JDBC DataSource或Hibernate SessionFactory)定义为bean。
需要访问资源的应用程序对象通过bean引用接收对此类预定义实例的引用。

以下为XML应用程序上下文定义，展示了如何在其上设置JDBC数据源和Hibernate SessionFactory：
```
<beans>
    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="org.hsqldb.jdbcDriver"/>
        <property name="url" value="jdbc:hsqldb:hsql://localhost:9001"/>
        <property name="username" value="sa"/>
        <property name="password" value=""/>
    </bean>

    <bean id="mySessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="myDataSource"/>
        <property name="mappingResources">
            <list>
                <value>product.hbm.xml</value>
            </list>
        </property>
        <property name="hibernateProperties">
            <value>
                hibernate.dialect=org.hibernate.dialect.HSQLDialect
            </value>
        </property>
    </bean>
</beans>
```

从本地Jakarta Commons DBCP BasicDataSource切换到位于JNDI的数据源，如下例所示：
```
<beans>
    <jee:jndi-lookup id="myDataSource" jndi-name="java:comp/env/jdbc/myds"/>
</beans>
```

### 基于HibernateAPI实现DAO
Hibernate有一个称为上下文会话的功能，其中Hibernate自己管理每个事务的一个当前会话。
这大致相当于Spring在每个事务中同步一个Hibernate会话。

基于普通Hibernate API的相应DAO实现类似于以下示例：
```
public class ProductDaoImpl implements ProductDao {

    private SessionFactory sessionFactory;

    public void setSessionFactory(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    public Collection loadProductsByCategory(String category) {
        return this.sessionFactory.getCurrentSession()
                .createQuery("from test.Product product where product.category=?")
                .setParameter(0, category)
                .list();
    }
}
```

除了在实例变量中保存SessionFactory之外，该样式与Hibernate参考文档和示例中的样式类似。
我们强烈推荐这种基于实例的设置，而不是Hibernate的CaveatEmptor示例应用程序中的老式静态HibernateUtil类。
(通常，除非绝对必要，否则不要将任何资源保存在静态变量中。)

前面的DAO示例遵循依赖项注入模式。它非常适合Spring IOC容器，就像根据Spring的HibernateTemplate编码一样。
您也可以在纯Java中设置这样的DAO(例如，在单元测试中)。为此，实例化它并调用setSessionFactory(..)。具有所需的工厂参考。
作为Spring bean定义，DAO如下所示：
```
<beans>
    <bean id="myProductDao" class="product.ProductDaoImpl">
        <property name="sessionFactory" ref="mySessionFactory"/>
    </bean>
</beans>
```

这种DAO风格的主要优点是它只依赖于Hibernate API。不需要导入任何Spring类。
从非侵入性的角度来看，这很有吸引力，对Hibernate开发人员来说可能会感觉更自然。

可以基于普通的Hibernate API实现DAO，同时仍然能够参与Spring管理的事务。

### 声明式事务划分
我们建议您使用Spring的声明性事务支持，它允许您将Java代码中的显式事务划分API调用替换为AOP事务拦截器。
您可以使用Java注释或XML在Spring容器中配置此事务拦截器。
这种声明性事务功能使您可以使业务服务免于重复的事务分界代码，并专注于添加业务逻辑，这才是您的应用程序的真正价值所在。

您可以用@Transaction注释注释服务层，并指示Spring容器查找这些注释并为这些带注释的方法提供事务语义。以下示例显示如何执行此操作：
```
public class ProductServiceImpl implements ProductService {

    private ProductDao productDao;

    public void setProductDao(ProductDao productDao) {
        this.productDao = productDao;
    }

    @Transactional
    public void increasePriceOfAllProductsInCategory(final String category) {
        List productsToChange = this.productDao.loadProductsByCategory(category);
        // ...
    }

    @Transactional(readOnly = true)
    public List<Product> findAllProducts() {
        return this.productDao.findAllProducts();
    }
}
```

在容器中，您需要设置```PlatformTransactionManager```实现(作为bean)和```<tx：Annotation-Driven/>```条目，并在运行时选择```@Transaction```。
以下示例显示如何执行此操作：
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

    <!-- SessionFactory, DataSource, etc. omitted -->

    <bean id="transactionManager"
            class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <tx:annotation-driven/>

    <bean id="myProductService" class="product.SimpleProductService">
        <property name="productDao" ref="myProductDao"/>
    </bean>

</beans>
```

### 编程式事务划分
您可以在跨越任意数量的操作的较低级别的数据访问服务之上，在较高级别的应用程序中划分事务。对周围业务服务的实现也不存在限制。
它只需要一个Spring的```PlatformTransactionManager```。

下面这对代码片段显示了Spring应用程序上下文中的事务管理器和业务服务定义，以及业务方法实现的示例：
```
<beans>
    <bean id="myTxManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="mySessionFactory"/>
    </bean>
    <bean id="myProductService" class="product.ProductServiceImpl">
        <property name="transactionManager" ref="myTxManager"/>
        <property name="productDao" ref="myProductDao"/>
    </bean>
</beans>
```

```
public class ProductServiceImpl implements ProductService {

    private TransactionTemplate transactionTemplate;
    private ProductDao productDao;

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public void setProductDao(ProductDao productDao) {
        this.productDao = productDao;
    }

    public void increasePriceOfAllProductsInCategory(final String category) {
        this.transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            public void doInTransactionWithoutResult(TransactionStatus status) {
                List productsToChange = this.productDao.loadProductsByCategory(category);
                // do the price increase...
            }
        });
    }
}
```
Spring的```TransactionInterceptor```允许使用回调代码抛出任何已检查的应用程序异常，而TransactionTemplate仅限于回调中未检查的异常。
```TransactionTemplate```在发生未检查的应用程序异常或事务仅由应用程序标记为仅回滚时触发回滚(通过设置TransactionStatus)。
默认情况下，TransactionInterceptor的行为方式相同，但允许每个方法使用可配置的回滚策略。

### 事物管理策略
TransactionTemplate和TransactionInterceptor都将实际事务处理委托给PlatformTransactionManager实例或Hibernate应用程序的JtaTransactionManager。
您甚至可以使用自定义的PlatformTransactionManager实现。
从本机Hibernate事务管理切换到JTA只是一个配置问题。
您可以用Spring的JTA事务实现替换Hibernate事务管理器。
事务界定和数据访问代码都无需更改即可工作，因为它们使用通用事务管理API。


### 容器管理的资源和本地定义的资源比较
略
### Hibernate发出的虚假应用程序服务器警告
略

## !!!JPA
Spring JPA可在```org.springframework work.orm.jpa```包下获得。
它以类似于与Hibernate集成的方式提供对Java持久性API的全面支持，同时了解底层实现以提供附加功能。

### JPA的三个设置选项
Spring JPA支持提供了三种设置JPA EntityManagerFactory的方法，应用程序使用它来获取实体管理器：
+ 使用LocalEntityManagerFactoryBean
+ 从JNDI获取EntityManagerFactory
+ 使用LocalContainerEntityManagerFactoryBean

#### 使用LocalEntityManagerFactoryBean
您只能在简单的部署环境(如独立应用程序和集成测试)中使用此选项。

LocalEntityManagerFactoryBean创建一个EntityManagerFactory，适用于应用程序仅使用JPA进行数据访问的简单部署环境。
工厂Bean使用JPA PersistenceProvider自动检测机制(根据JPA的Java SE引导)，并且在大多数情况下，只需要指定持久性单元名称。

下面的XML示例配置这样的Bean：
```
<beans>
    <bean id="myEmf" class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean">
        <property name="persistenceUnitName" value="myPersistenceUnit"/>
    </bean>
</beans>
```
这种形式的JPA部署是最简单和最有限的。
您不能引用现有的JDBC DataSource bean定义，并且不存在对全局事务的支持。
此外，持久类的编织(字节码转换)是特定于提供程序的，通常需要在启动时指定特定的JVM代理。
此选项仅适用于独立应用程序和测试环境，而JPA规范正是针对这些环境而设计的。

#### 从JNDI获取EntityManagerFactory
在部署到Java EE服务器时，可以使用此选项。
查看服务器文档，了解如何将自定义JPA提供程序部署到服务器中，从而允许使用不同于服务器默认提供程序的提供程序。

从JNDI(例如，在Java EE环境中)获取EntityManagerFactory需要更改XML配置，如下面的示例所示：
```		
<beans>
    <jee:jndi-lookup id="myEmf" jndi-name="persistence/myPersistenceUnit"/>
</beans>
```

#### 使用LocalContainerEntityManagerFactoryBean
您可以在基于Spring的应用程序环境中使用此选项来实现完整的JPA功能。
这包括Tomcat等Web容器、独立应用程序和具有复杂持久性要求的集成测试。

LocalContainerEntityManagerFactoryBean完全控制EntityManagerFactory配置，适用于需要细粒度定制的环境。
LocalContainerEntityManagerFactoryBean基于sistence.xml文件、提供的dataSourceLookup策略和指定的loadTimeWeaver创建PersistenceUnitInfo实例。因此，可以使用JNDI之外的自定义数据源并控制编织过程。
以下示例显示了LocalContainerEntityManagerFactoryBean的典型Bean定义：
```
<beans>
    <bean id="myEmf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="someDataSource"/>
        <property name="loadTimeWeaver">
            <bean class="org.springframework.instrument.classloading.InstrumentationLoadTimeWeaver"/>
        </property>
    </bean>
</beans>
```

下面的示例显示了一个的persistence.xml文件：
```
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
    <persistence-unit name="myUnit" transaction-type="RESOURCE_LOCAL">
        <mapping-file>META-INF/orm.xml</mapping-file>
        <exclude-unlisted-classes/>
    </persistence-unit>
</persistence>
```

### 基于JPA实现DAO
通过使用注入的```EntityManagerFactory```或```EntityManager```，可以在没有任何Spring依赖的情况下针对普通JPA编写代码。
如果启用了```PersistenceAnnotationBeanPostProcessor```，Spring可以在字段和方法级别使用```@PersistenceUnit```和```@PersistenceContext```注释。

下面的示例显示了一个使用@PersistenceUnit注释的普通JPA DAO实现：
```
public class ProductDaoImpl implements ProductDao {

    private EntityManagerFactory emf;

    @PersistenceUnit
    public void setEntityManagerFactory(EntityManagerFactory emf) {
        this.emf = emf;
    }

    public Collection loadProductsByCategory(String category) {
        try (EntityManager em = this.emf.createEntityManager()) {
            Query query = em.createQuery("from Product as p where p.category = ?1");
            query.setParameter(1, category);
            return query.getResultList();
        }
    }
}
```

前面的DAO不依赖于Spring，仍然非常适合Spring应用程序上下文。
此外，DAO利用注释要求注入默认的EntityManagerFactory，如下面的示例Bean定义所示：
```
<beans>
    <!-- bean post-processor for JPA annotations -->
    <bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>
    <bean id="myProductDao" class="product.ProductDaoImpl"/>
</beans>
```

作为显式定义PersistenceAnnotationBeanPostProcessor的替代方法，可以考虑在应用程序上下文配置中使用Spring ```Context：Annotation-config```XML元素。这样做会自动为基于注释的配置注册所有Spring标准后处理器，包括CommonAnnotationBeanPostProcessor等。
考虑以下示例：
```
<beans>
    <!-- post-processors for all standard config annotations -->
    <context:annotation-config/>
    <bean id="myProductDao" class="product.ProductDaoImpl"/>
</beans>
```
这种DAO的主要问题是它总是通过工厂创建一个新的EntityManager。您可以通过请求注入事务性EntityManager而不是工厂来避免这种情况。
以下示例显示如何执行此操作：
```
public class ProductDaoImpl implements ProductDao {
	
    @PersistenceContext
    private EntityManager em;

    public Collection loadProductsByCategory(String category) {
        Query query = em.createQuery("from Product as p where p.category = :category");
        query.setParameter("category", category);
        return query.getResultList();
    }
}
```


### Spring驱动的JPA事务
推荐的JPA策略是通过JPA的本地事务支持进行本地事务。Spring的JpaTransactionManager针对任何常规JDBC连接池(不需要XA)提供了许多从本地JDBC事务中得知的功能(例如特定于事务的隔离级别和资源级只读优化)。

Spring JPA还允许已配置的JpaTransactionManager向访问相同数据源的JDBC访问代码公开JPA事务，前提是注册的JpaDialect支持底层JDBC连接的检索。Spring为EclipseLink和Hibernate JPA实现提供了方言。

### 了解JpaDialect和JpaVendorAdapter
作为一项高级功能，JpaTransactionManager和AbstractEntityManagerFactoryBean的子类允许将自定义JpaDialect传递给jpaDialect bean属性。
JpaDialect实现通常以特定于供应商的方式启用Spring支持的以下高级功能：
+ 应用特定的事务语义(如自定义隔离级别或事务超时)。
+ 检索事务性JDBC连接(用于暴露基于JDBC的DAO)。
+ PersistenceExceptions到Spring DataAccessException的高级翻译。

这对于特殊的事务语义和异常的高级翻译特别有价值。默认实现(DefaultJpaDialect)不提供任何特殊功能，如果需要前面列出的功能，则必须指定适当的方言。

### 使用JTA事务管理设置JPA
作为替代方案JpaTransactionManager，Spring还允许通过JTA在Java EE环境中或与独立的事务协调器（例如Atomikos）一起进行多资源事务协调。
除了选择SpringJtaTransactionManager而不是之外 JpaTransactionManager，您还需要采取其他步骤：
+ 底层的JDBC连接池必须具有XA功能，并且必须与事务协调器集成在一起。这在Java EE环境中通常很简单，DataSource通过JNDI公开了另一种类型。有关详细信息，请参见您的应用程序服务器文档。类似地，独立的事务协调器通常带有特殊的XA集成DataSource变体。再次，检查其文档。
+ EntityManagerFactory需要为JTA配置JPA设置。这是特定于提供程序的，通常通过将特殊属性指定为jpaProperties on LocalContainerEntityManagerFactoryBean。对于Hibernate，这些属性甚至是特定于版本的。有关详细信息，请参见Hibernate文档。
+ SpringHibernateJpaVendorAdapter强制实施某些面向Spring的默认值，例如连接释放模式on-close，它与Hibernate 5.0中的Hibernate自己的默认值匹配，但在Hibernate 5.1+中不再匹配。对于JTA设置，请确保将持久性单元事务类型声明为“ JTA”。或者，将Hibernate 5.2的 hibernate.connection.handling_mode属性设置 DELAYED_ACQUISITION_AND_RELEASE_AFTER_STATEMENT为恢复Hibernate自己的默认值。有关相关说明，请参见带有Hibernate的Spurious Application Server警告。
+ 或者，考虑EntityManagerFactory从您的应用程序服务器本身（通过JNDI查找而不是在本地声明 LocalContainerEntityManagerFactoryBean）获得。服务器提供的服务器EntityManagerFactory 可能需要在服务器配置中进行特殊定义（使部署的可移植性降低），但已针对服务器的JTA环境进行了设置。

### 用于JPA交互的本机Hibernate设置和本机Hibernate事务
LocalSessionFactoryBean与结合使用的本机设置HibernateTransactionManager 允许@PersistenceContext与其他JPA访问代码进行交互。Hibernate 现在SessionFactory本机实现JPA的EntityManagerFactory接口，而HibernateSession处理本机就是JPA EntityManager。Spring的JPA支持工具会自动检测本地Hibernate会话。

因此，在许多情况下，这种本地Hibernate设置可以替代标准JPA LocalContainerEntityManagerFactoryBean和JpaTransactionManager组合，从而允许与同一本地事务内的SessionFactory.getCurrentSession() （也可以与之交互HibernateTemplate）@PersistenceContext EntityManager。这样的设置还提供了更强大的Hibernate集成和更大的配置灵活性，因为它不受JPA引导合同的约束。

HibernateJpaVendorAdapter在这种情况下，您不需要配置，因为Spring的本机Hibernate安装程序提供了更多功能（例如，自定义Hibernate Integrator安装程序，Hibernate 5.3 Bean容器集成以及对只读事务的更强优化）。最后但并非最不重要的一点是，您还可以通过LocalSessionFactoryBuilder，与@Bean样式配置无缝集成（不FactoryBean涉及）来表示本地Hibernate设置。

# Marshalling XML by Using Object-XML Mappers 使用对象-XML映射器编组XML
略

# Appendix 附录
## XML协议
### tx协议
这些tx标记用于在Spring的事务综合支持中配置所有这些bean。这些标签在标题为“事务管理”的章节中介绍 。

为了完整性，要在tx架构中使用元素，您需要在Spring XML配置文件的顶部具有以下序言。
以下代码段中的文本引用了正确的架构，以便tx您可以使用名称空间中的标记：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" 
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx https://www.springframework.org/schema/tx/spring-tx.xsd 
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```

### jdbc协议
这些jdbc元素使您可以快速配置嵌入式数据库或初始化现有数据源。

要使用jdbc架构中的元素，您需要在Spring XML配置文件的顶部具有以下序言。
以下代码段中的文本引用了正确的架构，以便jdbc您可以使用名称空间中的元素：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc" 
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/jdbc https://www.springframework.org/schema/jdbc/spring-jdbc.xsd"> 

    <!-- bean definitions here -->

</beans>
```