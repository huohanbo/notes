# 参考资料
> [Spring参考文档](https://docs.spring.io/spring-framework/docs/current/reference/html/)
> [Testing](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing)

--------------------------------------------------
本章介绍了Spring对集成测试的支持以及单元测试的最佳实践。
Spring团队提倡测试驱动开发（test-driven development，TDD）。
Spring团队发现正确使用控制反转（inversion of control，IoC）确实使单元测试和集成测试更加容易。

# Spring测试简介
测试是企业软件开发不可或缺的一部分。
本章重点介绍IoC原则为```单元测试```带来的价值，以及Spring框架对```集成测试```的支持所带来的好处。

# 单元测试
与传统的Java EE开发相比，依赖注入应该使您的代码对容器的依赖程度降低。
组成应用程序的POJO应该可以在JUnit或TestNG测试中进行测试，并且对象可以使用new 运算符实例化，而无需使用Spring或任何其他容器。
可以使用模拟对象来单独测试代码。
如果您遵循Spring的体系结构建议，那么代码库的最终分层和组件化将使单元测试更加容易。
例如，您可以通过存根或模拟DAO或存储库接口来测试服务层对象，而无需在运行单元测试时访问持久性数据。

真正的单元测试通常运行非常快，因为没有可设置的运行时基础架构。
将真正的单元测试作为开发方法的一部分可以提高生产率。
您可能不需要测试章节的这一部分来帮助您为基于IoC的应用程序编写有效的单元测试。
但是，对于某些单元测试方案，Spring框架提供了模拟对象和测试支持类，本章对此进行了介绍。

## Mock Objects 模拟对象
Spring包含许多专门用于mocking的软件包：
+ Environment
+ JNDI
+ Servlet API
+ Spring Web Reactive

### Environment
包```org.springframework.mock.env```包含```Environment```和```PropertySource```抽象的模拟实现。
```MockEnvironment```和```MockPropertySource```对于开发依赖于环境特定属性的代码的容器外测试非常有用。

### JNDI
包```org.springframework.mock.jndi```包含JNDI的SPI的部分实现，您可以使用它为测试套件或独立应用程序设置简单的JNDI环境。
例如，如果JDBC DataSource实例在测试代码中绑定到与Java EE容器中相同的JNDI名称，则无需修改即可在测试场景中重用应用程序代码和配置。

### Servlet API
包```org.springframework.mock.web```包含一组全面的servlet API模拟对象，这些对象对测试Web上下文、控制器和过滤器很有用。
这些模拟对象是针对Spring的Web MVC框架使用的，通常比动态模拟对象(如EasyMock)或替代的servlet API模拟对象(如MockObjects)更易于使用。

### Spring Web Reactive
包```org.springframework.mock.http.server.reactive```包含WebFlux应用程序中使用的ServerHttpRequest和ServerHttpResponse的模拟实现。
包```org.springframework.mock.web.server```包含一个依赖于这些模拟请求和响应对象的模拟ServerWebExchange。

```MockServerHttpRequest```和```MockServerHttpResponse```都扩展自与特定于服务器的实现相同的抽象基类，并与它们共享行为。
例如，模拟请求一旦创建就是不可变的，但是您可以使用ServerHttpRequest中的matate()方法来创建修改后的实例。

为了让模拟响应正确实现写约定并返回一个写完成句柄，它默认使用一个带有```cache().Then()```的Flux，该函数缓冲数据并使其可用于测试中的断言。
应用程序可以设置自定义写入函数()。

```WebTestClient```构建在模拟请求和响应之上，以支持在没有HTTP服务器的情况下测试WebFlux应用程序。
该客户端还可以用于与正在运行的服务器进行端到端测试。

## Unit Testing Support Classes 单元测试工具类
Spring包含许多可以帮助进行单元测试的类。它们分为两类：
+ 通用测试工具类
+ Spring MVC测试工具类

### 通用测试工具类
软件包org.springframework.test.util包含几个用于单元和集成测试的通用实用程序。

ReflectionTestUtils是基于反射的实用程序方法的集合。
您可以使用测试，你需要改变一个常量的值情况下这些方法，建立非public现场，调用非publicsetter方法，
或者调用非public 配置或生命周期回调方法测试使用的情况下，这样的应用程序代码时，如下：
+ 纵容private或protected现场访问的ORM框架（例如JPA和Hibernate），而不是public域实体中属性的设置方法。
+ 弹簧的注解（如支持@Autowired，@Inject和@Resource），即用于提供依赖注入private或protected字段，setter方法和配置方法。
+ 注释的使用，例如@PostConstruct和@PreDestroy用于生命周期回调方法。

AopTestUtils是与AOP相关的实用程序方法的集合。您可以使用这些方法来获取对隐藏在一个或多个Spring代理后面的基础目标对象的引用。
例如，如果您已通过使用诸如EasyMock或Mockito之类的库将bean配置为动态模拟，并且该模拟包装在Spring代理中，则可能需要直接访问基础模拟以配置对它的期望并执行验证。

### Spring MVC测试工具类
软件包```org.springframework.test.web```包含 ModelAndViewAssert。
您可以将其与JUnit，TestNG或任何其他测试框架结合使用以进行处理Spring MVCModelAndView对象的单元测试。

# 集成测试
## 总览
能够执行一些集成测试而无需部署到应用程序服务器或连接到其他企业基础结构，这一点很重要。这样做可以测试诸如：
+ Spring IoC容器上下文的正确接线。
+ 使用JDBC或ORM工具进行数据访问。这可以包括诸如SQL语句的正确性，Hibernate查询，JPA实体映射之类的东西。

Spring框架为```spring-test```模块中的集成测试提供了一流的支持。
实际的JAR文件的名称可能包括发行版，也可能是长```org.springframework.test```格式，具体取决于从何处获取。
该库包括该```org.springframework.test```软件包，其中包含用于与Spring容器进行集成测试的有价值的类。
此测试不依赖于应用程序服务器或其他部署环境。
此类测试的运行速度比单元测试慢，但比依赖于部署到应用程序服务器的等效Selenium测试或远程测试快。

单元和集成测试支持以注释驱动的Spring TestContext Framework的形式提供。
TestContext框架与使用的实际测试框架无关，该框架允许在各种环境中对测试进行检测。

## 集成测试的目标
Spring的集成测试支持具有以下主要目标：
+ 在测试之间管理Spring IoC容器缓存。
+ 提供测试夹具实例的依赖注入。
+ 提供适合集成测试的事务管理。
+ 提供特定于Spring的基类，以帮助开发人员编写集成测试。

### 上下文管理和缓存
Spring TestContext Framework提供了对SpringApplicationContext实例和WebApplicationContext实例的一致加载以及对这些上下文的缓存。
对加载的上下文的缓存的支持很重要，因为启动时间可能会成为一个问题-不是因为Spring本身的开销，而是因为Spring容器实例化的对象需要时间才能实例化。
例如，具有50到100个Hibernate映射文件的项目可能需要10到20秒来加载映射文件，并且在每个测试夹具中运行每个测试之前要承担该费用，这会导致整体测试运行速度变慢，从而降低了开发人员的工作效率。

测试类通常声明XML或Groovy配置元数据的资源位置数组（通常在类路径中），或声明用于配置应用程序的组件类数组。
这些位置或类与web.xml生产部署中或其他配置文件中指定的位置或类相同或相似。

默认情况下，配置文件一旦加载，ApplicationContext便会重新用于每个测试。
因此，每个测试套件仅产生一次安装成本，并且随后的测试执行要快得多。
在这种情况下，术语“测试套件”是指所有测试均在同一JVM中运行-例如，所有测试均从给定项目或模块的Ant，Maven或Gradle构建运行。
在不太可能的情况下，测试破坏了应用程序上下文并需要重新加载（例如，通过修改Bean定义或应用程序对象的状态），可以将TestContext框架配置为重新加载配置并重建应用程序上下文，然后再执行下一个测试。

### 测试夹具的依赖注入
当TestContext框架加载您的应用程序上下文时，可以选择使用依赖注入来配置测试类的实例。
这提供了一种方便的机制，可以通过在应用程序上下文中使用预配置的bean来设置测试装置。
这里的一个强大好处是，您可以在各种测试场景中重用应用程序上下文，从而避免了为单个测试用例重复复杂的测试夹具设置的需要。

例如，考虑一个场景，其中我们有一个类（HibernateTitleRepository），该类实现了Title域实体的数据访问逻辑。
我们要编写集成测试来测试以下方面：
+ Spring配置：基本上，与HibernateTitleRepositoryBean配置相关的一切都 正确且存在吗？
+ Hibernate映射文件配置：是否正确映射了所有内容，并且是否有正确的延迟加载设置？
+ HibernateTitleRepository的逻辑：此类的配置实例是否按预期执行？

### 事物管理
访问真实数据库的测试中的一个常见问题是它们对持久性存储状态的影响。
即使使用开发数据库，​​对状态的更改也可能会影响以后的测试。
同样，许多操作（例如插入或修改持久数据）无法在事务之外执行（或验证）。

TestContext框架解决了这个问题。默认情况下，框架为每个测试创建并回滚事务。
您可以编写可以假定存在事务的代码。如果在测试中调用事务代理对象，则对象将根据其配置的事务语义正确运行。
此外，如果测试方法在为测试管理的事务中运行时删除选定表的内容，则该事务将默认回滚，并且数据库将返回到执行测试之前的状态。
通过使用PlatformTransactionManager在测试的应用程序上下文中定义的bean，可以为测试提供事务支持。

### 集成测试工具类
Spring TestContext框架提供了几个abstract支持类，这些类简化了集成测试的编写。
这些基础测试类为测试框架提供了定义明确的钩子，以及方便的实例变量和方法，可用于访问以下内容：
+ ApplicationContext，用于执行bean的查找或者测试背景下作为一个整体的状态。
+ JdbcTemplate，用于执行SQL语句以查询数据库。您可以在执行与数据库相关的应用程序代码之前和之后使用此类查询来确认数据库状态，并且Spring确保此类查询在与应用程序代码相同的事务范围内运行。与ORM工具一起使用时，请确保避免误报。

## JDBC测试支持
软件包```org.springframework.test.jdbc```包含JdbcTestUtils，它是JDBC相关实用程序功能的集合，旨在简化标准数据库测试方案。
具体来说，JdbcTestUtils提供以下静态实用程序方法。
+ countRowsInTable(..)：计算给定表中的行数。
+ countRowsInTableWhere(..)：使用提供的WHERE子句计算给定表中的行数。
+ deleteFromTables(..)：删除指定表中的所有行。
+ deleteFromTableWhere(..)：使用提供的WHERE子句从给定表中删除行 。
+ dropTables(..)：删除指定的表。

## !!!注解
### !!!Spring测试注解
Spring框架提供了以下特定于Spring的注释集，可以在单元测试和集成测试中将它们与TestContext框架结合使用。

#### @BootstrapWith
```@BootstrapWith```是一个类级别的注释，可用于配置如何引导Spring TestContext Framework。
具体来说，您可以用```@BootstrapWith```来指定一个custom TestContextBootstrapper。

#### @ContextConfiguration
```@ContextConfiguration```定义用于确定如何加载和配置ApplicationContext集成测试的类级元数据。
具体来说， ```@ContextConfiguration```声明应用程序上下文资源```locations```或```classes```用于加载上下文的组件。

资源位置通常是位于类路径中的XML配置文件或Groovy脚本，而组件类通常是@Configuration类。
但是，资源位置也可以引用文件系统中的文件和脚本，而组件类可以是@Component类，@Service类等。

以下示例显示了一个@ContextConfiguration引用XML文件的注释：
```
@ContextConfiguration("/test-config.xml") 
class XmlApplicationContextTests {
    // class body...
}
```

以下示例显示了一个@ContextConfiguration引用类的注释：
```
@ContextConfiguration(classes = TestConfig.class) 
class ConfigClassApplicationContextTests {
    // class body...
}
```

作为声明资源位置或组件类的替代方法或补充，您可以使用@ContextConfiguration声明ApplicationContextInitializer类。
以下示例显示了这种情况：
```
@ContextConfiguration(initializers = CustomContextIntializer.class) 
class ContextInitializerTests {
    // class body...
}
```

您也可以选择使用@ContextConfiguration来声明该ContextLoader策略。
但是请注意，由于默认加载器支持initializers资源locations或component ，因此通常不需要显式配置加载器classes。
以下示例同时使用位置和装载程序：
```
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class) 
class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

#### @WebAppConfiguration
@WebAppConfiguration是类级别的注释，用来声明集成测试加载的ApplicationContext为应该是WebApplicationContext。
测试类上仅使用@WebAppConfiguration就可以确保为测试加载WebApplicationContext，并使用“file：src/main/webapp”作为指向Web应用程序根的路径(即资源库路径)的默认值。
资源基路径在后台用于创建MockServletContext，它充当测试的WebApplicationContext的ServletContext。

以下示例显示了如何使用@WebAppConfiguration注释：
```
@ContextConfiguration
@WebAppConfiguration 
class WebAppTests {
    // class body...
}
```

要覆盖默认值，可以使用隐式value属性指定其他基础资源路径。
无论```classpath:```和```file:```资源前缀的支持。如果未提供资源前缀，则假定该路径是文件系统资源。
以下示例显示如何指定类路径资源：
```
@ContextConfiguration
@WebAppConfiguration("classpath:test-web-resources") 
class WebAppTests {
    // class body...
}
```

请注意，```@WebAppConfiguration```必须与```@ContextConfiguration```结合使用，无论是在单个测试类中还是在测试类层次结构中。

#### @ContextHierarchy
```@ContextHierarchy```是类级别的注释，用于定义ApplicationContext集成测试的实例层次结构。
@ContextHierarchy应该用一个或多个@ContextConfiguration实例的列表声明，每个实例定义上下文层次结构中的一个级别。

以下示例演示了@ContextHierarchy在单个测试类中的用法（@ContextHierarchy也可以在测试类层次结构中使用）：
```
@ContextHierarchy({
    @ContextConfiguration("/parent-config.xml"),
    @ContextConfiguration("/child-config.xml")
})
class ContextHierarchyTests {
    // class body...
}
```
```
@WebAppConfiguration
@ContextHierarchy({
    @ContextConfiguration(classes = AppConfig.class),
    @ContextConfiguration(classes = WebConfig.class)
})
class WebIntegrationTests {
    // class body...
}
```
如果您需要合并或覆盖测试类层次结构中上下文层次结构的给定级别的配置，则必须通过向类层次结构中每个相应级别的@ContextConfiguration中的Name属性提供相同的值来显式命名该级别。

#### @ActiveProfiles
```@ActiveProfiles```是一个类级别的注释，用于声明在ApplicationContext为集成测试加载时应激活哪些bean定义配置文件。

以下示例表明该```dev```配置处于活动状态：
```
@ContextConfiguration
@ActiveProfiles("dev") 
class DeveloperTests {
    // class body...
}
```

以下示例表明```dev```和```integration```处于活动状态：
```
@ContextConfiguration
@ActiveProfiles({"dev", "integration"}) 
class DeveloperIntegrationTests {
    // class body...
}
```

#### @TestPropertySource
```@TestPropertySource```是一个类级批注，您可以使用它来配置属性文件和内联属性的位置，以便为集成测试加载的ApplicationContext将属性文件和内联属性添加到Environment中的PropertySources集。

下面的示例演示如何从类路径声明属性文件：
```
@ContextConfiguration
@TestPropertySource("/test.properties") 
class MyIntegrationTests {
    // class body...
}
```

下面的示例演示如何声明内联属性：
```
@ContextConfiguration
@TestPropertySource(properties = { "timezone = GMT", "port: 4242" }) 
class MyIntegrationTests {
    // class body...
}
```

#### @DynamicPropertySource
```@DynamicPropertySource```是一个方法级批注，可用于为集成测试加载的ApplicationContext注册要添加到Environment中PropertySources集合的动态属性。
当您事先不知道属性的值时(例如，如果属性由外部资源管理，如TestContainers项目管理的容器的属性)，则动态属性非常有用。

下面的示例演示如何注册动态属性：
```
@ContextConfiguration
class MyIntegrationTests {

    static MyExternalServer server = // ...

    @DynamicPropertySource 
    static void dynamicProperties(DynamicPropertyRegistry registry) { 
        registry.add("server.port", server::getPort); 
    }

    // tests ...
}
```

#### @DirtiesContext
```@DirtiesContext```表示底层Spring ApplicationContext在测试执行期间已被污染，应该将其关闭。
当应用程序上下文被标记为脏时，它将从测试框架的缓存中移除并关闭。
因此，对于需要具有相同配置元数据的上下文的任何后续测试，都会重新构建底层Spring容器。

您可以在同一个类或类层次结构中将@DirtiesContext用作类级批注和方法级批注。
在这种情况下，ApplicationContext在任何此类带注释的方法之前或之后以及当前测试类之前或之后标记为脏，具体取决于配置的Method Mode和classMode。

以下示例说明了在各种配置方案下何时弄脏上下文：

在当前测试类之前，在类模式设置为的类上声明时 BEFORE_CLASS。
```
@DirtiesContext(classMode = BEFORE_CLASS) 
class FreshContextTests {
    // 一些测试需要新的Spring容器
}
```

在当前测试类之后，当在类模式设置为 AFTER_CLASS（即默认类模式）的类上声明时。
```
@DirtiesContext 
class ContextDirtyingTests {
    // 一些导致Spring容器被弄脏的测试
}
```

在当前测试类中的每个测试方法之前，在类模式设置为的类上声明时 BEFORE_EACH_TEST_METHOD。
```
@DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD) 
class FreshContextTests {
    // 一些测试需要新的Spring容器
}
```

在当前测试类中的每个测试方法之后，在类模式设置为的类上声明时 AFTER_EACH_TEST_METHOD。
```
@DirtiesContext(classMode = AFTER_EACH_TEST_METHOD) 
class ContextDirtyingTests {
    // 一些导致Spring容器被弄脏的测试
}
```

在当前测试之前，在方法模式设置为的方法上声明时 BEFORE_METHOD。
```
@DirtiesContext(methodMode = BEFORE_METHOD) 
@Test
void testProcessWhichRequiresFreshAppCtx() {
    // 需要新Spring容器的一些逻辑
}
```

在当前测试之后，当在方法模式设置为 AFTER_METHOD（即默认方法模式）的方法上声明时。
```
@DirtiesContext 
@Test
void testProcessWhichDirtiesAppCtx() {
    // 导致Spring容器被弄脏的一些逻辑
}
```

如果在使用@DirtiesContext将上下文配置为上下文层次结构一部分的测试中使用@ContextHierarchy，则可以使用该hierarchyMode标志来控制如何清除上下文缓存。
默认情况下，使用穷举算法清除上下文缓存，不仅包括当前级别，还包括共享当前测试共有的祖先上下文的所有其他上下文层次结构。
ApplicationContext驻留在公共祖先上下文的子层次结构中的所有实例都将从上下文缓存中删除并关闭。
如果穷举算法对于特定用例而言过于矫kill过正，则可以指定更简单的当前级别算法，如以下示例所示。
```
@ContextHierarchy({
    @ContextConfiguration("/parent-config.xml"),
    @ContextConfiguration("/child-config.xml")
})
class BaseTests {
    // class body...
}

class ExtendedTests extends BaseTests {

    @Test
    @DirtiesContext(hierarchyMode = CURRENT_LEVEL) 
    void test() {
        // some logic that results in the child context being dirtied
    }
}
```

#### @TestExecutionListeners
```@TestExecutionListeners```定义了用于配置TestExecutionListener应向进行注册的实现的类级元数据 TestContextManager。
通常，@TestExecutionListeners与结合使用 @ContextConfiguration。

下面的示例演示如何注册两个TestExecutionListener实现：
```
@ContextConfiguration
@TestExecutionListeners({CustomTestExecutionListener.class, AnotherTestExecutionListener.class}) 
class CustomTestExecutionListenerTests {
    // class body...
}
```

#### @RecordApplicationEvents
@RecordApplicationEvents是一个类级别的注释，用于指示 Spring TestContext Framework记录ApplicationContext在单个测试的执行期间发布的所有应用程序事件 。

可以ApplicationEvents在测试中通过API访问记录的事件。

#### @Commit
@Commit表示应该在测试方法完成后提交用于事务测试方法的事务。
您可以用@Commit直接替换@Rollback(false)以更明确地传达代码的意图。
与相似@Rollback，@Commit也可以声明为类级别或方法级别的注释。

以下示例显示了如何使用@Commit注释：
```
@Commit 
@Test
void testProcessWithoutRollback() {
    // ...
}
```

#### @Rollback
@Rollback指示是否应在测试方法完成后回退事务测试方法的事务。
如果为true，则事务将回滚。否则，将提交事务（另请参见 @Commit）。
true即使@Rollback未明确声明，Spring TestContext Framework中集成测试的回滚默认为。

当声明为类级注释时，@Rollback为测试类层次结构内的所有测试方法定义默认回滚语义。
当声明为方法级注释时，@Rollback为特定的测试方法定义回滚语义，可能覆盖类级@Rollback或@Commit语义。

以下示例使测试方法的结果不回滚（即，结果已提交到数据库）：
```
@Rollback(false) 
@Test
void testProcessWithoutRollback() {
    // ...
}
```

#### @BeforeTransaction
@BeforeTransaction表示void对于已配置为通过使用Spring@Transactional注释在事务内运行的测试方法，应在开始事务之前运行带注释的方法。
@BeforeTransaction方法不是必需的，public并且可以在基于Java 8的接口默认方法中声明。

以下示例显示了如何使用@BeforeTransaction注释：
```
@BeforeTransaction 
void beforeTransaction() {
    // logic to be run before a transaction is started
}
```

#### @AfterTransaction
@AfterTransaction表示void对于已配置为使用Spring@Transactional注释在事务内运行的测试方法，在事务结束后应运行带注释的方法。
@AfterTransaction方法不是必需的，public并且可以在基于Java 8的接口默认方法中声明。
```
@AfterTransaction 
void afterTransaction() {
    // logic to be run after a transaction has ended
}
```

#### @Sql
@Sql用于注释测试类或测试方法，以配置在集成测试期间针对给定数据库运行的SQL脚本。

以下示例显示了如何使用它：
```
@Test
@Sql({"/test-schema.sql", "/test-user-data.sql"}) 
void userTest() {
    // run code that relies on the test schema and test data
}
```

#### @SqlConfig
@SqlConfig定义用于确定如何解析和运行使用@Sql注释配置的SQL脚本的元数据。

以下示例显示了如何使用它：
```
@Test
@Sql(
    scripts = "/test-user-data.sql",
    config = @SqlConfig(commentPrefix = "`", separator = "@@") 
)
void userTest() {
    // run code that relies on the test data
}
```

#### @SqlMergeMode
@SqlMergeMode用于注释测试类或测试方法，以配置是否将方法级@Sql声明与类级@Sql声明合并。
如果@SqlMergeMode未在测试类或测试方法上声明，则OVERRIDE默认使用合并模式。
使用该OVERRIDE模式，方法级@Sql声明将有效地覆盖类级@Sql声明。

请注意，方法级别的@SqlMergeMode声明将覆盖类级别的声明。

以下示例显示了如何@SqlMergeMode在类级别使用。
```
@SpringJUnitConfig(TestConfig.class)
@Sql("/test-schema.sql")
@SqlMergeMode(MERGE) 
class UserTests {

    @Test
    @Sql("/user-test-data-001.sql")
    void standardUserProfile() {
        // run code that relies on test data set 001
    }
}
```

以下示例显示了如何@SqlMergeMode在方法级别使用。
```
@SpringJUnitConfig(TestConfig.class)
@Sql("/test-schema.sql")
class UserTests {

    @Test
    @Sql("/user-test-data-001.sql")
    @SqlMergeMode(MERGE) 
    void standardUserProfile() {
        // run code that relies on test data set 001
    }
}
```

#### @SqlGroup
@SqlGroup是一个聚合多个@Sql注释的容器注释。您可以原@SqlGroup生地声明多个嵌套的@Sql注释，也可以将其与Java 8对可重复注释的支持结合使用，在@Sql同一类或方法上可以重复声明多次，隐式生成此容器注释。

下面的示例显示如何声明一个SQL组：
```
@Test
@SqlGroup({ 
    @Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`")),
    @Sql("/test-user-data.sql")
)}
void userTest() {
    // run code that uses the test schema and test data
}
```

### Spring标准注解支持
### Spring JUnit 4 测试注解
### Spring JUnit Jupiter 测试注解
### 测试的元注解支持

## Spring TestContext
## WebTestClient
## MockMvc
## Testing Client Applications

# 其他资源