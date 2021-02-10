# 参考资料
> [Spring参考文档](https://docs.spring.io/spring-framework/docs/current/reference/html/)
> [Core Technologies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)

--------------------------------------------------
这一部分涵盖了Spring框架必不可少的所有技术，其中最重要的是控制反转（IoC）容器 和 面向方面的编程（AOP）技术。
Spring还提供了与AspectJ的集成。

# IoC容器
IOC（Inversion of Control）：控制反转

## 容器简介
IoC也称为依赖注入DI（Dependency Injection）。

在此过程中，对象仅通过构造函数参数，工厂方法的参数或在构造或从工厂方法返回后在对象实例上设置的属性来定义其依赖项（即，与它们一起使用的其他对象）。然后，容器在创建bean时注入那些依赖项。
此过程从根本上讲是通过使用类的直接构造或诸如服务定位器模式之类的控件来控制其依赖项的实例化或位置的bean本身的逆过程（因此称为Control的倒置）。

包**org.springframework.beans**和**org.springframework.context**是Spring框架的IoC容器的基础。

BeanFactory接口 提供了一种高级配置机制，能够管理任何类型的对象。 ApplicationContext 是 BeanFactory的子接口。它增加了：
+ 与Spring的AOP功能轻松集成
+ 消息资源处理（用于国际化）
+ 活动发布
+ 应用层特定的上下文，例如WebApplicationContext 用于Web应用程序中的

简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext增加了更多针对企业的功能。

## Bean简介
在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。

Bean是由Spring IoC容器实例化，组装和管理的对象。否则，bean仅仅是应用程序中许多对象之一。

Bean及其之间的依赖关系反映在容器使用的配置元数据中。

## !!!容器概述
org.springframework.context.ApplicationContext接口 代表IoC容器，并负责实例化，配置和组装Bean。
容器通过读取配置元数据获取有关要实例化，配置和组装哪些对象的指令。
配置元数据可以使用XML，Java注解或Java代码，从而表示组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

Spring提供了 ApplicationContext接口 的几种实现。
在独立应用程序中，通常创建ClassPathXmlApplicationContext 或的实例 FileSystemXmlApplicationContext。
尽管XML是定义配置元数据的传统格式，但是可以通过提供少量XML配置来声明性地启用对这些其他元数据格式的支持，从而指示容器将Java注解或代码用作元数据格式。

在大多数应用场景中，不需要显式用户代码即可实例化一个Spring IoC容器的一个或多个实例。
例如，在Web应用程序场景中，应用程序文件中的简单八行样板Web描述符XML web.xml 通常就足够了。

![Spring IoC容器](image/container-magic.png)

### 配置元数据
Spring IoC容器配置一组元数据。此配置元数据表示您作为应用程序开发人员如何告诉Spring容器实例化，配置和组装应用程序中的对象。

基于XML的元数据不是配置元数据的唯一允许形式。Spring IoC容器本身与实际写入此配置元数据的格式完全脱钩。
有关在Spring容器中使用其他形式的元数据的信息，请参见：
+ 基于注解的配置：Spring 2.5引入了对基于注解的配置元数据的支持。
+ 基于Java的配置：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。

**基于XML的配置元数据的基本结构：**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->
</beans>
```
+ id属性 是标识单个bean定义的字符串。
+ class属性 定义Bean的类型并使用完全限定的类名。

### !!!实例化容器
给 ApplicationContext构造函数 提供一个或多个位置路径，即资源字符串，可让容器从各种外部资源（例如本地文件系统，Java等）中加载配置元数据CLASSPATH。
```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
服务层对象(services.xml)配置文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```
数据访问对象daos.xml文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

#### 组成基于XML的配置元数据
bean的定义跨越多个XML文件可能很常见，每个单独的XML配置文件都代表一个体系结构中的逻辑层或模块。
您可以使用应用程序上下文构造函数从所有这些XML片段中加载bean定义，如上。
还可以使用一个或多个出现的<import/>元素从另一个文件中加载bean定义。以下示例显示了如何执行此操作：
```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```
所有位置路径是相对于定义文件做进口，因此services.xml必须在同一个目录或类路径位置作为文件做进口，而 messageSource.xml并themeSource.xml必须在resources进口文件的位置下方的位置。
如您所见，斜杠被忽略。但是，鉴于这些路径是相对的，最好不要使用任何斜线。
+ 可以使用相对路径 “ ../” 引用父目录中的文件，但不建议。
+ 可以使用完全限定资源位置来代替相对路径：例如file:C:/config/services.xml或classpath:/config/services.xml。最好为这样的绝对位置保留一个间接寻址“$ {…}”占位符。

### !!!使用容器
ApplicationContext是一个维护bean定义以及相互依赖的注册表的高级工厂的接口。
通过使用方法 T getBean(String name, Class<T> requiredType)，可以检索bean的实例。

将ApplicationContext让你读bean定义和访问它们，如下例所示：
```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

最灵活的变体是GenericApplicationContext与读取器委托结合使用，例如，与XmlBeanDefinitionReaderXML文件结合使用，如以下示例所示：
```
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

实际上，您的应用程序代码应该根本不调用该 getBean()方法，因此完全不依赖于Spring API。
例如，Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖项注入，使您可以通过元数据（例如自动装配注解）声明对特定Bean的依赖项。

## !!!Bean概述
Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的（例如，以XML<bean/>定义的形式 ）。

在容器本身内，这些bean定义表示为 BeanDefinition 对象，这些对象包含（除其他信息外）以下元数据：
+ 包限定的类名称：通常，定义了Bean的实际实现类。
+ Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。
+ 引用其他bean进行其工作所需的bean。这些引用也称为协作者或依赖项。
+ 要在新创建的对象中设置的其他配置设置-例如，池的大小限制或在管理连接池的bean中使用的连接数。

元数据转换为每个bean定义一组属性。下表描述了这些属性：

|属性	|含义	|
|--	|--	|
|Class	|Bean类	|
|Name	|Bean名称	|
|Scope	|Bean范围	|
|Constructor arguments	|依赖注入	|
|Properties	|依赖注入	|
|Autowiring mode	|自动装配协作器	|
|Lazy initialization mode	|懒加载	|
|Initialization method	|初始化回调	|
|Destruction method	|销毁回调L	|

ApplicationContext 除了包含有关如何创建特定bean的信息的bean定义之外，还允许注册在容器外部（由用户）创建的现有对象。
通过方法访问ApplicationContext的BeanFactory来完成的getBeanFactory()，该方法返回BeanFactoryDefaultListableBeanFactory实现。
DefaultListableBeanFactory 通过registerSingleton(..)和 registerBeanDefinition(..)方法支持此注册。
但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用。

### Bean命名
每个bean具有一个或多个标识符。这些标识符在承载Bean的容器内必须唯一。
一个bean通常只有一个标识符。但是，如果需要多个，则可以将多余的别名视为别名。

**在基于XML配置文件，可以使用id属性，name属性或两者来指定bean标识符**

+ id属性可以精确指定一个ID。在Spring 3.1之前的版本中，id属性被定义为一种xsd:ID类型，该类型限制了可能的字符。从3.1开始，它被定义为xsd:string类型。注意，beanid唯一性仍然由容器强制执行，尽管不再由XML解析器执行。
+ name属性可以指定其他别名，使用逗号（,）、分号（;）或 空格 分隔。
+ 如果不提供 name或id显式提供，容器将为该bean生成一个唯一的名称。但是，如果要按名称引用该bean，则必须通过使用ref元素或服务定位器样式查找来提供名称。不提供名称的动机与使用内部bean和自动装配合作者有关。

**Bean命名约定**

约定是在命名bean时将标准Java约定用于实例字段名称。也就是说，bean名称以小写字母开头，并从那里用驼峰式大小写。
这样的名字的例子包括accountManager， accountService，userDao，loginController，等等。

#### 在Bean定义之外为Bean定义别名

在实际定义bean的地方指定所有别名并不总是足够的，有时需要为 在别处定义的bean 引入别名。
在大型系统中通常是这种情况，在大型系统中，配置在每个子系统之间分配，每个子系统都有自己的对象定义集。
以下示例显示了如何执行此操作：
```
<alias name="fromName" alias="toName"/>
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

### Bean实例化
Bean定义实质上是创建一个或多个对象的方法。
容器在被询问时会查看命名bean的配方，并使用该bean定义封装的配置元数据来创建（或获取）实际对象。

**如果使用基于XML的配置元数据，则可以在<bean/>元素的class属性中指定要实例化的对象的类型（或类）**

class属性通常是必需的，可以通过以下Class两种方式之一使用该属性：
+ 通常，在容器本身通过反射性地调用其构造函数直接创建Bean的情况下，指定要构造的Bean类，这在某种程度上等同于使用new运算符。
+ 要指定包含static要创建对象而调用的工厂方法的实际类，在不太常见的情况下，容器要static在类上调用工厂方法来创建Bean。从static工厂方法的调用返回的对象类型可以是同一类，也可以是完全不同的另一类。
+ 如果要为static嵌套类配置Bean定义，则必须使用嵌套类的二进制名称。例如，如果您SomeThing在com.example包中有一个名为的类，并且 SomeThing该类有一个static名为的嵌套类OtherThing，则class bean定义上的属性值将为**com.example.SomeThing$OtherThing**。

#### 用构造函数实例化

当通过构造方法创建一个bean时，所有普通类都可以被Spring使用并兼容。也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。只需指定bean类就足够了。但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

使用基于XML的配置元数据，您可以如下指定bean类：
```
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

#### 用静态工厂方法实例化
在定义使用静态工厂方法创建的bean时，请使用class 属性指定包含static工厂方法的类，并使用命名factory-method为属性的属性来指定工厂方法本身的名称。

以下示例显示如何指定工厂方法：
```
<bean id="clientService" class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了可与前面的bean定义一起使用的类：
```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

#### 使用实例工厂方法实例化
使用实例工厂方法实例化从容器中调用现有bean的非静态方法以创建新bean。
要使用此机制，请将class属性留空，并在factory-bean属性中指定当前（或父容器或祖先容器）中包含要创建该对象的实例方法的Bean的名称，使用factory-method属性设置工厂方法本身的名称。

以下示例显示了如何配置此类Bean：
```
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

相应的类：
```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。
在Spring中，“factory bean” 是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。
相反，FactoryBean（大写）是指特定于Spring的 FactoryBean 实现类。

#### 确定Bean的运行时类型
确定特定bean的运行时类型并非易事。Bean元数据定义中的指定类只是初始类引用，可能与声明的工厂方法结合使用，或者是FactoryBean可能导致Bean的运行时类型不同的类，或者在实例的情况下根本不设置-级别工厂方法（通过指定factory-bean名称解析）。另外，AOP代理可以使用基于接口的代理包装bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。

找出特定bean的实际运行时类型的推荐方法是 **BeanFactory.getType** 调用指定的bean名称。这考虑了上述所有情况，并返回了 BeanFactory.getBean 针对相同bean名称的调用将要返回的对象的类型。

## 依赖关系
典型的企业应用程序不包含单个对象（或Spring术语中的bean）。
即使是最简单的应用程序，也有一些对象可以协同工作，以呈现最终用户视为一致的应用程序。
这部分将说明如何从定义多个独立的Bean定义到实现对象协作以实现目标的完全实现的应用程序。

### 依赖注入
依赖注入（DI）是一个过程，通过该过程，对象仅通过构造函数参数、工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性来定义其依赖关系，然后容器在创建bean时注入那些依赖项。
从根本上讲，此过程是通过使用类的直接构造或服务定位器模式来控制bean自身依赖关系的实例化或位置的bean本身的逆过程（因此称为Control Inversion）。

使用DI原理，代码更加简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。
更易于测试，尤其是当依赖项依赖于接口或抽象基类时，它们允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体：**基于构造函数的依赖注入和基于Setter的依赖注入**。

#### 基于构造函数的依赖注入

基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项。
以下示例显示了只能通过构造函数注入进行依赖项注入的类：
```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
注意，该类没有什么特别的。它是一个POJO，不依赖于特定于容器的接口，基类或注解。

**构造函数参数解析**

构造函数参数解析匹配通过使用参数的类型进行。

如果Bean定义的构造函数参数中没有潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。考虑以下类别：
```
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

```
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当使用诸如的简单类型时，Spring无法确定值的类型，因此在没有帮助的情况下无法按类型进行匹配。考虑以下类别：
```
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

根据**构造函数参数类型**匹配，使用 type属性 显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。如下例所示：
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

根据**构造函数参数索引**匹配，使用该 index属性 来明确指定构造函数参数的索引（索引从0开始），如以下示例所示：
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解决多个简单值的歧义性之外，指定索引还可以解决歧义，其中构造函数具有两个相同类型的参数。

根据**构造函数参数名称**匹配，可以使用构造函数参数名称来消除歧义，如以下示例所示：
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
要使用该功能（构造函数参数名称），必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。
如果您不能或不想使用debug标志编译代码，则可以使用 @ConstructorProperties JDK注解显式命名构造函数参数。然后，样本类必须如下所示：
```
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#### 基于Setter的依赖注入
基于设置器的DI是先通过在调用无参数构造函数或无参数static工厂方法以实例化您的bean，之后在bean上调用setter方法来完成的。

下面的示例显示只能通过使用纯setter注入来依赖注入的类：
```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

**选择基于构造函数或基于setter的DI？**

由于可以混合使用基于构造函数的DI和基于设定值的DI，因此将构造函数用于强制性依赖项并将setter方法或配置方法用于可选的依赖项是一个很好的经验法则。

注意， 在setter方法上使用@Required注解可以使该属性成为必需的依赖项。但是，最好使用带有参数的程序验证的构造函数注入。

#### 依赖性解析过程
容器执行bean依赖项解析，如下所示：
+ 使用ApplicationContext描述所有bean的配置元数据创建和初始化。可以通过XML，Java代码或注解指定配置元数据。
+ 对于每个bean，其依赖项都以属性，构造函数参数或static-factory方法的参数的形式表示（如果使用它而不是普通的构造函数）。实际创建Bean时，会将这些依赖项提供给Bean。
+ 每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。
+ 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring能够以String类型提供值转换成所有内置类型，比如int， long，String，boolean，等等。

**循环依赖**

如果主要使用构造函数注入，则可能会创建无法解决的循环依赖方案。

例如：A类通过构造函数注入需要B类的实例，而B类通过构造函数注入需要A类的实例。如果您将A类和B类的bean配置为相互注入，则Spring IoC容器会在运行时检测到此循环引用，并抛出 BeanCurrentlyInCreationException。

一种可能的解决方案是编辑某些类的源代码，这些类的源代码由设置者而不是构造函数来配置。或者，避免构造函数注入，而仅使用setter注入。换句话说，尽管不建议这样做，但是您可以使用setter注入配置循环依赖项。

与典型情况（没有循环依赖关系）不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。


#### 依赖注入的例子
基于Setter的DI：
```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

基于构造函数的DI：
```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

基于 static工厂方法 以返回对象的实例：
```
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

```
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
static工厂方法的参数由<constructor-arg/>元素提供，就像实际使用构造函数一样。
factory方法返回的类的类型不必与包含static factory方法的类具有相同的类型（尽管在此示例中是）。
非static工厂方法以基本上相同的方式使用（除了使用factory-bean属性代替使用class属性外）。

### 依赖配置详解
Spring基于XML的配置元数据，在<bean/>的子元素<property/>和<constructor-arg/>中配置。

#### 基本类型和字符串
在<property/>元素的 value 属性中指定字符串表示。Spring的转换服务将这些值从String转换为参数的实际类型。
以下示例显示了设置的各种值：
```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```
使用p-namespace进行更简洁的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```

还可以配置java.util.Properties实例：
```
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

<idref>元素是一个防错方式，**确保通过id引用的是另一个Bean，而非字符串**：
```
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```
这与下面配置等效：
```
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

#### 对其他Bean的引用
<ref/>元素是<constructor-arg/>或<property/>的内部元素，可以将属性 bean 的值设置为另一个 bean 的引用。

<ref/>元素通过**bean属性**指定目标bean，可以创建对同一容器或父容器中任何bean的引用。
bean属性 的值可以与目标Bean的 **id属性 或 name属性** 中的值之一相同。
以下示例显示如何使用ref元素：
```
<ref bean="someBean"/>
```

<ref/>元素通过**parent属性**指定目标Bean，将创建对当前容器的父容器中的Bean的引用。
parent属性 的值可以与目标Bean的 **id属性 或 name属性** 中的值之一相同。
```
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

#### 内部Bean
使用 <property/>或<constructor-arg/>元素 内部的 <bean/>元素 定义内部Bean。如下面的示例所示：
```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```
内部bean定义不需要定义的ID或名称。
如果指定，则容器不使用该值作为标识符。容器还会忽略scope创建时的标志，因为内部Bean始终是匿名的，并且始终与外部Bean一起创建。
不能独立访问内部bean，也不能将它们注入到协作bean中而不是封装在bean中。

#### 集合
<list/>，<set/>，<map/>，和<props/>元素分别设置Java中Collection类型List，Set，Map，和Properties的属性和参数。
以下示例显示了如何使用它们：
```
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map的键或set的值可以是以下任意元素：
```
bean | ref | idref | list | set | map | props | value | null
```

**集合合并：**

Spring容器还支持合并集合。子集合的值是合并父集合和子集合的元素的结果，子集合的元素将覆盖父集合中指定的值。
下面的示例演示了集合合并：
```
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```
这一合并行为同样适用于<list/>，<map/>和<set/> 集合类型。
当使用<list/>元素时，将维护与List集合类型关联的语义（排序），父级的值位于所有子级列表的值之前。

#### NULL和空字符串
Spring将属性为空的参数视为空字符串。以下基于XML的配置元数据片段将email属性设置为""的示列:
```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

<null/>元素设置 null。以下一个示例：
```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

#### XML快捷方式p-命名空间
p-namespace允许您使用 **bean元素的属性** （代替嵌套的 <property/>元素）来描述协作Bean的属性值。

以下示例显示了两个XML代码段（第一个使用标准XML格式，第二个使用p-命名空间），它们可以解析为相同的结果：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

下一个示例包括另外两个bean定义，它们都引用了另一个bean：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

#### XML快捷方式c-命名空间
c-namespace允许使用内联属性来配置构造函数参数，可以代替嵌套<constructor-arg/>元素。
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

#### 嵌套属性命名
设置bean属性时，可以使用复合属性命名或嵌套属性命名，只要路径中除了最终属性名称之外的所有组件都没有null。
参考以下bean定义：
```
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```
something具有fred属性，fred属性具有bob属性，bob属性具有sammy属性，并且最终sammy属性被设置为值123。

### 使用depends-on
如果一个bean是另一个bean的依赖项，则通常意味着将一个bean设置为另一个bean的属性。
通常，您可以使用基于XML的配置元数据中的<ref/> 元素来完成此操作，但是，有时bean之间的依赖性不太直接。
例如，何时需要触发类中的静态初始值设定项，用于数据库驱动程序注册。
**depends-on属性可以在初始化使用此元素的bean之前显式强制初始化一个或多个bean**。

以下示例使用该depends-on属性表示对单个bean的依赖关系：
```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表达对多个bean的依赖关系，请提供一个bean名称列表作为该depends-on属性的值（逗号，空格和分号是有效的分隔符）：
```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

depends-on属性既可以指定初始化时间依赖性，也可以仅在单例情况下指定相应的销毁时间依赖性。
与给定bean 定义了的依赖关系的（depends-on） 从属bean首先被销毁，然后再销毁给定bean本身。

### 延迟加载Bean
默认情况下，ApplicationContext 实现会在初始化过程中创建和配置所有单例Bean。
如果不希望使用此行为，则可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。
延迟初始化的bean告诉IoC容器在首次请求时（而不是在启动时）创建一个bean实例。

在XML中，此行为由 <bean/>元素上的lazy-init属性 控制，如以下示例所示：
```
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

您还可以通过使用元素default-lazy-init上的属性在容器级别上控制延迟初始化 <beans/>，如以下示例所示：
```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 自动装配
Spring容器可以自动装配协作bean之间的关系。

自动装配具有以下优点：
+ 自动装配可以大大减少指定属性或构造函数参数的需要。
+ 随着对象的发展，自动装配可以更新配置。

使用基于XML的配置元数据时，您可以使用 bean元素 的 autowire属性 为 bean定义指定自动装配模式。

自动装配功能具有四种模式。下表描述了四种自动装配模式：

|模式	|说明	|
|--	|--	|
|no（默认）	|无自动装配。Bean引用必须由ref元素定义。	|
|byName	|按名称自动装配。Spring查找与需要自动装配的属性同名的bean。	|
|byType	|如果容器中恰好存在一个属性类型的bean，则使该属性自动连接。如果存在多个，则会引发致命异常。	|
|constructor	|类似于byType但适用于构造函数参数。	|

#### 自动装配的局限性和缺点
+ 显式依赖项property和constructor-arg设置始终会覆盖自动装配。
+ 您无法自动装配简单的属性，例如基元 Strings，和Classes（以及此类简单属性的数组）。
+ 自动装配不如显式接线精确。Spring管理的对象之间的关系不再明确记录。
+ 装配信息可能不适用于需要从Spring容器生成文档的工具。
+ 容器内的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。对于数组，集合或 Map实例，这不一定是问题。但是，对于需要单个值的依赖项，不会任意解决此歧义。如果没有唯一的bean定义可用，则引发异常。

#### 从自动装配中排除Bean
在每个bean的基础上，您可以从自动装配中排除一个bean。
使用Spring的XML格式，将bean元素的autowire-candidate属性设置为false。
容器使特定的bean定义不用于自动装配基础结构（包括注解配置，例如@Autowired）。
autowire-candidate属性仅影响基于类型的自动装配。它不会影响按名称的显式引用，即使未将指定的Bean标记为自动装配候选，该名称也可得到解析。因此，如果名称匹配，按名称自动装配仍会注入Bean。

### 方法注入
在大多数应用场景中，容器中的大多数bean是单例模式。当一个单例Bean需要与另一个单例Bean协作或一个非单例Bean需要与另一个非单例Bean协作时，通常可以通过将一个Bean定义为另一个Bean的属性来处理依赖关系。

当bean的生命周期不同时会出现问题。假设单例bean A需要使用非单例（原型）bean B，也许在A的每个方法调用上都使用。容器仅创建一次单例bean A，因此只有一次机会来设置属性。每次需要一个容器时，容器都无法为bean A提供一个新的bean B实例。

一个解决方案是放弃某些控制反转。通过实现接口 ApplicationContextAware，每次 BeanA 需要beanB 时，都通过容器调用getBean("B")方法来请求（通常是新的）beanB实例。
以下示例显示了此方法： 
```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
这并不是理想的，因为业务代码知道并耦合到Spring框架。

#### 查找方法注入
查找方法注入是容器重写Bean上的方法，并返回容器中另一个命名Bean的查找结果的能力。
Spring框架通过使用CGLIB库中的字节码生成来动态生成覆盖该方法的子类，从而实现此方法注入。

提示：
+ 为了使此动态子类起作用，Spring Bean容器的子类的类不能为final，要覆盖的方法也不能为final。
+ 对具有abstract方法的类进行单元测试，需要对该类进行子类化，并提供该abstract方法的存根实现。
+ 组件扫描也需要具体的方法，这需要具体的类。
+ 查找方法不适用于工厂方法，尤其不适@Bean用于配置类中的方法，因为在这种情况下，容器不负责创建实例，因此无法在其上创建运行时生成的子类。

调整后的CommandManager类没有任何Spring的依赖，如下示列所示：
```
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

要注入的方法需要以下形式的签名：
```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```
如果方法为abstract，则动态生成的子类将实现该方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。

示例：
```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
如果需要一个新的 myCommand 实例，被标识为commandManager的bean就会调用其createCommand()方法创建新示列。
请注意，如果myCommand是单例，则每次都会返回Bean的相同实例。

另外，在基于注解的组件模型中，可以通过@Lookup注解声明一个查找方法，示例：
```
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```
或者：
```
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

#### 任意方法注入
略

## Bean作用域
Spring支持6个作用域，其中有4个只有在使用网络的ApplicationContext中可用。
此外可以创建自定义作用域。

Bean作用域：

|作用域	|描述	|
|--	|--	|
|singleton（默认）	|将单个bean的作用域限定为每个SpringIoC容器只有一个对象实例。	|
|prototype	|将单个bean的作用域限定为任意数量的对象实例。	|
|request	|将单个bean的作用域限定为 单个HttpRequest 的生命周期。	|
|session	|将单个bean的作用域限定为 HttpSession 的生命周期。	|
|application	|将单个bean的作用域限定为 ServletContext的生命周期	|
|websocket	|将单个bean的作用域限定为 WebSocket的生命周期	|

从Spring 3.0开始，线程作用域可用，但默认情况下未开启。

### 单例范围
当定义一个bean为单例时，Spring IoC容器将为该bean所定义的对象创建一个实例。
单例对象存储在高速缓存中，并且对该命名bean的所有后续请求和引用都返回该高速缓存的对象。

Spring的单例概念不同于“GoF模式”一书中定义的singleton模式。
GoF单例对对象的范围进行硬编码，以使每个ClassLoader只能创建一个特定类的一个实例。

最好将Spring单例的范围描述为每个容器和每个bean。
如果您在单个Spring容器中为特定类定义一个bean，则Spring容器将创建该bean定义所定义的类的一个且只有一个实例。

XML中，要将bean定义为的单例，示例如下：
```
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

### 原型范围
每次对原型范围的bean发起请求时，容器都会创建一个新bean实例。
例如，Bean被注入到另一个Bean中，或者通过容器上的getBean()方法调用来请求它。

通常，**将原型作用域用于有状态Bean，将单例作用域用于无状态Bean**。
数据访问对象（DAO）通常不配置为原型，因为典型的DAO不拥有任何对话状态。

在XML中将bean定义为原型，示例如下：
```
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域不同，Spring不管理原型作用域Bean的完整生命周期。
容器将实例化，配置或组装原型对象，然后将其交给客户端，但不会对该实例作进一步记录。
所有作用域的的Bean都会调用初始化生命周期回调方法，但原型作用域的Bean不会调用销毁生命周期方法。
客户端代码必须清除原型作用域内的对象，并释放原型Bean拥有的昂贵资源。

### 具有原型Bean依赖关系的单例Bean
当使用对原型Bean有依赖性的单例作用域Bean时，依赖关系在单例Bean实例化时就已完成。
因此，如果将依赖项原型的bean依赖项注入到单例范围的bean中，则将实例化新的原型bean，然后将依赖项注入到单例bean中，原型实例是曾经提供给单例范围的bean的唯一实例。

如果希望单例作用域的bean在运行时重复获取原型作用域的bean的新实例，就不能将原型作用域的bean依赖项注入到您的单例bean中，因为当Spring容器实例化单例bean并解析并注入其依赖项时，该注入仅发生一次。

如果在运行时不止一次需要原型bean的新实例，请使用 【方法注入】 方式。

### Request Session Application WebSocket 作用域
request，session，application，和websocket作用域只能在基于web的Spring容器（如XmlWebApplicationContext）中使用。
如果在普通的Spring容器（如ClassPathXmlApplicationContext），将会抛出一个异常IllegalStateException（未知的bean作用域）。

#### 初始Web配置
为了使用web作用域，需要做少量的初始化配置。

如何完成此初始设置取决于您的特定Servlet环境。

如果是在Spring Web MVC中访问作用域化的bean，实际上请求是在Spring的DispatcherServlet中处理的，不需要进行特殊的设置。

如果使用Servlet 2.5 Web容器，并且在Spring的DispatcherServlet之外处理请求（如JSF或Struts），则需要注册 org.springframework.web.context.request.RequestContextListener ServletRequestListener。

如果使用Servlet 3.0+，可以使用 WebApplicationInitializer 接口以编程方式完成此操作。

对于较旧的容器，将以下声明添加到Web应用程序的web.xml文件中：
```
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

另外，如果监听器设置存在问题，请考虑使用Spring的 RequestContextFilter。过滤器映射取决于Web应用程序配置，因此需要对其进行更改。
以下清单显示了Web应用程序的过滤器部分：
```
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

DispatcherServlet，RequestContextListener和RequestContextFilter实际上做着完全相同的事情，即将HTTP请求对象绑定到正在为该请求提供服务的Thread对象，这使得在请求链和会话范围内的bean可以在调用链的更下游使用。

####  Request作用域
独立于单个 Http Request。

xml配置：
```
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

注解配置：
```
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

#### Session作用域
独立于单个 Http Session。

xml配置：
```
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

注解配置：
```
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```


#### Application作用域
Application作用域的Bean在整个Web应用程序只定义一次实例。
Application作用域是在ServletContext级别上，并存储为常规 ServletContext 属性。
这类似于Spring的单例作用域，但有两个重要的区别：
+ Application作用域是整个ServletContext只有一个实例，而不是每个Spring的ApplicationContext（在一个Web应用程序中可能都有多个） 一个示列。
+ Application作用域作为ServletContext属性是公开的。


xml配置：
```
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

注解配置：
```
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

#### 范围Bean作为依赖项
略

### 自定义作用域
Bean作用域机制是可扩展的。可以定义自己的作用域，甚至重新定义现有作用域。
不能覆盖内置作用域 singleton 和 prototype 作用域。

#### 创建自定义作用域
要创建自定义作用域，需要实现接口 org.springframework.beans.factory.config.Scope。

Scope接口有四种方法可以从作用域中获取对象，将它们从作用域中删除，然后将其销毁。

略

#### 使用自定义作用域
略

## 定义Bean特性
Spring框架提供了许多接口，可用于自定义Bean的特性。如下：
+ 生命周期回调接口
+ ApplicationContextAware和BeanNameAware接口
+ 其他Aware接口

### 生命周期回调
通过容器对bean生命周期的进行管理进行，可以实现Spring的InitializingBean和DisposableBean接口。
调用前者的afterPropertiesSet()方法和和后者的destroy()方法，分别在bean的初始化和销毁时执行一些操作。

JSR-250（Java Specification Requests Java 规范提案）的@PostConstruct和@PreDestroy注解被认为是在现代Spring应用程序中接收生命周期回调的最佳实践，使用这些注解意味着您的bean没有耦合到特定于Spring的接口。
如果不想使用JSR-250注解，仍然要解耦合，可以通过init-method和destroy-methodbean定义元数据。

除了初始化和销毁​​回调外，Spring托管的对象还可以实现Lifecycle接口，以便这些对象可以在容器自身生命周期的驱动下参与启动和关闭过程。

#### 初始化回调
实现org.springframework.beans.factory.InitializingBean接口允许bean执行初始化工作。InitializingBean接口指定一个方法：
```
void afterPropertiesSet() throws Exception;
```

示列：
```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```
```
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

可以使用bean元素的init-method属性指定具有无参数签名的方法的名称，这是解耦的。示列如下：
```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```
```
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

#### 销毁回调
实现org.springframework.beans.factory.DisposableBean接口允许bean在被销毁时回调。DisposableBean接口指定一个方法：
```
void destroy() throws Exception;
```

示列：
```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```
```
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

可以使用bean元素的destroy-method属性指定具有无参数签名的方法的名称，这是解耦的。示列如下：
```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```
```
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

#### 默认初始化和销毁​​方法
理想情况下，生命周期回调方法的名称应在整个项目中标准化，以便所有开发人员都使用相同的方法名称并确保一致性。

假设初始化回调方法命名为init()，而销毁回调方法命名为destroy()。示列如下：
```
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```
```
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

beans元素的属性default-init-method，可以使Spring IoC容器将在bean类上的init()方法识别为初始化方法回调。
beans元素的属性default-destroy-method，在XML中以类似方式的配置销毁方法回调 。

Spring容器保证在为bean提供所有依赖项后，立即调用初始化回调。

#### 组合生命周期机制
从Spring 2.5开始，您可以使用三种方式来控制Bean生命周期行为：
+ 实现InitializingBean和 DisposableBean回调接口
+ 使用init()和destroy()方法
+ 使用注解@PostConstruct和@PreDestroy

如果为一个bean配置了多个生命周期机制，并且为**每个机制配置了不同的方法名称**，则每个配置的方法都将按照此注解后列出的顺序运行。但是，如果为多个生命周期机制中的多个生命周期机制配置了相同的方法名（如初始化方法init），则该方法将运行一次。

为同一个bean配置的不同初始化方法，加载顺序如下：
+ 使用注解@PostConstruct的方法
+ InitializingBean接口的afterPropertiesSet()方法
+ XML配置的init()方法

销毁方法的调用顺序相同：
+ 使用注解@PreDestroy的方法
+ DisposableBean接口destroy()方法
+ XML配置的destroy()方法

#### 启动和关机回调
Lifecycle接口定义了生命周期要求的基本方法：
```
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现该Lifecycle接口。
当ApplicationContext接收到启动和停止信号时（例如，对于运行时的停止/重新启动场景），它将这些调用级联到在该上下文中定义的Lifecycle的所有实现类，它通过委派LifecycleProcessor进行通知。

LifecycleProcessor本身是Lifecycle 接口的扩展。它还添加了两种其他方法来对刷新和关闭的上下文做出反应。
LifecycleProcessor如下面的清单所示：
```
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

org.springframework.context.Lifecycle接口是用于显式启动和停止通知的普通协议，并不意味着在上下文刷新时自动启动。

启动和关闭调用的顺序很重要,如果任何两个对象之间存在“依赖”关系，则依赖方在其依赖之后开始，而在依赖之前停止。
但有时直接依赖项是未知的，可能只知道某种类型的对象应该先于另一种类型的对象开始。
在这些情况下，SmartLifecycle接口提供了一个选项，即父接口Phased的getPhase()方法。

Phased接口的定义：
```
public interface Phased {

    int getPhase();
}
```

SmartLifecycle接口的定义：
```
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，相位最低的对象首先启动；停止时，遵循相反的顺序。
实现SmartLifecycle，并且在其方法getPhase()返回的对象定位Integer.MIN_VALUE将是第一个启动且最后一个停止的对象。
相位值Integer.MAX_VALUE表示该对象应最后启动并首先停止（可能因为该对象取决于正在运行的其他进程）。

#### 在非Web应用程序中正常关闭Spring IoC容器
如果在非Web应用程序环境中（例如，在富客户端桌面环境中）使用Spring的IoC容器，请向JVM注册一个关闭钩子。
这样做可以确保正常关机，并在您的Singleton bean上调用相关的destroy方法，以便释放所有资源。

要注册关闭挂钩，请调用接口registerShutdownHook()上声明的方法ConfigurableApplicationContext，如以下示例所示：
```
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

### ApplicationContextAware和BeanNameAware接口
当创建实现org.springframework.context.ApplicationContextAware接口的对象实例时，该实例将获得对ApplicationContext接口的引用。
以下清单显示了ApplicationContextAware接口的定义：
```
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

自动装配是获得对ApplicationContext的引用的另一种方法：
```
@Autowired
private ApplicationContext context;
```

当创建一个实现该 org.springframework.beans.factory.BeanNameAware接口的类时，该类将获得对其关联对象定义中定义的名称的引用。
以下清单显示了BeanNameAware接口的定义：
```
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

### 其他Aware接口
除了ApplicationContextAware和BeanNameAware，Spring还提供了各种各样的Aware回调接口：

|名称	|注入依赖	|
|--	|--	|
|ApplicationContextAware	|ApplicationContext	|
|ApplicationEventPublisherAware	|附带事件发布者的ApplicationContext	|
|BeanClassLoaderAware	|类加载器	|
|BeanFactoryAware	|BeanFactory	|
|BeanNameAware	|声明bean的名称	|
|LoadTimeWeaverAware	|定义的编织器，用于在加载时处理类定义	|
|MessageSourceAware	|解决消息的已配置策略（支持参数化和国际化）	|
|NotificationPublisherAware	|JMX通知发布者	|
|ResourceLoaderAware	|配置的加载程序，用于对资源的低级访问	|
|ServletConfigAware	|当前容器中运行的ServletConfig。仅在可感知网络的Spring中有效 ApplicationContext	|
|ServletContextAware	|当前容器中运行的ServletContext。仅在可感知网络的Spring中有效 ApplicationContext	|

使用这些接口会将您的代码与Spring API绑定在一起，并且不遵循“控制反转”原则。

## Bean定义继承
Bean定义可以包含许多配置信息，包括构造函数参数，属性值和特定于容器的信息，例如初始化方法，静态工厂方法名称等。

子bean定义从父定义继承配置数据。子定义可以覆盖某些值或根据需要添加其他值。
使用父bean和子bean定义可以节省很多输入。实际上，这是一种模板形式。

当使用基于XML的配置元数据时，可以通过使用parent属性来指示子bean定义，并指定父bean作为此属性的值。
以下示例显示了如何执行此操作：
```
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果父定义未指定类，请根据abstract需要显式标记父bean定义，如以下示例所示：
```
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

## 容器扩展
略

### 使用BeanPostProcessor自定义Bean

### 使用BeanFactoryPostProcessor自定义配置元数据

### 使用FactoryBean自定义实例化逻辑

## !!!基于注解的容器配置
**注解配置比XML配置更好吗？**

每种方法都有其优缺点，通常由开发人员决定哪种策略更适合他们。
+ 注解配置比在声明中提供了很多上下文，从而使配置更短，更简洁。
+ XML配置擅长连接组件而不接触其源代码或重新编译它们。

一些开发人员更喜欢靠近源的位置注解方式。
另一些开发人员则认为带注解的类不再是POJO，而且配置变得分散并且难以控制。
无论选择如何，Spring都可以容纳两种样式，甚至可以将它们混合在一起。
**注解注入在XML注入之前执行，因此XML配置将覆盖通过两种方法连接的属性的注解。**

可以将它们注册为单独的bean定义，但也可以通过在基于XML的Spring配置中包含以下标记来隐式注册它们。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

**context:annotation-config 只会查找它所在的应用程序上下文中的bean的注解。**
如果把 context:annotation-config 放在 在WebApplicationContextt 中，将会只扫描controllers的bean中的注解，而不会扫描services。

### @Required
@Required表示必须在配置时通过bean定义中的显式属性值或通过自动装配来填充受影响的bean属性。

@Required适用于bean属性的setter方法，示例如下：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
	
从 Spring Framework 5.1开始，正式弃用了@Required注解。

### @Autowired
JSR330的注解 @Inject 可以代替Spring的注解 @Autowired。

@Autowired注解应用于构造函数，示例以下：
```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

@Autowired注解应用于setter方法，示例以下：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

@Autowired注解应用于字段，示例以下：
```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

@Autowired注解应用于数组或集合中，Spring将提供特定类型的所有bean，示例以下：
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```
```
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
如果希望数组或列表中的项目以特定顺序排序，则目标bean可以实现org.springframework.core.Ordered接口或使用@Order或标准@Priority注解。

@Autowired注解应用于Map，value包含所有预期类型的​​bean，key包含相应的bean名称，示例以下：
```
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
默认情况下，当给定注入点没有匹配的候选bean可用时，自动装配将失败。对于声明的数组，集合或映射，至少应有一个匹配元素。

@Autowired注解的required属性设置为false，标记为非必须的注入点。
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

可以通过Java 8来表达特定依赖项的非必需性质java.util.Optional，如以下示例所示：
```
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

从Spring Framework 5.0开始，可以使用@Nullable注解：
```
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

可以使用@Autowired注解注入常见的接口：BeanFactory，ApplicationContext，Environment，ResourceLoader， ApplicationEventPublisher，和MessageSource。
这些接口及其扩展接口（例如ConfigurableApplicationContext或ResourcePatternResolver）将自动解析，而无需进行特殊设置。
以下示例自动装配ApplicationContext对象：
```
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```
@Autowired，@Inject，@Value，和@Resource注解是由Spring BeanPostProcessor实现的。
这意味着不能在BeanPostProcessor类型或BeanFactoryPostProcessor类型中使用这些注解，必须通过使用XML配置或Spring的@Bean方法显式地“连接” 。

### 使用@Primary 调整自动装配
由于按类型自动装配可能会出现多个候选对象，因此有必要对选择过程进行更多控制，一种解决方法是使用Spring的@Primary注解。
@Primary指示当多个bean是要自动装配到单值依赖项的候选对象时，应给予特定bean优先权。
如果候选bean中恰好存在一个@Primary的bean，它将成为自动装配的值。

如下定义firstMovieCatalog为MovieCatalog的主要配置：
```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

如下MovieRecommender将自动配置firstMovieCatalog：
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

基于xml的bean元素的配置示列：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

### 使用@Qualifier 调整自动装配
@Primary可以确定一个主要候选对象，是按类型使用自动装配的有效方法，当需要更精准地控制选择过程时，可以使用Spring的@Qualifier注解。
@Qualifier注解可以将限定符值与特定的参数相关联，从而缩小类型匹配的范围，以便为每个参数选择特定的bean。
示例以下：
```
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```
或者用在构造方法：
```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

基于xml的bean元素的配置示列：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

### 使用泛型 调整自动装配
除了@Qualifier注解之外，可以将Java泛型类型用作资格的隐式形式。
示列如下：
```
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

```
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean

// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

### 使用CustomAutowireConfigurer
CustomAutowireConfigurer 是一个BeanFactoryPostProcessor，可以注册自己的自定义限定符注解类型。
```
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

AutowireCandidateResolver通过以下步骤确定自动装配候选对象：
+ bean中定义的autowire-candidate值
+ beans元素中定义的default-autowire-candidates值
+ @Qualifier注解 和 通过CustomAutowireConfigurer注册的所有自定义注解

当多个bean符合自动装配条件时，如果恰好有一个bean的primary属性设置为true，则将其选中。

### @Resource
Spring支持在字段或setter方法使用JSR-250的@Resource注解进行自动注入。
这是JavaEE中的常见模式：例如JSF和JAX-WS。

@Resource有个一个属性name，Spring将name值视为要注入的Bean的名称。示例如下：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
如果未指定name，则根据字段名称或setter方法获取名称：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

当@Resource根据name匹配失败后，将会像@Autowired一样，根据类型进行匹配。
在以下示例中，首先查找名称"customerPreferenceDao"查找，然后根据类型CustomerPreferenceDao查找Bean：
```
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

**@Autowired 与 @Resource：**
+ @Autowired为Spring的注解。
+ @Resource为JSR-250注解。
+ @Autowired，首先按类型选择候选bean，然后在这些候选对象中选取的特定的bean。
+ @Resource，通过名称选择bean，与类型无关。
+ @Autowired用于字段，构造函数和多参数方法，从而允许在参数级别缩小限定符注解的范围。
+ @Resource 仅支持具有单个参数的字段和bean属性设置器方法。

### @Value
@Value通常用于注入外部属性，示例如下：
```
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

使用以下配置：
```
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```
和application.properties文件：
```
catalog.name=MovieCatalog
```
在这种情况下，catalog参数和字段将赋予MovieCatalog值。

Spring默认提供了一个的宽松的内嵌值解析器。
它尝试解析属性值，如果无法解析，则会将属性名称（例如${catalog.name}）作为值。

如果要严格控制不存在的值，则应声明一个PropertySourcesPlaceholderConfigurerbean，如以下示例所示：
```
@Configuration
public class AppConfig {

     @Bean
     public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
           return new PropertySourcesPlaceholderConfigurer();
     }
}
```
当使用JavaConfig配置PropertySourcesPlaceholderConfigurer时，@Bean方法必须是static的。

使用上述配置，如果${}无法解析任何占位符，将会使Spring初始化失败。

**Spring Boot默认情况下配置了一个PropertySourcesPlaceholderConfigurerBean，它将从application.properties和application.yml文件中获取属性。**

当@Value包含**SpEL表达式**时，该值将在运行时动态计算，如以下示例所示：
```
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```
支持更复杂的数据结构：
```
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

### @PostConstruct和@PreDestroy
CommonAnnotationBeanPostProcessor不仅支持了了@Resource注解，也支持了JSR-250的生命周期注解：javax.annotation.PostConstruct和 javax.annotation.PreDestroy。
在Spring 2.5中引入了对这些注解的支持，为 初始化回调和 销毁回调 提供了一种替代方式。

在以下示例中，缓存在初始化时预先填充，并在销毁时清除：
```
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

## !!!类路径扫描和组件管理
可以通过扫描类路径来隐式检测候选组件的选项，候选组件是与过滤条件匹配的类，并具有在容器中注册的相应bean定义。

可以使用注解（例如@Component），AspectJ类型表达式或您自己的自定义过滤条件来选择哪些类在容器中注册了bean定义。
	
从Spring 3.0开始，Spring JavaConfig项目提供的许多功能是核心Spring Framework的一部分，这使得可以使用Java而不是使用传统的XML文件来定义bean。

### !!!@Component和其他组件注解
Spring提供一系列组件注解：@Component，@Repository，@Service，和 @Controller。

@Component是任何Spring托管组件的通用构造型。
@Repository，@Service和@Controller分别是@Component针对特定用例的专业化（分别在持久性，服务和表示层）。 

因此，您可以用@Component来注解左右组件，但是，通过@Repository，@Service或者@Controller进行注解，能更好地适合于通过工具处理，或与切面进行关联。

另外，@Repository在持久层中已经支持将其作为自动异常转换的标记。

### 元注解和组合注解
Spring提供的许多注解都可以在您自己的代码中用作元注解。元注解是可以应用于另一个注解的注解。

@Service注解有@Component注解，如下面的示例所示：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```

### !!!自动扫描
**Spring可以自动检测构造型类，并使用ApplicationContext来注册相应的BeanDefinition实例。**

例如，以下两个类别有资格进行这种自动检测：
```
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}

@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，您需要添加 @ComponentScan 到 @Configuration类中，其中basePackages属性扫描范围。
可以指定一个逗号，分号或空格分隔的扫描范围列表。

示列如下：
```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

使用XML：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```
**使用 context:component-scan 将会隐式启用 context:annotation-config 功能。**

### 扫描过滤器
**通常，有@Component，@Repository，@Service，@Controller， @Configuration的类会是唯一检测到的候选组件。**
但是，可以通过自定义过滤器来修改和扩展此行为。

@ComponentScan注解的includeFilters或excludeFilters属性。
XML配置中context:component-scan元素的 context:include-filter或context:exclude-filter 子元素<。
每个过滤器元素都有type和expression属性。

过滤选项示例：
|过滤器类型	|示例	|描述	|
|--	|--	|--	|
|annotation (default)	|org.example.SomeAnnotation	|目标组件中要存在相应注解	|
|assignable	|org.example.SomeClass	|目标组件要分配给（扩展或实现）的类（或接口）|
|aspectj	|org.example..*Service+	|目标组件要匹配的AspectJ类型表达式	|
|regex	|org\.example\.Default.*	|目标组件要匹配正则表达式	|
|custom	|org.example.MyTypeFilter	|org.springframework.core.type.TypeFilter接口的自定义实现	|

以下示例，显示了忽略所有@Repository注解并改为使用“stub”存储库的配置：
```
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```
等效的XML配置：
```
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

### 组件中定义Bean元数据
可以在带@Configuration注解的类中使用@Bean注解定义Bean元数据。

以下示例显示了如何执行此操作：
```
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

其它bean特性定义，@Qualifier，@Scope，@Lazy和自定义限定器注解。示例如下：
```
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

**@component类中的@Bean注解和@Configuration的处理方式不同。**
@Component并未CGLIB增强类来拦截方法和字段的调用。
@Configuration通过CGLIB创建Bean元数据引用以协作对象。

CGLIB代理是一种方法，通过该方法可以调用类中的方法或@Bean方法中的字段。
此类方法不是用普通的Java语义调用的，而是通过容器进行的，以提供Spring Bean的常规生命周期管理和代理。

### 组件命名
当组件被自动检测为扫描过程的一部分时，其bean名称由该BeanNameGenerator扫描器已知的策略生成。
如果这样的注解不包含name和value，则缺省bean名称生成器将返回不使用大写字母的非限定类名称。

如果检测到以下组件类，则名称分别为myMovieLister和movieFinderImpl：
```
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```
```
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

如果不想依赖默认的Bean命名策略，则可以提供自定义Bean命名策略。
首先，实现 BeanNameGenerator 接口，并确保包括默认的no-arg构造函数。
然后，在配置扫描器时提供完全限定的类名。
示例以下：
```
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    // ...
}
```
```
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

### 组件作用域
与Spring管理的组件一样，自动检测到的组件的默认范围也是最常见的范围是singleton。
可以使用@Scope注解指定的其他范围。
如以下示例所示：
```
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

### 提供带有注解的限定符元数据
一下示列使用@Qualifier注解和自定义限定符注解在解析自动装配候选时提供细粒度的控制。
```
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```
```
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```
```
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

### 组件索引
尽管类路径扫描非常快，但可以通过在编译时创建静态候选列表来提高大型应用程序的启动性能。
在这种模式下，作为组件扫描目标的所有模块都必须使用此机制。

要生成索引，请向每个包含组件的模块添加附加依赖关系，这些组件是组件扫描指令的目标。
以下示例显示了如何使用Maven进行操作：
```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.3.1</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

这将在jar文件中生成一个 META-INF/spring.components 文件。

## 使用JSR-330标准注解
从Spring 3.0开始，Spring提供对JSR-330标准注解（依赖注入）的支持。
这些注解的扫描方式与Spring注解的扫描方式相同。
要使用它们，需要在类路径中有相关的jar：
```
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

略

### @Inject和@Named
### @Named和@ManagedBean
### JSR-330标准注解的局限性

## !!!基于Java的容器配置
### !!!@Bean和@Configuration
Spring中支持的Java配置主要是 @Configuration注解的类 和 @Bean注解的方法。

@Bean注解被用于指示一个方法实例，可以配置并初始化到由Spring IoC容器进行管理的新对象。
@Bean可以在任意@Component注解的类中使用，但最常用的搭配是@Configurationbean。

@Configuration表明一个类的主要目的是**作为定义Bean的来源**。
@Configuration类可以通过调用同一类中的其他@Bean方法来定义Bean之间的依赖关系。

最简单的@Configuration类如下：
```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
这等效于以下XML：
```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

### 使用AnnotationConfigApplicationContext实例化Spring容器
在Spring 3.0，引入了AnnotationConfigApplicationContext了。

这种通用的ApplicationContext实现不仅可以接受@Configuration类作为输入，还可以接受普通@Component类和使用JSR-330元数据注解的类。

当提供@Configuration类作为输入时，@Configuration类本身将注册为Bean，并且该类中所有已声明@Bean的方法也将注册为Bean。

当提供@Component和JSR-330类时，它们被注册为bean定义，并且假定在必要时在这些类中使用DI元数据，例如@Autowired或@Inject。

#### 配置简单
与实例化ClassPathXmlApplicationContext时将Spring XML文件用作输入的方式相同，实例化AnnotationConfigApplicationContext时可以将@Configuration注解的类用作输入。
这允许完全不使用XML来实例化Spring容器，如下面的示例所示：
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

AnnotationConfigApplicationContext不仅限于使用@Configuration类，还可以使用@Component和带有JSR-330注解的类，如以下示例所示：
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

可以使用无参构造函数实例化AnnotationConfigApplicationContext ，然后使用register()方法配置它。
以编程方式构建AnnotationConfigApplicationContext时，此方法特别有用。
以下示例显示了如何执行此操作：
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

#### 启用组件扫描
启用组件扫描，可以按如下方式注解@Configuration类：
```
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

AnnotationConfigApplicationContext可以使用scan(String…​)方法实现扫描功能，如以下示例所示：
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

#### 通过AnnotationConfigWebApplicationContext支持Web应用程序
一个AnnotationConfigApplicationContext的Web应用变种是的AnnotationConfigWebApplicationContext。
基于此实现，可以配置Spring的 ContextLoaderListener， servlet listener，Spring MVC DispatcherServlet等。

以下web.xml代码片段配置了典型的Spring MVC Web应用程序（请注意使用contextClasscontext-param和init-param）：
```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>	
```

### @Bean注解
@Bean是方法级别的注解。
@Bean可以在@Configuration注解的类或@Component注解的类中使用。

#### Bean定义
定义一个bean，可以对方法使用@Bean注解。默认情况下，Bean名称与方法名称相同。
以下示例显示了@Bean方法声明：
```
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```
或者
```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

#### Bean依赖
带@Bean注解的方法可以具有任意数量的参数，这些参数描述了构建该bean所需的依赖关系。
例如，如果TransferService需要AccountRepository，可以使用以下示例所示：
```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```
解析机制与基于构造函数的依赖注入几乎相同。

#### Bean生命周期回调
使用@Bean注解的任何类支持常规的生命周期回调，并且可以使用JSR-250中的@PostConstruct和@PreDestroy注解。

支持常规的Spring生命周期回调。如果bean实现InitializingBean，DisposableBean或Lifecycle，则容器将调用它们各自的方法。

支持标准*Aware接口集（例如BeanFactoryAware， BeanNameAware， MessageSourceAware， ApplicationContextAware等）。

示例如下：
```
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

#### Bean作用域
Spring使用@Scope注解指定bean的作用域。

默认范围是singleton，但是您可以使用@Scope注解覆盖它，如以下示例所示：
```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

#### Bean命名
默认情况下，配置类使用@Bean方法的名称作为结果bean的名称。

可以使用name属性覆盖此功能，如以下示例所示：
```
@Configuration
public class AppConfig {

    @Bean(name = "myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

#### Bean别名
有时希望为Bean提供多个名称，称为Bean别名。 
@Bean注解的name属性接受String数组。以下示例显示了如何为bean设置多个别名：
```
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

#### Bean描述
有时，提供有关bean的更详细的文本描述会很有帮助。如暴露出bean（可能通过JMX）以进行监视时，这特别有用。

可以使用 @Description 注解，如以下示例所示：
```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

### @Configuration注解
@Configuration是类级别的注解，指示对象是Bean定义的源。

#### 注入bean间的依赖关系
当bean彼此依赖时，表达这种依赖就像让一个bean方法调用另一个一样简单，如以下示例所示：
```
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

#### 查找方法注入
查找方法注入是一项高级功能，在单例作用域的bean依赖于原型作用域的bean的情况下，这很有用。
将Java用于这种类型的配置为实现此模式提供了自然的方法。下面的示例演示如何使用查找方法注入：
```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

#### 有关基于Java的配置如何在内部工作的更多信息
以下示例显示了一个带@Bean注解的方法被调用两次，而实际上只会执行一次：
```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```
clientDao()被clientService1()和clientService2()分别调用了一次。
由于此方法创建的新实例ClientDaoImpl并返回它，因此通常希望有两个实例（每个服务一个）。
但这在在Spring中肯定是有问题的，实例化的bean在默认情况下具有singleton作用域，这就是神奇的地方，所有@Configuration类在启动时都使用CGLIB子类化。
在子类中，子方法在调用父方法并创建新实例之前，首先检查容器中是否有任何缓存（作用域）的bean。

### !!!组合基于Java的配置
#### !!!@Import注解
正如XML文件中的import元素来帮助模块化配置一样，@Import注解允许@Bean从另一个配置类加载配置信息。
如以下示例所示：
```
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```
这样，实例化应用上下文无需同时指定两者ConfigA和ConfigB，只需显式提供ConfigB。
如以下示例所示：
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

考虑以下具有多个@Configuration 类的更真实的场景，每个类都取决于其他类中声明的bean：
```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

#### !!!有条件地启用@Configuration类或@Bean方法
基于某些系统状态，有条件地启用或禁用@Configuration类或@Bean方法通常很有用。

@Profile 注解可以在Spring中启用了特定环境的配置文件时才激活Bean。
@Profile 注解实际是通过使用一种更灵活的注解 @Conditional 执行。
@Conditional注解指示一个 org.springframework.context.annotation.Condition ，@Bean在注册进行判断是否启用。
Condition接口的实现提供了matches(…​) 一种返回true或的方法false。

以下示列显示了用于的实现@Profile的Condition：
```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // Read the @Profile annotation attributes
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```

#### !!!Java配置和XML配置结合使用
Spring的@Configuration类支持并不旨在100％完全替代Spring XML。某些工具（例如Spring XML名称空间）仍然是配置容器的理想方法。

可以使用ClassPathXmlApplicationContext，以“以XML为主”的方式实例化容器。

可以使用AnnotationConfigApplicationContext和@ImportResource注解，以“以Java为主”方式 实例化容器。 
	
**以XML为主时使用@Configuration**

将@Configuration类声明为纯bean元素。示例如下：
```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

```
dbc.url = jdbc：hsqldb：hsql：// localhost / xdb
jdbc.username = sa
jdbc.password =
```

```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

使用```context:component-scan```扫描```@Configuration```类。
这种情况下，无需声明 context:annotation-config，因为context:component-scan启用了相同的功能。
system-test-config.xml文件修改后如下：
```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

**以@Configuration类为主的使用XML**
可以@ImportResource注解根据需要使用和定义尽可能多的XML。示例如下：
```
jdbc.properties
jdbc.url = jdbc：hsqldb：hsql：// localhost / xdb
jdbc.username = sa
jdbc.password =
```

```
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```
	
## !!!Environment 环境信息抽象管理
Environment接口是集成在容器中的抽象，可以对应用程序环境的两个关键方面进行建模：profile 和 properties。

profile是仅在给定概要文件处于活动状态时才向容器注册的Bean定义的命名逻辑组。可以将Bean分配给profile，无论是以XML定义还是带有注释。
Environment对象与profile相关的作用是确定当前哪些配置文件（如果有）处于活动状态，以及默认情况下哪些配置文件（如果有）应处于活动状态。

properties在几乎所有应用程序中都起着重要作用，并且可能源自多种来源：属性文件，JVM系统属性，系统环境变量，JNDI，Servlet上下文参数，Properties对象，Map对象等。
Environment对象与properties有关的角色是为用户提供方便的服务界面，用于配置属性源并从中解析属性。

### profile
Bean定义profile在核心容器中提供了一种机制，该机制允许在不同环境中注册不同的Bean。
“环境”一词对不同的用户而言可能意味着不同的含义，并且此功能可以在许多用例中提供帮助，包括：
+ 在开发中针对内存中的数据源进行工作，而不像生产环境那样查找数据源。
+ 仅在将应用程序部署到性能环境中时注册监视基础结构。
+ 为客户A和客户B部署注册bean的自定义实现。

以第一种情况为例，在开发环境使用如下数据源：
```
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```
在生成环境使用如下数据源：
```
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```
现在的问题是如何根据当前环境在使用这两种变体之间进行切换。

#### 使用注解@Profile
@Profile 注解可以指示组件可以注册在一个或多个指定的配置文件。
使用前面的示例，可以修改dataSource配置：
开发环境
```
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```
生产环境
```
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

profile 配置内容可以是profile名称或profile表达式。
profile表达式允许表达更复杂的配置文件逻辑（例如production & us-east），支持以下运算符：
+ !：配置文件的逻辑“不”
+ &：配置文件的逻辑“与”
+ |：配置文件的逻辑“或”

不能在不使用括号的情况下混合使用&and|运算符。例如， production & us-east | eu-central不是有效的表达式。它必须表示为 production & (us-east | eu-central)。

可以将其@Profile用作元注释，以创建自定义的组合注释。
以下示例定义了一个自定义 @Production注解，可以将其用作@Profile("production")的替代品：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

@Profile 也可以在方法级别声明为仅包含配置类的一个特定Bean（例如，特定Bean的替代变体），如以下示例所示：
```
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

#### 使用XML属性profile
XML对应项是beans元素的profile属性。
前面的示例配置可以用两个XML文件重写，示例如下：
```
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```
```
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以在同一文件中拆分和嵌套元素，如以下示例所示：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

#### 定义Profile
配置好了配置项之后，还需要指定Spring激活的Profile。

可以通过多种方式来激活配置文件，最直接的方法是通过Environment以API编程方式进行配置。示例如下：
```
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

可以通过spring.profiles.active属性声明性地激活配置文件。
该属性可以通过系统环境变量，JVM系统属性，servlet上下文参数web.xml等方式配置。
在集成测试中，可以使用@ActiveProfiles注释来声明。

可以一次激活多个配置文件。以下示例激活多个配置文件：
```
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```
或
```
-Dspring.profiles.active =“ profile1，profile2”
```

#### 默认Profile
默认配置文件表示默认情况下启用的配置文件。考虑以下示例：
```
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```
如果没有配置文件处于活动状态，则将dataSource创建。如果启用了任何配置文件，则默认配置文件不适用。

可以通过更改默认的配置文件的名称setDefaultProfiles()上Environment，或者使用spring.profiles.default属性。

### PropertySource
Environment抽象提供了可配置属性源层次结构上的搜索操作。考虑以下清单：
```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```
Environment对象在PropertySource的Set上执行搜索 。
PropertySource是对任何键-值对源的简单抽象。
Spring的StandardEnvironment 配置有两个PropertySource对象：
+ 一个代表JVM系统属性（System.getProperties()）
+ 一个代表系统环境变量（System.getenv()

默认属性源，存在于中StandardEnvironment，供标准应用程序中使用。
其他属性源，比如servlet配置和servlet上下文参数，存在于StandardServletEnvironment。

属性搜索是分层的，默认情况下，系统属性优先于环境变量。
属性值不会合并，而是会被前面的条目完全覆盖。
例如，StandardServletEnvironment，完整层次结构如下，最高优先级条目位于顶部：
+ ServletConfig参数（如果适用，例如在DispatcherServlet上下文的情况下）
+ ServletContext参数（web.xml上下文参数条目）
+ JNDI环境变量（java:comp/env/条目）
+ JVM系统属性（-D命令行参数）
+ JVM系统环境（操作系统环境变量）

### @PropertySource
@PropertySource注解可以添加PropertySource到Spring的Environment。
```
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

### 占位符解析
以往，元素中占位符的值只能根据JVM系统属性或环境变量来解析。  
现在，由于Environment抽象是在整个容器中集成的，因此很容易通过它路由占位符的解析。
这意味着可以按照自己喜欢的任何方式配置解析过程。可以更改搜索系统属性和环境变量的优先级，也可以完全删除它们。
您还可以根据需要将自己的属性源添加到组合中。

具体而言，无论该customer 属性在何处定义，以下语句均有效，只要该属性在Environment可用：
```
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

## LoadTimeWeaver
LoadTimeWeaver类被spring用做动态转换类。
若要启用加载时编织，需要将 @EnableLoadTimeWeaving 添加到一个 @Configuration类。

如以下示例所示：
```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

对于XML配置，可以使用context:load-time-weaver元素：
```
<beans>
    <context:load-time-weaver/>
</beans>
```
一旦针对进行了配置，则其中的ApplicationContext任何bean都ApplicationContext 可以实现LoadTimeWeaverAware，从而接收到对加载时韦弗实例的引用。

与Spring的JPA支持结合使用时，该功能特别有用， 因为对于JPA类转换，可能需要加载时间编织。

## ApplicationContext的其他功能
org.springframework.beans.factory包提供了用于管理和操作Bean的基本功能。

org.springframework.context包的ApplicationContext接口封装扩展了BeanFactory接口，以更面向框架的方式提供附加的功能。

org.springframework.context包还提供以下功能：
+ 通过MessageSource接口访问i18n国际化信息。
+ 通过ResourceLoader接口访问资源，例如URL和文件。
+ 通过ApplicationEventPublisher接口完成事件发布。
+ 加载多个（分层）上下文，每个上下文都通过HierarchicalBeanFactory接口集中在一个特定层上，例如应用程序的Web层。

### 使用MessageSource进行国际化
ApplicationContext接口扩展了MessageSource接口，因此提供了国际化（i18n）功能。
Spring还提供了HierarchicalMessageSource接口，该接口可以分层解析消息。

这些接口一起提供了Spring消息解析的基础。这些接口上定义的方法包括：
+ String getMessage(String code, Object[] args, String default, Locale loc)：用于从中检索消息的基本方法MessageSource。如果找不到指定语言环境的消息，则使用默认消息。使用MessageFormat标准库提供的功能，传入的所有参数都将成为替换值。
+ String getMessage(String code, Object[] args, Locale loc)：与以前的方法基本相同，但有一个区别：不能指定默认消息。如果找不到该消息，则抛出NoSuchMessageException异常。
+ String getMessage(MessageSourceResolvable resolvable, Locale locale)：前述方法中使用的所有属性也都封装在名为MessageSourceResolvable的类中，您可以将其与此方法一起使用。

当ApplicationContext被加载时，它自动搜索在上下文中定义的 MessageSource bean。Bean必须具有名称 messageSource。
如果找到了这样的bean，则将对先前方法的所有调用都委派此消息源。
如果找不到消息源，则ApplicationContext尝试查找包含同名bean的父对象。如果是这样，它将使用该bean作为MessageSource。
如果ApplicationContext找不到消息的任何来源，则将 DelegatingMessageSource实例化为空 ，以便能够接受对上述方法的调用。

Spring提供了两种MessageSource实现，ResourceBundleMessageSource和 StaticMessageSource。
两者都是HierarchicalMessageSource为了执行嵌套消息传递而实现的。
在StaticMessageSource很少使用，但提供编程的方式向消息源添加消息。

以下示例显示ResourceBundleMessageSource：
```
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```
这个例子假设你在类路径定义的三个资源包format，exceptions并windows。
解析消息的任何请求都通过JDK标准的ResourceBundle对象解析消息的方式来处理。

假定上述两个资源束文件的内容如下：
```
#in format.properties 
message=Alligators rock!

#in exceptions.properties
argument.required=The {0} argument is required.
```

下一个示例显示了运行该MessageSource功能的程序：
请记住，所有ApplicationContext实现也是MessageSource 实现，因此可以强制转换为MessageSource接口。
```
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```
以上程序的结果输出如下：
```
Alligators rock!
```

MessageSource 定义在名为beans.xml的文件中，该文件位于类路径的根目录下。
MessageSource bean的basenames属性定义一组资源包，如上示列，basenames属性的列表中传递的三个值，意味着存在于classpath根目录下的三个文件format.properties，exceptions.properties和 windows.properties。

下面示例显示了传递给带参数的Message。这些参数将转换为String对象，并插入到Message中的占位符中：
```
<beans>
    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>
</beans>
```
```
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```
execute()方法调用的结果输出如下：
```
The userDao argument is required.
```

Spring的各种 MessageSource实现 遵循与标准JDK的ResourceBundle相同的**语言环境解析和后备规则**。

如果想对英国（en-GB）语言环境的消息进行解析，需要分别创建名为format_en_GB.properties，exceptions_en_GB.properties 和 windows_en_GB.properties的文件。

通常，语言环境解析由应用程序的周围环境管理。在以下示例中，手动指定了针对其解析（英国）消息的语言环境：
```
#in exceptions_en_GB.properties
argument.required=Ebagum lad, the ''{0}'' argument is required, I say, required.
```

```
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

运行上述程序的结果输出如下：
```
Ebagum lad, the 'userDao' argument is required, I say, required.
```

可以使用 MessageSourceAware接口 访问对已定义的所有MessageSource的引用。

Spring提供了类 ReloadableResourceBundleMessageSource 作为ResourceBundleMessageSource 的替代。此变体支持相同的文件格式，但比基于标准JDK的ResourceBundleMessageSource实现更灵活。尤其是，它允许从任何Spring资源位置（不仅从类路径）读取文件，并支持热加载属性文件（同时在它们之间进行有效缓存）。

### 标准事件和自定义事件
ApplicationContext通过 ApplicationEvent类 和 ApplicationListener接口 提供事件处理。

如果将 实现ApplicationListener接口的Bean 部署到上下文中，则每次ApplicationEvent 被发布到时ApplicationContext时，都会通知该Bean。本质上，这是标准的观察者设计模式。

从Spring 4.2开始，事件基础结构得到了显着改进，并提供了基于注释的模型以及发布任意事件的功能。

Spring提供的标准内置事件：

|事件	|说明	|
|--	|--	|
|ContextRefreshedEvent	|在ApplicationContext初始化或刷新时发布（例如，通过使用接口refresh()上的方法ConfigurableApplicationContext）	|
|ContextStartedEvent	|ApplicationContext使用接口start()上的方法 启动时发布ConfigurableApplicationContext	|
|ContextStoppedEvent	|ApplicationContext使用接口stop()上的方法 停止时发布ConfigurableApplicationContext	|
|ContextClosedEvent	|当发布时间ApplicationContext是由使用封闭close()方法的上ConfigurableApplicationContext接口或经由JVM关闭挂钩	|
|RequestHandledEvent	|一个特定于Web的事件，告诉所有Bean HTTP请求已得到服务	|
|ServletRequestHandledEvent	|该类的子类RequestHandledEvent添加了特定于Servlet的上下文信息	|

可以创建和发布自己的自定义事件。下面的示例显示了一个简单的类，该类扩展了Spring的ApplicationEvent基类：
```
public class BlockedListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlockedListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

发布自定义ApplicationEvent，需要调用 ApplicationEventPublisher中的方法publishEvent()。通过创建一个实现ApplicationEventPublisherAware并注册为Spring bean的类来完成。以下示例显示了此类：
```
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blockedList;
    private ApplicationEventPublisher publisher;

    public void setBlockedList(List<String> blockedList) {
        this.blockedList = blockedList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blockedList.contains(address)) {
            publisher.publishEvent(new BlockedListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

接收自定义ApplicationEvent，可以创建一个实现 ApplicationListener并注册为Spring bean的类。以下示例显示了此类：
```
public class BlockedListNotifier implements ApplicationListener<BlockedListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```
#### 基于注解的事件监听器
从Spring 4.2开始，可以使用@EventListener注释在托管Bean的任何公共方法上注册事件侦听器。BlockedListNotifier可改写如下：
```
public class BlockedListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlockedListEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```
方法签名（BlockedListEvent event）声明其侦听的事件类型。

如果想要侦听多个事件，或者完全不使用任何参数来定义它，则事件类型也可以在注解本身上指定。以下示例显示了如何执行此操作：
```
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

可以通过使用condition定义SpEL表达式的注释的属性来添加其他运行时过滤，该表达式应匹配以针对特定事件实际调用该方法。
以下示例说明了仅当content事件的属性等于时，才可以重写我们的通知程序以进行调用 my-event：
```
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlockedListEvent(BlockedListEvent blockedListEvent) {
    // notify appropriate parties via notificationAddress...
}
```

#### 异步监听器
如果希望特定的侦听器异步处理事件，可以使用@Async支持。以下示例显示了如何执行此操作：
```
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent is processed in a separate thread
}
```

使用异步事件时，请注意以下限制：
+ 如果异步事件侦听器抛出Exception，不会传播到调用者。
+ 异步事件侦听器方法无法通过返回值来发布后续事件。如果您需要发布另一个事件作为处理的结果，请插入一个 ApplicationEventPublisher 以手动发布事件。

#### 有序调用监听器
如果需要在某一个监听器之前先调用另一个监听器，则可以将@Order注解添加到方法声明中，如以下示例所示：
```
@EventListener
@Order(42)
public void processBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

#### 一般事件
可以使用泛型来进一步定义事件的结构。参考使用 EntityCreatedEvent<T> 创建的实际实体的类型。
例如，您可以创建以下侦听器定义只接收EntityCreatedEvent<Person>：
```
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

### 便捷地访问低级资源
为了最佳使用和理解应用程序上下文，应该熟悉Spring的Resource抽象。

应用程序上下文（application context）是一个 ResourceLoader，可用于加载Resource对象。
Resource本质上是JDKjava.net.URL类的一个功能更丰富的版本。实质上，Resource封装了一个java.net.URL的实例。

Resource几乎可以从任何位置获取低级资源，包括类路径，文件系统位置，可使用标准URL描述的任何位置以及一些其他变体。
如果资源位置字符串是没有任何特殊前缀的简单路径，则这些资源的来源是特定的，并且适合于实际的应用程序上下文类型。

可以配置部署到应用程序上下文中的Bean，实现特殊的回调接口ResourceLoaderAware，以便在初始化时使用作为传入的应用程序上下文本身自动进行回调ResourceLoader。
可以公开Resource的type属性，用于访问静态资源。它们像其他任何属性一样注入其中。
可以将这些Resource属性指定为简单String路径，并在Resource部署bean时依靠从这些文本字符串到实际对象的自动转换。

提供给ApplicationContext构造函数的一个或多个位置路径实际上是资源字符串，并且根据特定的上下文实现以简单的形式对其进行了适当处理。例如，ClassPathXmlApplicationContext将简单的位置路径视为类路径位置。
也可以使用带有特殊前缀的位置路径（资源字符串）来强制从类路径或URL中加载定义，而不管实际的上下文类型如何。

### 应用程序运行监控
ApplicationContext管理着Spring应用程序的生命周期，并围绕组件提供丰富的编程模型，复杂的应用程序可能具有同样复杂的组件图和启动阶段。

使用特定的指标跟踪应用程序的启动步骤可以帮助了解启动阶段所花的时间，也可以用作更好地了解整个上下文生命周期的一种方式。

AbstractApplicationContext（及其子类）使用 ApplicationStartup 进行可视化运行期监控。
它收集StartupStep有关各种启动阶段数据：
+ 应用程序上下文生命周期（基本软件包扫描，配置类管理）
+ bean生命周期（实例化，智能初始化，后处理）
+ 应用程序事件处理

AnnotationConfigApplicationContext相关示列：
```
// create a startup step and start recording
StartupStep scanPackages = this.getApplicationStartup().start("spring.context.base-packages.scan");
// add tagging information to the current step
scanPackages.tag("packages", () -> Arrays.toString(basePackages));
// perform the actual phase we're instrumenting
this.scanner.scan(basePackages);
// end the current step
scanPackages.end();
```

应用程序上下文已通过多个步骤进行了检测。记录后，可以使用特定工具收集，显示和分析这些启动步骤

### Web应用程序中的便捷ApplicationContext实例化
可以使用 ContextLoader 以声明方式创建 ApplicationContext实例 。
可以使用 ApplicationContext实现类 以编程方式创建 ApplicationContext实例。

可以使用ContextLoaderListener来注册一个ApplicationContext，如以下示例所示：
```
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>
```

监听器 ContextLoaderListener 检查contextConfigLocation参数。
+ 如果参数不存在，那么监听器将使用 /WEB-INF/applicationContext.xml 作为默认值。
+ 当参数确实存在时，监听器将使用预定义的分隔符（逗号，分号和空格）将参数值分隔，并将这些值用作搜索应用程序上下文的位置。

还支持蚂蚁风格的路径模式。例如：
/WEB-INF/*Context.xml，位于WEB-INF目录下且名称以Context.xml结尾。
/WEB-INF/**/*Context.xml， 位于WEB-INF任何子目录下且名称以Context.xml结尾。

### 将 Spring应用程序 部署为 JavaEE的RAR文件
可以将SpringApplicationContext 部署为RAR文件，将上下文及其所有必需的Bean类和库JAR封装在Java EE RAR部署单元中。
这等效于引导ApplicationContext能够访问Java EE服务器功能的独立服务器（仅托管在Java EE环境中）。
对于部署无头WAR文件的情况，RAR部署是一种更自然的选择。实际上，这种WAR文件没有任何HTTP入口点，仅用于ApplicationContext在Java EE环境中引导Spring 。

对于不需要HTTP入口点而仅由消息端点和计划作业组成的应用程序上下文，RAR部署是理想的选择。
在这样的上下文中，Bean可以使用应用程序服务器资源，例如JTA事务管理器和绑定到JNDI的JDBC DataSource实例和JMSConnectionFactory实例，还可以通过平台的JMX服务器注册-全部通过Spring的标准事务管理以及JNDI和JMX支持工具。
应用程序组件还可以WorkManager通过Spring的TaskExecutor抽象与应用程序服务器的JCA进行交互。

将Spring ApplicationContext作为Java EE RAR文件进行简单部署：
1. 将所有应用程序类打包到RAR文件（这是具有不同文件扩展名的标准JAR文件）中。将所有必需的库JAR添加到RAR归档文件的根目录中。添加 META-INF/ra.xml部署描述符（如javadoc中的SpringContextResourceAdapter所示）和相应的Spring XML bean定义文件（通常为 META-INF/applicationContext.xml）。
2. 将生成的RAR文件拖放到应用程序服务器的部署目录中。

此类RAR部署单元通常是独立的。
它们不会将组件暴露给外界，甚至不会暴露给同一应用程序的其他模块。
与基于RAR的交互ApplicationContext通常是通过与其他模块共享的JMS目标进行的。
基于RAR的文件ApplicationContext还可以例如安排一些作业或对文件系统（或类似文件）中的新文件做出反应。
如果需要允许来自外部的同步访问，则可以（例如）导出RMI端点，该端点可以由同一台计算机上的其他应用程序模块使用。

## BeanFactory
BeanFactoryAPI为Spring的IoC功能提供了基础。

它的具体功能是主要用于与Spring的其他部分以及相关的第三方框架集成，并且它的DefaultListableBeanFactory实现是高层GenericApplicationContext容器中的关键委托。

BeanFactory及其相关接口（如BeanFactoryAware，InitializingBean， DisposableBean）是对于其他框架组件的重要结合点。
不通过任何注释甚至是反射，它们允许容器及其组件之间非常有效的交互。
应用程序级Bean可以使用相同的回调接口，但通常更喜欢通过注释或通过程序配置进行声明式依赖注入。

请注意，核心BeanFactoryAPI级别及其DefaultListableBeanFactory 实现不对配置格式或要使用的任何组件注释进行假设。
所有这些风格都通过扩展名（例如XmlBeanDefinitionReader和AutowiredAnnotationBeanPostProcessor）进入，并在共享BeanDefinition对象上作为核心元数据表示形式进行操作。
这就是使Spring的容器如此灵活和可扩展的本质。

### BeanFactory 或 ApplicationContext？
BeanFactory和 ApplicationContext容器级别之间的区别以及对引导的影响。

除非有充分的理由，否则应使用ApplicationContext。
GenericApplicationContext其及其子类AnnotationConfigApplicationContext 这些是用于所有常见目的的Spring核心容器的主要入口点：加载配置文件，触发类路径扫描，以编程方式注册Bean定义和带注释的类，以及（从5.0版本开始）注册功能性Bean定义。

下表列出了BeanFactory 和 ApplicationContext接口和实现所提供的功能：

|特性	|BeanFactory	|ApplicationContext	|
|--	|--	|--	|
|Bean实例化	|有	|有	|
|生命周期管理	|没有	|有	|
|自动BeanPostProcessor注册	|没有	|有	|
|自动BeanFactoryPostProcessor注册	|没有	|有	|
|便捷MessageSource访问（用于内部化）	|没有	|有	|
|内置ApplicationEvent发布机制	|没有	|有	|

# 资源
## 介绍
Java的java.net.URL和用于各种URL前缀的标准类和标准处理程序不足以对所有低级资源进行访问。

没有标准化的URL实现 可用于访问需要从类路径或相对于类路径获取的资源 ServletContext。

尽管可以为专用URL 前缀注册新的处理程序（类似于诸如这样的前缀的现有处理程序http:），但这通常相当复杂，并且URL接口仍然缺少某些理想的功能，例如一种检查资源是否存在的方法。

## Resource接口
Spring的Resource接口旨在成为一种功能更强大的接口，用于抽象化对低级资源的访问。
以下清单显示了Resource接口定义：
```
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();
}
```
它扩展了InputStreamSource 接口。
以下清单显示了InputStreamSource 接口的定义：
```
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;
}
```

该Resource界面中一些最重要的方法是：
+ getInputStream()：找到并打开资源，并返回一个资源以InputStream供读取。预计每次调用都会返回一个新的 InputStream。调用者有责任关闭流。
+ exists()：返回boolean，指示此资源是否实际以物理形式存在。
+ isOpen()：返回boolean，指示此资源是否表示具有打开流的句柄。如果为true，InputStream则不能多次读取，必须只读取一次，然后将其关闭以避免资源泄漏。返回false所有常用资源实现（除外）InputStreamResource。
+ getDescription()：返回对此资源的描述，以便在使用该资源时用于错误输出。这通常是标准文件名或资源的实际URL。

## 内置Resource实现
+ UrlResource
+ ClassPathResource
+ FileSystemResource
+ ServletContextResource
+ InputStreamResource
+ ByteArrayResource

### UrlResource
UrlResource封装了一个java.net.URL，通常可用于访问通过URL访问的任何对象，例如文件，HTTP目标，FTP目标等。

所有URL都有一个标准化的String表示形式，因此，使用适当的标准化前缀来表示另一种URL类型。
file:文件系统路径，http:HTTP协议，ftp:通过FTP访问资源等。

UrlResource是由Java代码通过显式使用UrlResource构造函数创建的，但通常在调用带有String 用于表示路径的参数的API方法时隐式创建。对于后一种情况，JavaBeans PropertyEditor最终决定Resource创建哪种类型。如果路径字符串包含众所周知的前缀（例如classpath:），则它将Resource为该前缀创建一个适当的专门名称。但是，如果它不能识别前缀，则假定该字符串是标准URL字符串并创建一个UrlResource。

### ClassPathResource
ClassPathResource 从类路径获取资源。它使用线程上下文类加载器、给定的类加载器或给定的类 来加载资源。

此Resource实现支持解析，就java.io.File好像类路径资源驻留在文件系统中一样，而不支持类路径资源驻留在jar中并且尚未（通过servlet引擎或任何环境被扩展）到文件系统中。为了解决这个问题，各种Resource实现始终支持将分辨率作为java.net.URL。

ClassPathResource是由Java代码通过显式使用ClassPathResource 构造函数创建的，但通常在调用带有String用于表示路径的参数的API方法时隐式创建 。对于后一种情况，JavaBeans会 在字符串路径上PropertyEditor识别特殊前缀，classpath:并ClassPathResource在这种情况下创建一个。

### FileSystemResource
FileSystemResource 是为了实现java.io.File和java.nio.file.Path。它支持解析File和URL。

### ServletContextResource
这是资源的Resource实现，该ServletContext资源解释相关Web应用程序根目录中的相对路径。

它始终支持流访问和URL访问，但java.io.File仅在扩展Web应用程序档案且资源实际位于文件系统上时才允许访问。它是在文件系统上扩展还是直接扩展，或者是直接从JAR或其他类似数据库（可以想到的）中访问，实际上取决于Servlet容器。

### InputStreamResource
一个InputStreamResource是Resource给定的实现InputStream。仅当没有特定的Resource实现适用时才应使用它。特别地，在可能ByteArrayResource的Resource情况下，首选 或任何基于文件的实现。

与其他Resource实现相反，这是一个已经打开的资源的描述符。因此，它true从返回isOpen()。如果您需要将资源描述符保留在某个地方，或者需要多次读取流，请不要使用它。

### ByteArrayResource
这是Resource给定字节数组的实现。它ByteArrayInputStream为给定的字节数组创建一个 。

这对于从任何给定的字节数组中加载内容很有用，而无需使用一次使用InputStreamResource。

## ResourceLoader
ResourceLoader接口由可以返回 Resource实例的对象实现。
以下清单显示了ResourceLoader 接口定义：
```
public interface ResourceLoader {

    Resource getResource(String location);
}
```

所有应用程序上下文均实现该ResourceLoader接口。因此，所有应用程序上下文都可用于获取Resource实例。
当您调用getResource()特定的应用程序上下文时，并且指定的位置路径没有特定的前缀时，您将获得Resource适合该特定应用程序上下文的类型。
例如，假定针对ClassPathXmlApplicationContext实例运行以下代码段：
```
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```
针对ClassPathXmlApplicationContext，该代码返回ClassPathResource。
如果对FileSystemXmlApplicationContext实例运行相同的方法，它将返回 FileSystemResource。
对于WebApplicationContext，它将返回 ServletContextResource。
类似地，它将为每个上下文返回适当的对象。

另一方面，无论应用程序上下文类型如何，都可以通过指定特殊classpath:前缀来强制使用ClassPathResource，如以下示例所示：
```
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

同样，可以通过指定任何标准java.net.URL前缀来强制使用UrlResource。以下两个示例使用file和http：
```
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```
```
Resource template = ctx.getResource("https://myhost.com/resource/path/myTemplate.txt");
```

## ResourceLoaderAware接口
ResourceLoaderAware接口是一个特殊的回调接口，用于标识期望提供ResourceLoader引用的组件。
以下清单显示了ResourceLoaderAware接口的定义：
```
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```
当一个类实现ResourceLoaderAware并部署到应用程序上下文中（作为Spring托管的bean）时，该类被应用程序上下文识别为ResourceLoaderAware。然后setResourceLoader(ResourceLoader)，应用程序上下文调用，将自身作为参数提供（请记住，Spring中的所有应用程序上下文都实现了该ResourceLoader接口）。

由于ApplicationContext也是一个ResourceLoader，因此bean也可以实现 ApplicationContextAware接口并直接使用提供的应用程序上下文来加载资源。但是，通常需要的话，最好使用专用接口ResourceLoader。该代码将仅耦合到资源加载接口（可以视为实用程序接口），而不耦合到整个Spring ApplicationContext接口。

## Resources 作为依赖
如果Bean本身是通过某种动态过程来确定和提供资源路径，那么对于Bean来说，使用该ResourceLoader 接口来加载资源可能是有意义的。例如，考虑加载某种模板，其中所需的特定资源取决于用户的角色。如果资源是静态的，则有必要完全消除对ResourceLoader 接口的使用，让Bean公开所需的Resource属性，并期望将其注入其中。

注入这些属性很简单，因为所有应用程序上下文都注册并使用了特殊的JavaBeans PropertyEditor，它们可以将String路径转换为Resource对象。因此，如果myBean的template属性的type为Resource，则可以使用该资源的简单字符串对其进行配置，如以下示例所示：
```
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

请注意，资源路径没有前缀，ResourceLoader 将根据应用程序上下文的类型判断加载 ClassPathResource，aFileSystemResource 或 ServletContextResource。

如果需要强制使用特定Resource类型，则可以使用前缀。
以下两个示例显示了如何强制aClassPathResource和a UrlResource（后者用于访问文件系统文件）：
```
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

## 应用程序上下文和资源路径
### 构造应用程序上下文
应用程序上下文构造函数（针对特定的应用程序上下文类型）通常采用字符串或字符串数​​组作为资源的位置路径，例如构成上下文定义的XML文件。

当这样的位置路径没有前缀时，Resource从该路径构建并用于加载Bean定义的特定类型取决于特定应用程序上下文，并且适用于该特定应用程序上下文。

如下示例创建一个 ClassPathXmlApplicationContext：
```
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```
这里的Bean定义是从类路径加载的。

如下示例创建一个FileSystemXmlApplicationContext：
```
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/appContext.xml");
```
这里的bean定义是从文件系统位置（在这种情况下，是相对于当前工作目录）加载的。

#### 快捷构造ClassPathXmlApplicationContext实例
ClassPathXmlApplicationContext 提供了多种构造方法以便于实例。

基本思想是，您只能提供一个字符串数组，该字符串数组仅包含XML文件本身的文件名（不包含前导路径信息），并且还提供一个Class。所述ClassPathXmlApplicationContext 然后导出从提供类的路径信息。

请考虑以下目录布局：
```
com /
  foo /
    services.xml
    daos.xml
    MessengerService.class
```

下面的示例示出了如何ClassPathXmlApplicationContext豆组成的实例中命名的文件中定义services.xml和daos.xml（其是在类路径）可以被实例化：
```
ApplicationContext ctx = new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```
	
### 应用程序上下文构造函数中 资源路径的通配符
应用程序上下文构造函数值中的资源路径可以是简单路径（如前所述），每个路径都具有到目标的一对一映射Resource，或者可以包含特殊的“ classpath *：”前缀或内部Ant-样式正则表达式（使用Spring的PathMatcher实用程序进行匹配）。后者都是有效的通配符。

这种机制的一种用途是当您需要进行组件样式的应用程序组装时。所有组件都可以将上下文定义片段“发布”到一个公共的位置路径，并且当使用前缀为的相同路径创建最终应用程序上下文时 classpath*:，所有组件片段都将自动被拾取。

请注意，此通配符特定于在应用程序上下文构造函数中使用资源路径（或当您PathMatcher直接使用实用程序类层次结构时），并在构造时解决。它与Resource类型本身无关。您不能使用classpath*:前缀构造一个real Resource，因为资源一次仅指向一个资源。

#### Ant-style
路径位置可以包含Ant样式的模式，如以下示例所示：
```
/WEB-INF/*-context.xml
com / mycompany / ** / applicationContext.xml
文件：C：/ some / path / *-context.xml
classpath：com / mycompany / ** / applicationContext.xml
```

当路径位置包含Ant样式的模式时，解析程序将遵循更复杂的过程来尝试解析通配符。它Resource为到达最后一个非通配符段的路径生成一个，并从中获取一个URL。如果此URL不是jar:URL或特定zip:于容器的变体（例如WebLogic，wsjarWebSphere等），java.io.File则从中获取a并通过遍历文件系统来解析通配符。对于jar URL，解析器可以从中获取ajava.net.JarURLConnection或手动解析jar URL，然后遍历jar文件的内容以解析通配符。

#### classpath*:
构造基于XML的应用程序上下文时，位置字符串可以使用特殊classpath*:前缀，如以下示例所示：
```
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
```

这个特殊的前缀指定必须获取与给定名称匹配的所有类路径资源（内部，这实际上是通过调用来实现 ClassLoader.getResources(…​)），然后合并以形成最终的应用程序上下文定义。

#### 有关通配符的其他说明
请注意，classpath*:与Ant样式的模式结合使用时，除非模式文件实际存在于目标文件中，否则在模式启动之前，它至少必须与至少一个根目录可靠地配合使用。这意味着诸如之类的模式 classpath*:*.xml可能不会从jar文件的根目录检索文件，而只会从扩展目录的根目录检索文件。

Spring检索类路径条目

### FileSystemResource警告
一个FileSystemResource未连接到FileSystemApplicationContext 绝对路径和相对路径。
相对路径是相对于当前工作目录的，而绝对路径是相对于文件系统的根的。

但是，出于向后兼容（历史）的原因，当FileSystemApplicationContext是时，情况会发生变化 ResourceLoader。该 FileSystemApplicationContext部队所有连接的FileSystemResource情况下，把所有的位置路径为相对的，他们是否具有领先的斜线或无法启动。实际上，这意味着以下示例是等效的：
```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/context.xml");
```
```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("/conf/context.xml");

```

# 验证，数据绑定和类型转换
验证作为业务逻辑是有利有弊的。
验证不应与Web层绑定，并且应该易于本地化，并且应该可以插入任何可用的验证器。
考虑到这些问题，Spring提供了一个Validator规范，该规范可以在应用程序的每个层中使用。

数据绑定对于使用户输入动态绑定到应用程序的域模型非常有用。
Spring提供了恰如其分的命名DataBinder来做到这一点。
Validator 和 DataBinder 组成了validation包，它主要用于但不限于 web 层。

BeanWrapper是Spring框架中的基本概念，并在很多地方被使用。
不需要直接使用BeanWrapper，在尝试将数据绑定到对象时最有可能使用它。

Spring的DataBinder和较低级别的BeanWrapper都使用 PropertyEditorSupport实现来解析和格式化属性值。

Spring通过安装基础结构和Spring自己的Validator的适配器来支持Java Bean验证。
应用程序可以全局启用一次Bean验证，并将其专门用于所有验证需求。
在Web层中，应用程序为每个DataBinder注册本地的Spring Validator实例，这对于插入自定义验证逻辑很有用。

## 使用Validator接口进行验证
Spring有一个可用于验证对象的 Validator 接口。
Validator接口通过使用Errors对象来工作，以便在验证时，验证器可以向该Errors对象报告验证失败。

一个小型数据对象示例Person：
```
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```

Person通过实现org.springframework.validation.Validator接口的以下两种方法来提供类的验证行为：
+ supports(Class)：Validator是否支持验证所提供类实例
+ validate(Object, org.springframework.validation.Errors)：验证给定的对象，并在验证错误的情况下，向给定的Errors对象注册那些对象。

实现Validator非常简单，Spring提供了helper类 ValidationUtils。
以下示例Validator为Person实例实现：
```
public class PersonValidator implements Validator {

    /**
     * This Validator validates only Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```
ValidationUtils类的rejectIfEmpty(..)方法用于拒绝name属性，当它是null或空字符串。

虽然可以实现单个Validator类来验证丰富对象中的每个嵌套对象，但最好将对象的每个嵌套类的验证逻辑封装在自己的Validator实现中。

一个“丰富”对象的简单示例是一个Customer由两个String 属性（第一个和第二个名称）和一个复杂Address对象组成的对象。Address对象可以独立于Customer对象使用，因此AddressValidator 已实现了不同的对象。如果要CustomerValidator重用AddressValidator该类中包含的逻辑而不求助于复制和粘贴，则可以AddressValidator在您的中依赖注入或实例化一个CustomerValidator，如以下示例所示：
```
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * This Validator validates Customer instances, and any subclasses of Customer too
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```

## 将代码解析为错误消息
在上一节中显示的示例中，我们拒绝了name和age字段。

如果我们想使用MessageSource来输出错误消息 ，我们可以使用拒绝字段时提供的错误代码（'name'和'age'）来完成。

当从接口调用（直接或间接地，例如通过使用ValidationUtils类）Errors的rejectValue或其他reject方法时，基础实现不仅会注册您传入的代码，还会注册许多其他错误代码。

MessageCodesResolver 确定Errors接口注册哪些错误代码。默认情况下，使用DefaultMessageCodesResolver不仅使用您提供的代码注册一条消息，而且还注册包含您传递给reject方法的字段名称的消息。因此，如果使用拒绝字段rejectValue("age", "too.darn.old")，则除了too.darn.old代码外，Spring还将注册too.darn.old.age和 too.darn.old.age.int（第一个包含字段名称，第二个包含字段类型）。

## Bean操作和BeanWrapper

## 弹簧类型转换

## Spring格式转换

## 配置全局日期和时间格式

## JavaBean验证

# Spring表达式（SpEL）
Spring表达式语言（简称“ SpEL”）是一种功能强大的表达式语言，支持在运行时查询和操作对象图。
语言语法类似于Unified EL，但提供了其他功能，最著名的是方法调用和字符串模板功能。

虽然SpEL是Spring产品组合中表达评估的基础，但它并不直接与Spring绑定，而是可以独立使用。
为了自成一体，本章中的许多示例都将SpEL用作独立的表达语言。这需要创建一些自举基础结构类，例如解析器。大多数Spring用户不需要处理此基础结构，而只能编写表达式字符串进行评估。这种典型用法的一个示例是将SpEL集成到创建XML或基于注释的Bean定义中，如Expression对定义Bean定义的支持所示 。

本章介绍了表达语言，其API和语言语法的功能。在几个地方，Inventor和Society类为目标的表达评价对象使用。这些类声明和用于填充它们的数据在本章末尾列出。

表达式语言支持以下功能：
+ 文字表达
+ 布尔运算符和关系运算符
+ 常用表达
+ 类表达式
+ 访问属性，数组，列表和映射
+ 方法调用
+ 关系运算符
+ 分配
+ 调用构造函数
+ Bean参考
+ 阵列构造
+ 内联列表
+ 内联地图
+ 三元运算符
+ 变数
+ 用户定义的功能
+ 集合投影
+ 馆藏选择
+ 模板表达式

## 解析
使用SpEL 解析字符串表达式：
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'"); 
String message = (String) exp.getValue();
```
message变量的值为"Hello World"。

使用SpEL 调用String的concat方法：
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')"); 
String message = (String) exp.getValue();
```
现在的值message是"Hello World！"。

使用SpEL 调用String的bytes属性：
```
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes"); 
byte[] bytes = (byte[]) exp.getValue();
```

示例如何从Inventor类的实例检索name属性或创建布尔条件：
```
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// The constructor arguments are name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name"); // Parse name as an expression
String name = (String) exp.getValue(tesla);
// name == "Nikola Tesla"

exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(tesla, Boolean.class);
// result == true
```

#### 理解EvaluationContext
EvaluationContext接口用作评估表达式以解析属性、方法或字段，并帮助执行类型转换时。

Spring提供了两种实现。
+ SimpleEvaluationContext：针对不需要全部SpEL语言语法范围且应受到有意义限制的表达式类别，公开了SpEL基本语言功能和配置选项的子集。示例包括但不限于数据绑定表达式和基于属性的过滤器。
+ StandardEvaluationContext：公开全部SpEL语言功能和配置选项。您可以使用它来指定默认的根对象，并配置每个可用的评估相关策略。

SimpleEvaluationContext设计为仅支持SpEL语言语法的子集。它不包括Java类型引用，构造函数和Bean引用。它还要求您明确选择对表达式中的属性和方法的支持级别。默认情况下，create()静态工厂方法仅启用对属性的读取访问。

您还可以获取构建器来配置所需的确切支持级别，并针对以下一种或某些组合：
+ PropertyAccessor仅自定义（无反射）
+ 只读访问的数据绑定属性
+ 读写的数据绑定属性

#### 解析器配置
可以通过使用解析器配置（org.springframework.expression.spel.SpelParserConfiguration）对象来配置SpEL表达式解析器。

当使用由一系列属性引用组成的表达式时，这很有用。下面的示例演示如何自动增加列表：
```
class Demo {
    public List<String> list;
}

// Turn on:
// - auto null reference initialization
// - auto collection growing
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list will now be a real collection of 4 entries
// Each entry is a new empty String
```

#### SpEL编译
Spring包含一个基本的表达式编译器,通常对表达式进行解释，这样可以在评估过程中提供很大的动态灵活性，但不能提供最佳性能。
对于偶尔使用表达式还好，但是当与其他组件（例如Spring Integration）一起使用并且不需要动态性时，性能可能非常重要。

SpEL编译器旨在满足这一需求。在评估期间，编译器会生成一个Java类，该类体现运行时的表达式行为，并使用该类来实现更快的表达式评估。
由于缺少在表达式周围输入内容的信息，因此编译器在执行编译时会使用在表达式的解释求值过程中收集的信息。
例如，它不仅仅从表达式中就知道属性引用的类型，而是在第一次解释求值时就知道它是什么。
当然，如果各种表达元素的类型随时间变化，则基于此类派生信息进行编译会在以后引起麻烦。
因此，编译最适合类型信息在重复求值时不会改变的表达式。

考虑以下基本表达式：
```
someArray [0] .someProperty.someOtherProperty <0.1
```
因为前面的表达式涉及数组访问，一些属性取消引用和数字运算，所以性能提升可能非常明显。
在一个示例中，进行了50000次迭代的微基准测试，使用解释器评估需要75毫秒，而使用表达式的编译版本仅需要3毫秒。


**编译器配置**

默认情况下不打开编译器，但是您可以通过两种不同的方式之一来打开它。
当SpEL用法嵌入到另一个组件中时，可以使用解析器配置过程（如前所述）或使用系统属性来打开它。

编译器可以在org.springframework.expression.spel.SpelCompilerMode枚举中捕获的三种模式之一中运行 。模式如下：
+ OFF （默认）：编译器已关闭。
+ IMMEDIATE：在立即模式下，将尽快编译表达式。通常是在第一次解释评估之后。如果编译后的表达式失败（通常是由于类型更改，如前所述），则表达式求值的调用者将收到异常。
+ MIXED：在混合模式下，表达式会随着时间静默在解释模式和编译模式之间切换。经过一定数量的解释运行后，它们会切换到编译形式，如果编译形式出了问题（例如，如前面所述的类型更改），则表达式会自动再次切换回解释形式。稍后，它可能会生成另一个已编译的表单并切换到该表单。基本上，用户进入IMMEDIATE模式的异常是在内部处理的。

选择模式后，使用SpelParserConfiguration来配置解析器。以下示例显示了如何执行此操作：
```
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```

当指定编译器模式时，还可以指定一个类加载器（允许传递null）。
编译的表达式是在提供的任何子类加载器中定义的。重要的是要确保，如果指定了类加载器，则它可以查看表达式评估过程中涉及的所有类型。
如果未指定类加载器，则使用默认的类加载器（通常是在表达式求值期间运行的线程的上下文类加载器）。

第二种配置编译器的方法是将SpEL嵌入到其他组件中，并且可能无法通过配置对象进行配置。
在这些情况下，可以使用系统属性。您可以设置spring.expression.compiler.mode 属性为一个SpelCompilerMode枚举值（off，immediate，或mixed）。

**编译器限制**

从Spring Framework 4.1开始，已经有了基本的编译框架。但是，该框架尚不支持编译每种表达式。最初的重点是可能在性能关键型上下文中使用的通用表达式。目前无法编译以下类型的表达式：
+ 涉及赋值的表达
+ 表达式依赖转换服务
+ 使用自定义解析器或访问器的表达式
+ 使用选择或投影的表达式
+ 将来会编译更多类型的表达。

## Bean中定义的表达式
您可以将SpEL表达式与基于XML或基于注释的配置元数据一起使用来定义BeanDefinition实例。
在这两种情况下，定义表达式的语法均为形式#{ <expression string> }。

### XML配置
可以使用表达式来设置属性或构造函数的参数值，如以下示例所示：
```
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
</bean>
```

应用程序上下文中的所有bean都可以作为具有其常规bean名称的预定义变量使用。
这包括用于访问运行时环境的标准上下文Bean，
例如environment（类型为 org.springframework.core.env.Environment）
以及systemProperties和 systemEnvironment（类型为Map<String, Object>）。

下面的示例显示作为SpEL对systemProperties变量的bean的访问：
```
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
    <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>
</bean>
```

还可以按名称引用其他bean属性，如以下示例所示：
```
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>

<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>

    <!-- other properties -->
</bean>
```

### 注释配置
若要指定默认值，可以将@Value注释放在字段，方法以及构造函数参数上。

下面的示例设置字段变量的默认值：
```
public class FieldValueTestBean {

    @Value("#{ systemProperties['user.region'] }")
    private String defaultLocale;

    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }
}
```

下面的示例显示等效的使用setter方法的示例：
```
public class PropertyValueTestBean {

    private String defaultLocale;

    @Value("#{ systemProperties['user.region'] }")
    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }
}
```

自动装配的方法和构造函数也可以使用@Value注释，如以下示例所示：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;
    private String defaultLocale;

    @Autowired
    public void configure(MovieFinder movieFinder,
            @Value("#{ systemProperties['user.region'] }") String defaultLocale) {
        this.movieFinder = movieFinder;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

```
public class MovieRecommender {

    private String defaultLocale;

    private CustomerPreferenceDao customerPreferenceDao;

    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao,
            @Value("#{systemProperties['user.country']}") String defaultLocale) {
        this.customerPreferenceDao = customerPreferenceDao;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

## 语言参考
Spring Expression Language的工作方式包含以下方式。

### 文字表达
支持的文字表达式的类型为字符串，数值（int，实数，十六进制），布尔值和null。
字符串用单引号引起来。要将单引号本身放在字符串中，请使用两个单引号字符。

以下清单显示了文字的简单用法：
```
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```
通常，它们不是像这样孤立地使用，而是作为更复杂的表达式的一部分使用-例如，在逻辑比较运算符的一侧使用文字。

数字支持使用负号，指数符号和小数点。默认情况下，使用Double.parseDouble() 解析实数。

### 属性，数组，列表，映射和索引器
**属性值**使用点 "." 来获取。
以下示列向下导航并获取特斯拉的出生年份和普平的出生城市，使用以下表达式：
```
// evals to 1856
int year = (Integer) parser.parseExpression("birthdate.year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.city").getValue(context);
```

**数组和列表**的内容通过使用方括号 "[]" 表示法获得，如以下示例所示：
```
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// Inventions Array

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(context, tesla, String.class);

// Members List

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("members[0].name").getValue(context, ieee, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("members[0].inventions[6]").getValue(context, ieee, String.class);
```

通过在方括号内指定文字键值可以获取映射的内容。
在下面的示例中，由于officers映射的键是字符串，因此我们可以指定字符串文字：
```
// Officer's Dictionary

Inventor pupin = parser.parseExpression("officers['president']").getValue(societyContext, Inventor.class);

// evaluates to "Idvor"
String city = parser.parseExpression("officers['president'].placeOfBirth.city").getValue(societyContext, String.class);

// setting values
parser.parseExpression("officers['advisors'][0].placeOfBirth.country").setValue(societyContext, "Croatia");
```

### 内联Lists
可以使用{}符号直接在表达式中表达List：
```
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```
{}本身意味着一个空列表。
出于性能原因，如果列表本身完全由固定的文字组成，则会创建一个常量列表来表示该表达式（而不是在每次求值时都建立一个新列表）。

### 内联Maps
可以使用{key:value}符号直接在表达式中表达Map：
```
// evaluates to a Java map containing the two entries
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```
{:}本身意味着一个空的地图。
出于性能原因，如果映射图本身由固定的文字或其他嵌套的常量结构（列表或映射图）组成，则会创建一个常量映射图来表示该表达式（而不是在每次求值时都构建一个新的映射图）。
映射键的引用是可选的。上面的示例不使用带引号的键。

### 数组构造
您可以使用熟悉的Java语法来构建数组，可以选择提供一个初始化程序，以在构造时填充该数组。

以下示例显示了如何执行此操作：
```
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

### 方法
### 经营者
### 种类
### 建设者
### 变数
### 职能
### Bean参考
### 三元运算符（If-Then-Else）
### 猫王算子
### 安全导航操作员
### 馆藏选择
### 集合投影
### 表达式模板

## 示例中使用的类

# 使用Spring进行面向切面编程
面向切面编程（AOP）通过提供另一种思考程序结构的方式来补充面向对象的编程（OOP）。

OOP中模块化的关键单元是类，而AOP中模块化的关键单元是切面。切面支持跨多种类型和对象的关注点（例如事务管理）的模块化。

Spring的关键组件之一是AOP框架。尽管Spring IoC容器不依赖于AOP，但是AOP对Spring IoC进行了补充，以提供功能强大的中间件解决方案。

AOP在Spring框架中用于：
+ 提供声明式企业服务。最重要的服务是 声明式事务管理。
+ 实现自定义方切面，以AOP补充其对OOP的使用。

## AOP概念
让我们首先定义一些主要的AOP概念和术语，这些术语不是特定于Spring的：
+ 方面（Aspect）：涉及多个类别的关注点的模块化。事务管理是企业Java应用程序中横切关注的一个很好的例子。在Spring AOP中，方面是通过使用常规类（基于模式的方法）或使用注释进行@Aspect注释的常规类 （@AspectJ样式）来实现的。
+ 连接点（Join point:）：在程序执行过程中的一点，例如方法的执行或异常的处理。在Spring AOP中，连接点始终代表方法的执行。
+ 增强（Advice）：方面在特定的连接点处采取的操作。不同类型的建议包括“周围”，“之前”和“之后”建议。（建议类型将在后面讨论。）包括Spring在内的许多AOP框架都将建议建模为拦截器，并在连接点周围维护一系列拦截器。
+ 切入点（Pointcut）：与连接点匹配的谓词。建议与切入点表达式关联，并在与该切入点匹配的任何连接点处运行（例如，执行具有特定名称的方法）。切入点表达式匹配的连接点的概念是AOP的核心，默认情况下，Spring使用AspectJ切入点表达语言。
+ 简介（Introduction）：代表类型声明其他方法或字段。Spring AOP允许您向任何建议对象引入新接口（和相应的实现）。例如，您可以使用简介使Bean实现 IsModified接口，以简化缓存。（在AspectJ社区中，介绍被称为类型间声明。）
+ 目标对象（Target object）：一个或多个方面建议的对象。也称为“建议对象”。由于Spring AOP是使用运行时代理实现的，因此该对象始终是代理对象。
+ AOP代理（AOP proxy）：由AOP框架创建的一个对象，用于实现方面合同（建议方法执行等）。在Spring Framework中，AOP代理是JDK动态代理或CGLIB代理。
+ 编织（Weaving）：将方面与其他应用程序类型或对象链接以创建建议的对象。这可以在编译时（例如，使用AspectJ编译器），加载时或在运行时完成。像其他纯Java AOP框架一样，Spring AOP在运行时执行编织。

Spring AOP包括以下类型的增强：
前置增强（Before advice）：在连接点之前运行的建议，但是它不能阻止执行流程继续进行到连接点（除非它引发异常）。
后置增强（After returning advice）：在连接点正常完成之后要运行的建议（例如，如果方法返回而没有引发异常）。
异常增强（After throwing advice）：如果方法因引发异常而退出，则运行建议。
后置增强（After (finally) advice）：无论连接点退出的方式如何（正常或特殊收益），均应执行建议。
环绕增强（Around advice）：围绕连接点的建议，例如方法调用。这是最有力的建议。周围建议可以在方法调用之前和之后执行自定义行为。它还负责选择是返回连接点还是通过返回其自身的返回值或引发异常来捷径建议的方法执行。

环绕增强是最通用的增强。

由于Spring AOP与AspectJ一样，提供了各种建议类型，因此我们建议您使用功能最弱的建议类型，以实现所需的行为。
例如，如果您只需要使用方法的返回值更新缓存，则最好使用返回后的建议而不是周围的建议，尽管周围的建议可以完成相同的事情。

使用最具体的建议类型可以提供更简单的编程模型，并减少出错的可能性。
例如，您不需要调用用于around建议的proceed() 方法JoinPoint，因此，您不会失败。

所有建议参数都是静态类型的，因此您可以使用适当类型（例如，方法执行的返回值的类型）而不是Object数组的建议参数。

切入点匹配的连接点的概念是AOP的关键，它与仅提供拦截功能的旧技术有所不同。切入点使建议的目标独立于面向对象的层次结构。
例如，您可以将提供声明性事务管理的环绕建议应用于跨越多个对象（例如服务层中的所有业务操作）的一组方法。

## Spring AOP的能力和目标
Spring AOP是用纯Java实现的。不需要特殊的编译过程。

Spring AOP不需要控制类加载器的层次结构。因此适合在Servlet容器或应用程序服务器中使用。

Spring AOP当前仅支持方法执行连接点。尽管可以在不破坏核心Spring AOP API的情况下添加对字段拦截的支持，但并未实现字段拦截。如果需要建议字段访问和更新连接点，请考虑使用诸如AspectJ之类的语言。

Spring AOP的AOP方法不同于大多数其他AOP框架。目的不是提供最完整的AOP实现，其目的是在AOP实现和Spring IoC之间提供紧密的集成，以帮助解决企业应用程序中的常见问题。因此，通常将Spring Framework的AOP功能与Spring IoC容器结合使用。

通过使用常规bean定义语法来配置方面。这是与其他AOP实现的关键区别。使用Spring AOP不能轻松或高效地完成某些事情，例如建议非常细粒度的对象，在这种情况下，AspectJ是最佳选择。但是，Spring AOP为AOP可以解决的企业Java应用程序中的大多数问题提供了出色的解决方案。

Spring AOP从未努力与AspectJ竞争以提供全面的AOP解决方案。我们认为，基于代理的框架（例如Spring AOP）和成熟的框架（例如AspectJ）都是有价值的，并且它们是互补的，而不是竞争。Spring无缝地将Spring AOP和IoC与AspectJ集成在一起，以在基于Spring的一致应用程序架构中支持AOP的所有使用。这种集成不会影响Spring AOP API或AOP Alliance API。Spring AOP保持向后兼容。

## AOP代理
Spring AOP默认将标准JDK动态代理用于AOP代理。这种代理类必须实现接口。

Spring AOP也可以使用CGLIB代理。这种代理类不是必须实现接口的。默认情况下，如果业务对象未实现接口，则使用CGLIB。

由于最好是对接口而不是对类进行编程，因此业务类通常实现一个或多个业务接口。在某些情况下，当需要在接口上声明的方法或需要将代理对象作为具体类型传递给方法时，可以强制使用CGLIB。

## 基于注解@AspectJ的AOP配置
@AspectJ是一种将切面声明为带有注释的常规Java类的样式。

@AspectJ样式是AspectJ项目在AspectJ 5版本中引入的 。

Spring使用AspectJ提供的用于切入点解析和匹配的库来解释与AspectJ 5相同的注释。但是，AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或编织器。

### 启用@AspectJ支持
要在Spring配置中使用@AspectJ方面，需要启用Spring支持以基于@AspectJ方面配置Spring AOP，并对这些切面进行自动代理。

通过自动代理，如果Spring确定一个或多个方面建议使用bean，它将自动为该bean生成一个代理以拦截方法调用并确保按需运行建议。

可以使用XML或Java样式的配置来启用@AspectJ支持。无论哪种情况，您都需要确保AspectJ的aspectjweaver.jar库位于应用程序的类路径（版本1.8或更高版本）上。该库在libAspectJ发行版本的目录中或从Maven Central存储库中可用 。

#### 通过Java配置启用@AspectJ支持
要使用Java启用@AspectJ支持@Configuration，请添加@EnableAspectJAutoProxy 注释，如以下示例所示：
```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

通过XML配置启用@AspectJ支持
要使用基于XML的配置启用@AspectJ支持，请使用aop:aspectj-autoproxy 元素，如以下示例所示：
```
<aop:aspectj-autoproxy/>
```

导入aop名称空间：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```

### 定义切面
启用@AspectJ支持后，Spring会自动检测在应用程序上下文中使用@AspectJ注解的任何bean，并用于配置Spring AOP。

首先在应用程序上下文中定义常规bean，该定义指向具有@Aspect注解的bean类：
```
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of the aspect here -->
</bean>
```
接下来定义NotVeryUsefulAspect类，该类定义带有org.aspectj.lang.annotation.Aspect注解：
```
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

切面（带有@Aspect注解的类）与任何其他类相同，可以具有方法和字段，可以包含切入点，建议和介绍的声明。	
	
#### 通过组件扫描自动检测切面
可以将切面类注册为Spring XML配置中的常规bean，也可以通过类路径扫描自动检测它们。
但是，@Aspect注解不足以完成在类路径中进行自动检测，需要添加一个@Component注释。

### 定义切点
切点确定了感兴趣的连接点，从而能够控制运行增强的时机。

Spring AOP仅支持Spring Bean的方法执行连接点，因此您可以将切入点视为与Spring Bean上的方法执行相匹配。

切入点声明由两部分组成：一个包含名称和任何参数的方法签名，和一个切入点表达式，该切入点表达式精确确定我们感兴趣的方法。

在AOP的@AspectJ注解样式中，常规方法定义提供了切入点签名，并通过使用@Pointcut注释指示切入点表达式（用作切入点签名的方法返回类型必须是void）。

以下示例定义了一个名为anyOldTransfer的切入点，该切入点与任何名为transfer的方法的执行相匹配：
```
@Pointcut("execution(* transfer(..))") // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```
@Pointcut注释值的切入点表达式是一个AspectJ 5的切入点表达式。

形成@Pointcut注释值的切入点表达式是一个常规的AspectJ 5切入点表达式。有关AspectJ的切入点语言的完整讨论，请参见AspectJ编程指南（以及扩展，包括 AspectJ 5开发人员的笔记本）或有关AspectJ的书籍之一（如Colyer等人的Eclipse AspectJ，或AspectJ in Action）。，由Ramnivas Laddad撰写）。

#### Spring支持的切点指示符
Spring AOP支持以下在切入点表达式中使用的AspectJ切入点指示符（PCD）：
+ execution：用于匹配方法执行的连接点。这是使用Spring AOP时要使用的主要切入点指示符。
+ within：将匹配限制为某些类型内的连接点（使用Spring AOP时，在匹配类型内声明的方法的执行）。
+ this：限制匹配到连接点（使用Spring AOP时方法的执行）的匹配，其中bean引用（Spring AOP代理）是给定类型的实例。
+ target：在目标对象（代理的应用程序对象）是给定类型的实例的情况下，将匹配限制为连接点（使用Spring AOP时方法的执行）。
+ args：将匹配限制为连接点（使用Spring AOP时方法的执行），其中参数是给定类型的实例。
+ @target：在执行对象的类具有给定类型的注释的情况下，将匹配限制为连接点（使用Spring AOP时方法的执行）。
+ @args：限制匹配的连接点（使用Spring AOP时方法的执行），其中传递的实际参数的运行时类型具有给定类型的注释。
+ @within：将匹配限制为具有给定注释的类型内的连接点（使用Spring AOP时，使用给定注释的类型中声明的方法的执行）。
+ @annotation：将匹配点限制为连接点的主题（在Spring AOP中运行的方法）具有给定注释的连接点。

完整的AspectJ切入点语言支持未在Spring支持额外的切入点指示符：call，get，set，preinitialization， staticinitialization，initialization，handler，adviceexecution，withincode，cflow， cflowbelow，if，@this，和@withincode。

在Spring AOP解释的切入点表达式中使用这些切入点指示符会导致IllegalArgumentException抛出异常。

Spring AOP支持的切入点指示符集合可能会在将来的版本中扩展，以支持更多的AspectJ切入点指示符。

#### 组合切点表达式
可以使用 &&, ||和! 组合切入点表达式，也可以按名称引用切入点表达式。

以下示例显示了三个切入点表达式：
```
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.myapp.trading..*)")
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {} 
```
+ anyPublicOperation 如果方法执行连接点表示任何公共方法的执行，则匹配。
+ inTrading 如果交易模块中有方法执行，则匹配。
+ tradingOperation 如果方法执行代表交易模块中的任何公共方法，则匹配。

最佳实践是从较小的命名组件中构建更复杂的切入点表达式。
当按名称引用切入点时，将应用常规的Java可见性规则。
可见性不影响切入点匹配。

#### 共享通用切入点定
在使用企业应用程序时，开发人员通常希望从多个方面引用应用程序的模块和特定的操作集。
我们建议定义一个CommonPointcuts为此方面，以捕获常见的切入点表达式。

这样的方面通常类似于以下示例：
```
package com.xyz.myapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class CommonPointcuts {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.myapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.myapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.myapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.myapp.abc.service and com.xyz.myapp.def.service) then
     * the pointcut expression "execution(* com.xyz.myapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.myapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.myapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

您可以在需要切入点表达式的任何地方引用在这样的方面中定义的切入点。例如，要使服务层具有事务性，您可以编写以下内容：
```
<aop:config>
    <aop:advisor
        pointcut="com.xyz.myapp.CommonPointcuts.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

#### 例子
最经常使用切入点指示符可能是 execution。其表达式的格式如下：
```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)

execution（修饰符模式？返回类型模式 声明类型模式？名称模式(参数模式) 抛出模式？）
```

除了返回类型模式（ret-type-pattern），名称模式和参数模式以外的所有部分都是可选的。
+ 返回类型模式确定该方法的返回类型必须是什么才能使连接点匹配。*最常用作返回类型模式。它匹配任何返回类型。
+ 名称模式与方法名称匹配。将*通配符用作名称模式的全部或一部分。如果指定了声明类型模式，请在其末尾添加一条尾随.以将其连接到名称模式组件。
+ 参数模式稍微复杂一点。()匹配不带参数的方法。(..)匹配任意数量（零个或多个）的参数。(*)模式与采用任何类型的一个参数的方法匹配。 (*,String)与采用两个参数的方法匹配，第一个可以是任何类型，而第二个必须是String。

以下示例显示了一些常见的切入点表达式：

+ 执行任何公共方法：
```
execution(public * *(..))
```

+ 执行名称以set开头的任何方法：
```
 execution(* set*(..))
```

+ 执行AccountService接口定义的任何方法：
```
execution(* com.xyz.service.AccountService.*(..))
```

+ 执行service包中定义的任何方法：
```
execution(* com.xyz.service.*.*(..))
```


+ 执行service包或其子包之一中定义的任何方法：
```
execution(* com.xyz.service..*.*(..))
```

+ service包中的任何连接点（仅在Spring AOP中执行方法）：
```
within(com.xyz.service.*)
```

+ service包或其子包之一中的任何连接点（仅在Spring AOP中执行方法）：
```
within(com.xyz.service..*)
```

+ 代理实现AccountService接口的任何连接点（仅在Spring AOP中执行方法） ：
```
this(com.xyz.service.AccountService)
```

#### 编写高效切点
在编译期间，AspectJ处理切入点以优化匹配性能。检查代码并确定每个连接点是否（静态或动态）匹配给定的切入点是一个昂贵的过程。（动态匹配意味着无法从静态分析中完全确定匹配，并且在代码中进行测试以确定在运行代码时是否存在实际匹配）。首次遇到切入点声明时，AspectJ将其重写为匹配过程的最佳形式。这是什么意思？基本上，切入点以DNF（析取范式）重写，并且对切入点的组件进行排序，以便首先检查那些较便宜的组件。

但是，AspectJ只能使用所告诉的内容。为了获得最佳的匹配性能，您应该考虑他们试图实现的目标，并在定义中尽可能缩小匹配的搜索空间。现有的指示符自然会属于以下三类之一：同类，作用域和上下文：
+ Kinded代号选择特定类型的连接点： execution，get，set，call，和handler。
+ 作用域指定者选择一组感兴趣的连接点（可能有多种）：within和withincode
+ 语境指示符基于上下文匹配（和任选的绑定）： ， this，target和@annotation

编写正确的切入点至少应包括前两种类型（种类和作用域）。您可以包括上下文指示符以根据连接点上下文进行匹配，也可以绑定该上下文以在建议中使用。仅提供同类的标识符或仅提供上下文的标识符是可行的，但是由于额外的处理和分析，可能会影响编织性能（使用的时间和内存）。范围指定符的匹配非常快，使用它们的使用意味着AspectJ可以非常迅速地消除不应进一步处理的连接点组。一个好的切入点应该始终包括一个切入点。


### 定义增强
增强与切入点表达式关联，并且在切入点匹配的方法执行之前，之后或周围运行。
切入点表达式可以是对切入点定义的引用，也可以是实时声明的切入点表达式。

#### 前置增强
可以使用@Before注解在切面@Aspect中在定义前置增强：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
}
```

如果定义实时切入点表达式，则可以将前面的示例重写为以下示例：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }
}
```

#### 后置增强
后置增强，当匹配的方法正常返回时运行。可以使用@AfterReturning注解进行声明：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
}
```

有时，需要在增强中访问返回值。您可以使用@AfterReturning绑定返回值的形式来获得该访问权限，如以下示例所示：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
}
```

returning属性中使用的名称必须与advice方法中的参数名称相对应。
当方法执行返回时，返回值将作为相应的参数值传递到通知方法。
returning属性同时也限制了只能匹配到指定类型返回值（在这种情况下，Object参数可以匹配任何返回值）。

#### 异常增强
异常增强，当匹配的方法执行抛出异常退出时运行。可以使用@AfterThrowing注解进行声明：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }
}
```

如果希望通知仅在特定类型异常时运行，或者需要在方法中访问异常信息。
可以使用该throwing属性来限制匹配，并将抛出的异常绑定到增强参数。
以下示例显示了如何执行此操作：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }
}
```

#### 最终增强
最终增强，当匹配的方法执行退出时运行。它通常用于释放资源和类似目的。
通过使用@After注释声明它。
以下示例显示了最终建议后的用法：
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }
}
```

#### 环绕增强
环绕增强在匹配方法的执行过程中“围绕”运行。它有机会在方法运行之前和之后进行工作，并确定何时、如何运行。
如果需要以线程安全的方式（例如，启动和停止计时器）在方法执行之前和之后共享状态，则通常使用绕行建议。

使用@Around注释来声明环绕增强。

增强方法的第一个参数必须为类型ProceedingJoinPoint。
通过ProceedingJoinPoint调用proceed()方法运行原始方法。
proceed方法还可以传递Object[]，数组中的值用作方法执行时的参数。
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }
}
```
环绕增强返回的值是该方法的调用者看到的返回值。
例如，如果一个简单的缓存方面有一个值，则可以从缓存中返回一个值，如果没有则调用proceed()。
proceed在环绕增强的正文中可能调用一次，多次调用或根本不调用该调用。所有这些都是合法的。

#### 增强的参数
任何增强方法都可以将org.aspectj.lang.JoinPoint类型的参数声明为它的第一个参数 
环绕增强需要声明ProceedingJoinPoint类型的参数，它是的子类JoinPoint。
JoinPoint接口提供了许多有用的方法：
+ getArgs()：返回方法参数。
+ getThis()：返回代理对象。
+ getTarget()：返回目标对象。
+ getSignature()：返回建议使用的方法的描述。
+ toString()：打印有关所建议方法的有用描述。

#### 增强的排序
当多条建议都希望在同一连接点上运行时会发生什么？

Spring AOP遵循与AspectJ相同的优先级规则来确定建议执行的顺序。
在切点入口处，优先级最高的增强将先运行。从切点出口中，优先级最高的增强将后运行。

当在不同切面定义的两条建议都需要在同一连接点上运行时，可以通过指定优先级来控制执行顺序。
通过在切面类中实现org.springframework.core.Ordered接口或对其使用@Order注释。

在@Aspect的类，需要在同一连接点运行基于分配优先级上按以下顺序他们的意见类型，从最高到最低的优先级：@Around，@Before，@After， @AfterReturning，@AfterThrowing。
@After将遵循AspectJ的“之后最终建议”语义，在相同切面的任何@AfterReturning或@AfterThrowing建议方法之后有效地调用@After建议方法。

当在同一@Aspect类中，有两个相同@After类型的建议都需要在同一连接点上运行时，其顺序是不确定的。请考虑将此类建议方法折叠为每个@Aspect类中每个连接点的一个建议方法，或将这些建议重构为单独的@Aspect类，您可以通过Ordered或在方面级别进行排序@Order。

### 引用
引用使切面可以声明增增对象实现给定的接口，并代表那些对象提供该接口的实现。

可以使用@DeclareParents注释进行介绍。

给定名为的接口UsageTracked和名为的接口的实现 DefaultUsageTracked，以下方面声明服务接口的所有实现者也都实现该UsageTracked接口（例如，通过JMX进行统计）：
```
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.CommonPointcuts.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```

要实现的接口由带注释的字段的类型确定。注释的 value属性@DeclareParents是AspectJ类型的模式。任何匹配类型的bean都实现该UsageTracked接口。请注意，在前面示例的建议中，服务Bean可以直接用作UsageTracked接口的实现。如果以编程方式访问bean，则应编写以下内容：
```
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

### 切面实例化模型
略

### AOP示例
有时由于并发问题（例如，死锁失败），业务服务的执行可能会失败。如果重试该操作，则很可能在下一次尝试中成功。
对于适合在这种情况下重试的业务服务，可以透明地重试该操作，通过使用一个环绕增强，可以多次调用proceed。
以下清单显示了基本方面的实现：
```
@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }
}
```

此切面实现了Ordered接口，可以将切面的优先级设置为高于事务增强（既每次重试时都希望有新的事务）。
maxRetries和order都Spring配置。主要动作发生在doConcurrentOperation周围建议中。相应的Spring配置如下：
```
<aop:aspectj-autoproxy/>
<bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
    <property name="maxRetries" value="3"/>
    <property name="order" value="100"/>
</bean>
```

为了改进切面，使其仅重试幂等操作，我们可以定义以下Idempotent注释：
```
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

我们可以使用注释来注释服务操作的实现。方面更改为仅重试幂等操作涉及更改切入点表达式，以便仅@Idempotent操作匹配，如下所示：
```
@Around("com.xyz.myapp.CommonPointcuts.businessService() && " +
        "@annotation(com.xyz.myapp.service.Idempotent)")
public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
    // ...
}
```

## 基于XML的AOP配置
Spring提供了使用aop命名空间标签定义方面的支持。

要使用本节中描述的aop名称空间标签，您需要导入 spring-aop模式：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```

所有Aspect和Advisor元素必须放置在一个<aop:config>元素内，在应用程序上下文配置中可以有多个<aop:config>元素。
一个<aop:config>元素可以包含切入点，顾问和纵横元件（注意这些必须按照这个顺序进行声明）。

### 定义切面
使用<aop:aspect>元素定义一个切面，并使用ref属性来引用支持bean ，如以下示例所示：
```
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
```

### 定义切点
可以在<aop:config>元素内定义一个切点，让切点在多个切面之间共享。
```
<aop:config>
    <aop:pointcut id="businessService" expression="execution(* com.xyz.myapp.service.*.*(..))"/>
</aop:config>
```

可以引用在切入点表达式中的类型（@Aspects）中定义的命名切入点。定义上述切入点的另一种方法如下：
```
<aop:config>
    <aop:pointcut id="businessService"
        expression="com.xyz.myapp.CommonPointcuts.businessService()"/>
</aop:config>
```

可以在切面中声明切点，与声明公告切点非常相似，如以下示例所示：
```
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>
        ...
    </aop:aspect>
</aop:config>
```

### 定义增强
#### 前置增强
在元素<aop:aspect>内部使用元素<aop:before>声明，如以下示例所示：
```
<aop:aspect id="beforeExample" ref="aBean">
    <aop:before
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

要改为内联定义切入点，请按如下所示pointcut-ref用pointcut属性替换属性：
```
<aop:aspect id="beforeExample" ref="aBean">
    <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

#### 后置增强
在元素<aop:aspect>内部使用元素<aop:after-returning>声明。如以下示例所示：
```
<aop:aspect id="afterReturningExample" ref="aBean">
    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

可以在建议正文中获取返回值。为此，使用returning属性指定返回值应传递到的参数的名称，如以下示例所示：
```
<aop:aspect id="afterReturningExample" ref="aBean">
    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        returning="retVal"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

doAccessCheck方法必须声明一个名为的参数retVal，该参数的类型以与所述相同的方式约束匹配@AfterReturning。示列如下：
```
public void doAccessCheck(Object retVal) {
	...
｝
```

#### 异常增强
在元素<aop:aspect>内部使用元素<aop:after-throwing>声明。如以下示例所示：
```
<aop:aspect id="afterThrowingExample" ref="aBean">
    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        method="doRecoveryActions"/>
    ...
</aop:aspect>
```

使用throwing属性指定异常应传递到的参数的名称，如以下示例所示：
```
<aop:aspect id="afterThrowingExample" ref="aBean">
    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        throwing="dataAccessEx"
        method="doRecoveryActions"/>
    ...
</aop:aspect>
```

#### 最终增强
在元素<aop:aspect>内部使用元素<aop:after>声明。如以下示例所示：
```
<aop:aspect id="afterFinallyExample" ref="aBean">
    <aop:after
        pointcut-ref="dataAccessOperation"
        method="doReleaseLock"/>
    ...
</aop:aspect>
```

#### 环绕增强
在元素<aop:aspect>内部使用元素<aop:around>声明。
```
<aop:aspect id="aroundExample" ref="aBean">
    <aop:around
        pointcut-ref="businessService"
        method="doBasicProfiling"/>
    ...
</aop:aspect>
```
方法的第一个参数必须为类型ProceedingJoinPoint。
使用ProceedingJoinPoint调用proceed()运行基本方法。

#### 增强的参数
略

#### 增强的排序
略

### 引用
引用 使切面可以声明实现给定的接口增强的对象，并代表那些对象提供该接口的实现。

使用aop:aspect中的aop:declare-parents元素进行介绍，使用aop:declare-parents元素声明匹配类型具有新的父对象。
```
<aop:aspect id="usageTrackerAspect" ref="usageTracking">
    <aop:declare-parents
        types-matching="com.xzy.myapp.service.*+"
        implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
        default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>
    <aop:before
        pointcut="com.xyz.myapp.CommonPointcuts.businessService()
            and this(usageTracked)"
            method="recordUsage"/>
</aop:aspect>
```

实现usageTrackingbean的类将包含以下方法：
```
public void recordUsage(UsageTracked usageTracked) {
    usageTracked.incrementUseCount();
}
```

要实现的接口由implement-interface属性确定。types-matching属性的值是AspectJ类型的模式。任何匹配类型的bean都实现该UsageTracked接口。

请注意，在前面示例的建议中，服务Bean可以直接用作UsageTracked接口的实现。要以编程方式访问bean，可以编写以下代码：
```
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

### 切面实例化模型
略

### 顾问 Advisors
“顾问”的概念来自Spring中定义的AOP支持，并且在AspectJ中没有直接等效的概念。
顾问就像一个独立的小方面，只有一条建议。
通知本身由bean表示，并且必须实现Spring的“建议类型”中描述的建议接口 之一。
顾问可以利用AspectJ切入点表达式。

Spring支持带有<aop:advisor>元素的顾问程序概念。通常会与事务建议结合使用，事务建议在Spring中也有其自己的名称空间支持。
以下示例显示顾问程序：
```
<aop:config>
    <aop:pointcut id="businessService" expression="execution(* com.xyz.myapp.service.*.*(..))"/>
    <aop:advisor pointcut-ref="businessService" advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```
除了前面的示例中使用的 pointcut-ref 属性外，还可以使用该 pointcut 属性来内联定义切入点表达式。

### AOP示例
略

## 如何选择AOP
使用Spring AOP 或者 AspectJ？
使用 注解@AspectJ 或者 Spring AOP的XML？
这些决定受许多因素的影响，包括应用程序需求，开发工具和团队对AOP的熟悉程度。

### Spring AOP还是Full AspectJ？
使用最简单的方法即可。
Spring AOP比使用完整的AspectJ更简单，因为不需要在开发和构建过程中引入AspectJ编译器/编织器。
如果您只需要建议在Spring Bean上执行操作，则Spring AOP是正确的选择。
如果您需要建议不受Spring容器管理的对象（通常是域对象），则需要使用AspectJ。
如果您希望建议除简单方法执行之外的连接点（例如，字段获取或设置连接点等），则还需要使用AspectJ。

### @AspectJ或Spring AOP的XML？
XML样式可能是现有Spring用户最熟悉的，并且得到了真正的POJO的支持。
当使用AOP作为配置企业服务的工具时，XML可能是一个不错的选择。
使用XML样式，可以说从您的配置中可以更清楚地了解系统中存在哪些方面。

XML样式有两个缺点。
首先，它没有完全将要解决的需求的实现封装在一个地方。DRY原则说，系统中的任何知识都应该有一个单一，明确，权威的表示形式。当使用XML样式时，关于如何实现需求的知识会在配置文件中的后备bean类的声明和XML中分散。当您使用@AspectJ样式时，此信息将封装在一个模块中：切面。
其次，与@AspectJ样式相比，XML样式在表达能力上有更多限制：仅支持“单例”方面实例化模型，并且无法组合以XML声明的命名切入点。

例如，使用@AspectJ样式，您可以编写如下内容：
```
@Pointcut("execution(* get*())")
public void propertyAccess() {}

@Pointcut("execution(org.xyz.Account+ *(..))")
public void operationReturningAnAccount() {}

@Pointcut("propertyAccess() && operationReturningAnAccount()")
public void accountPropertyAccess() {}
```
在XML样式中，您可以声明前两个切入点：
```
<aop:pointcut id="propertyAccess"
        expression="execution(* get*())"/>

<aop:pointcut id="operationReturningAnAccount"
        expression="execution(org.xyz.Account+ *(..))"/>
```

XML方法的缺点是无法通过组合这些定义来定义切入点。
@AspectJ样式支持其他实例化模型和更丰富的切入点组合。它具有将方面保持为模块化单元的优势。
@AspectJ样式还具有的优点是，Spring AOP和AspectJ都可以理解@AspectJ方面。如果以后需要AspectJ的功能来实现其他要求，则可以轻松地迁移到经典的AspectJ设置。
总而言之，Spring团队在自定义方面更喜欢@AspectJ样式，而不是简单地配置企业服务。

## 混合切面类型
通过使用自动代理支持，模式定义的<aop:aspect>方面，<aop:advisor>声明的顾问程序，甚至是同一配置中其他样式的代理和拦截器，完全可以混合@AspectJ样式的方面。

所有这些都是通过使用相同的基础支持机制实现的，并且可以毫无困难地共存。

## !!!代理机制
Spring AOP使用JDK动态代理或CGLIB创建给定目标对象的代理。
JDK内置了JDK动态代理，而CGLIB是一个通用的开源类定义库（重新打包为spring-core）。

**如果要代理的目标对象实现至少一个接口，则可以使用JDK动态代理。**
**如果目标对象未实现任何接口，则将创建CGLIB代理。**

可以强制开启使用CGLIB代理，考虑以下问题：
+ 使用CGLIB，不能增强final方法，因为不能在运行时生成的子类中覆盖方法。
+ 从Spring 4.0开始，由于CGLIB代理实例是通过Objenesis创建的，因此不再调用代理对象的构造函数两次。仅当您的JVM不允许绕过构造函数时，您才可以从Spring的AOP支持中看到两次调用和相应的调试日志条目。

强制使用CGLIB代理，请将<aop:config>元素的proxy-target-class属性值设置为true，如下所示：
```
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```
在使用@AspectJ自动代理支持时强制CGLIB代理，请将<aop:aspectj-autoproxy>元素的proxy-target-class属性设置为true，如下所示：
```
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

### !!!了解AOP代理
**Spring AOP是基于代理的。（或是JDK动态代理，或是CGLIB代理）**
在编写自己的方面或使用Spring框架随附的任何基于Spring AOP的方面之前，掌握这条语句实际含义的语义至关重要。

考虑以下情形：有一个普通的未经代理的的对象引用，如以下代码片段所示：
```
public class SimplePojo implements Pojo {

    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        this.bar();
    }

    public void bar() {
        // some logic...
    }
}
```

如果在对象引用上调用方法，则直接在该对象引用上调用该方法，如下图和清单所示：
```
public class Main {

    public static void main(String[] args) {
        Pojo pojo = new SimplePojo();
        // this is a direct method call on the 'pojo' reference
        pojo.foo();
    }
}
```

如果代码中的引用是代理时，情况会稍有变化。考虑以下图表和代码片段：
```
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```
这里要了解的关键是：
main方法中的代码是对代理的引用，这意味着该对象引用上的方法调用是代理上的调用，代理可以委派给与该特定方法调用相关的所有AOP增强。
但是一旦调用最终到达目标对象（SimplePojo），它对自身进行的任何方法调用（例如this.bar()或 this.foo()）都将针对this引用而不是代理进行调用。
这意味着**自调用不会运行与方法相关的AOP增强**。

那该怎么办？
**最好的方法是重构代码，以免发生自调用**。这确实需要您做一些工作，但这是最好的，侵入性最小的方法。
还有一种可怕的方法，可以完全将类中的逻辑与Spring AOP绑定在一起，如以下示例所示：
```
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```
这将您的代码完全耦合到Spring AOP，并且使类本身意识到在AOP上下文中使用它的事实，绝对可怕。

**AspectJ没有此自调用问题，因为它不是基于代理的AOP框架。**

## 以编程方式创建@AspectJ代理
除了使用<aop:config> 或<aop:aspectj-autoproxy>来声明配置中的各个方面外，还可以通过编程方式创建建议目标对象的代理。

可以使用 org.springframework.aop.aspectj.annotation.AspectJProxyFactory 类为@AspectJ切面增强的目标对象创建代理。
此类的基本用法非常简单，如以下示例所示：
```
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```

## 在Spring中使用AspectJ
如果需求超出了Spring AOP所提供的功能，那么需要使用AspectJ编译器或weaver代替Spring AOP或在其中使用。
Spring附带了一个小型的AspectJ方面库，可以在发行版中独立使用spring-aspects.jar。

### 通过Spring使用AspectJ注入域对象
略

### AspectJ的其他Spring方面
略

### 使用Spring IoC配置AspectJ Aspects
略

### 在Spring Framework中使用AspectJ进行加载时编织
略

## 更多资源
更多AspectJ资源可以在[AspectJ网站](https://www.eclipse.org/aspectj)上获取。

# Spring AOP API
上一章通过@AspectJ和基于模式的方面定义描述了Spring对AOP的支持。在本章中，我们讨论了较低级别的Spring AOP API。
对于常见的应用程序，我们建议将Spring AOP与AspectJ切入点一起使用，如上一章所述。

略

# Null安全
尽管Java不允许您使用其类型系统来表示空安全性，但Spring在org.springframework.lang包中提供了以下注释，以使您可以声明API和字段的空性：
+ @Nullable：表示特定参数，返回值或字段可以为的注释null。
+ @NonNull：表示不能指定特定参数，返回值或字段的注释null（分别不需要在参数/返回值和字段@NonNullApi以及@NonNullFields应用的字段上）。
+ @NonNullApi：程序包级别的注释，它声明非null为参数和返回值的默认语义。
+ @NonNullFields：程序包级别的注释，它声明非null作为字段的默认语义。

Spring框架本身利用了这些注释，但是它们也可以在任何基于Spring的Java项目中使用，以声明null安全的API和可选的null安全的字段。
其他常见的库（例如Reactor和Spring Data）提供了使用类似可空性设置的空安全API，从而为Spring应用程序开发人员提供了一致的总体体验。

## 用例
除了为Spring Framework API可空性提供显式声明外，IDE（例如IDEA或Eclipse）还可以使用这些注释来提供与空安全性相关的有用警告，从而避免NullPointerException在运行时出现警告。

## JSR-305元注释
Spring注释使用JSR-305元注释进行元注释。

JSR-305元注释使工具供应商（如IDEA或Kotlin）以通用方式提供了空安全支持，而无需对Spring注释进行硬编码支持。
既不需要也不建议向项目类路径中添加JSR-305依赖项以利用Spring空安全API。

# 数据缓冲区和编解码器
Java NIO提供了ByteBuffer，但是许多库在构建了自己的字节缓冲区API，特别是对于网络操作，其中重用缓冲区和/或使用直接缓冲区对性能有利。
例如，Netty具有ByteBuf层次结构，Undertow使用XNIO，Jetty使用具有要释放的回调的池化字节缓冲区，依此类推。

spring-core模块提供了一组抽象，可与各种字节缓冲区API配合使用，如下所示：
+ DataBufferFactory 抽象数据缓冲区的创建。
+ DataBuffer表示一个字节缓冲区，可以将其 合并。
+ DataBufferUtils 提供数据缓冲区的实用方法。
+ 编解码器将流数据缓冲区流解码或编码为更高级别的对象。

## DataBufferFactory
DataBufferFactory 用于通过以下两种方式之一创建数据缓冲区：
+ 分配一个新的数据缓冲区，可以选择预先指定容量（如果已知），即使容量的实现DataBuffer可以按需增长和缩小，该容量也会更有效。
+ 包装一个现有的byte[]或java.nio.ByteBuffer，用一个DataBuffer实现装饰给定的数据，并且不涉及分配。

请注意，WebFlux应用程序不会DataBufferFactory直接创建一个而是通过客户端上的ServerHttpResponse或访问它ClientHttpRequest。

工厂的类型取决于基础客户端或服务器，例如 NettyDataBufferFactoryReactor NettyDefaultDataBufferFactory等。

## DataBuffer
该DataBuffer界面提供的操作类似，java.nio.ByteBuffer但也带来了一些其他好处，其中一些是受Netty启发的ByteBuf。
以下是部分好处清单：
+ 具有独立位置的读取和写入，即不需要调用flip()在读取和写入之间交替。
+ 与一样，容量可以按需扩展java.lang.StringBuilder。
+ 通过合并缓冲和引用计数PooledDataBuffer。
+ 查看缓冲区java.nio.ByteBuffer，InputStream或OutputStream。
+ 确定给定字节的索引或最后一个索引。

## PooledDataBuffer
如Javadoc中 ByteBuffer所述，字节缓冲区可以是直接的也可以是非直接的。直接缓冲区可以驻留在Java堆之外，从而无需复制本机I / O操作。这使得直接缓冲区对于通过套接字接收和发送数据特别有用，但是它们的创建和释放也更昂贵，这导致了缓冲池的想法。

PooledDataBuffer是对DataBuffer引用计数的帮助，这对于字节缓冲区池至关重要。它是如何工作的？当PooledDataBuffer被分配的引用计数为1.呼吁以retain()增加计数，而调用release()递减它。只要计数大于0，就保证不会释放缓冲区。当计数减少到0时，可以释放池中的缓冲区，这实际上意味着将为缓冲区保留的内存返回到内存池。

请注意PooledDataBuffer，在大多数情况下，与其直接操作，不如在DataBufferUtils应用释放或保留到 DataBuffer的实例（仅当是的实例）中使用便捷方法，因此更好PooledDataBuffer。

## DataBufferUtils
DataBufferUtils 提供了许多实用的方法来对数据缓冲区进行操作：
+ 如果底层字节缓冲区API支持，则可以通过复合缓冲区将数据缓冲区的流连接到单个缓冲区（可能带有零副本），例如零复制。
+ 打开InputStream或NIOChannel成Flux<DataBuffer>，反之亦然 Publisher<DataBuffer>进入OutputStream或NIO Channel。
+ DataBuffer如果缓冲区是的实例，则 释放或保留的方法PooledDataBuffer。
+ 从字节流中跳过或获取，直到特定的字节数为止。

## Codecs
org.springframework.core.codec软件包提供以下策略接口：
+ Encoder编码Publisher<T>为数据缓冲区流。
+ Decoder解码Publisher<DataBuffer>成更高级别的对象流。

该spring-core模块提供byte[]，ByteBuffer，DataBuffer，Resource，和 String编码器和解码器实现。
该spring-web模块添加了Jackson JSON，Jackson Smile，JAXB2，协议缓冲区以及其他编码器和解码器。

## 使用 DataBuffer
使用数据缓冲区时，必须特别小心以确保释放缓冲区，因为它们可能会被合并。
我们将使用编解码器来说明其工作原理，但是这些概念将更普遍地应用。
让我们看看编解码器必须在内部执行哪些操作来管理数据缓冲区。

Decoder在创建更高级别的对象之前，A是最后一个读取输入数据缓冲区的对象，因此，它必须按以下方式释放它们：
+ 如果Decoder只需读取每个输入缓冲区并准备立即释放它，则可以通过进行操作DataBufferUtils.release(dataBuffer)。
+ 如果Decoder是使用Flux或Mono运营商，如flatMap，reduce和其他该预取和高速缓存的数据项内部，或者是使用运算符，如 filter，skip即离开了物品和其他，然后 doOnDiscard(PooledDataBuffer.class, DataBufferUtils::release)必须被添加到该组合物链以确保这些缓冲器之前被释放被丢弃，可能还导致错误或取消信号。
+ 如果Decoder以任何其他方式保留一个或多个数据缓冲区，则必须确保在完全读取时释放它们，或者在读取和释放缓存的数据缓冲区之前发生错误或取消信号的情况下。

请注意，这DataBufferUtils#join提供了一种安全有效的方法来将数据缓冲区流聚合到单个数据缓冲区中。同样skipUntilByteCount， takeUntilByteCount它们是供解码器使用的其他安全方法。

一个Encoder分配其他人必须读取（和释放）的数据缓冲区。因此，Encoder 没有太多的工作要做。但是，Encoder如果在使用数据填充缓冲区时发生序列化错误，则必须注意释放数据缓冲区。例如：
```
DataBuffer buffer = factory.allocateBuffer();
boolean release = true;
try {
    // serialize and populate buffer..
    release = false;
}
finally {
    if (release) {
        DataBufferUtils.release(buffer);
    }
}
return buffer;
```
Encoder负责释放其接收的数据缓冲区

# 日志
从Spring Framework 5.0开始，Spring在spring-jcl模块中实现了自己的Commons Logging桥。
该实现检查类路径中是否存在Log4j 2.x API和SLF4J 1.7 API，并使用发现的第一个作为日志记录实现。
如果Log4j 2.x和SLF4J都不可用，则回退到Java平台的核心日志记录设施（也称为JUL或java.util.logging）。
将Log4j 2.x或Logback（或其他SLF4J提供程序）放入您的类路径中，而无需任何额外的桥接，并让框架自动适应您的选择。

Spring的Commons Logging变体仅用于核心框架和扩展中的基础结构日志记录目的。
对于应用程序代码中的日志记录需求，建议直接使用Log4j 2.x，SLF4J或JUL。

Log实现可以通过检索org.apache.commons.logging.LogFactory如在下面的例子：
```
public class MyBean {
    private final Log log = LogFactory.getLog(getClass());
    // ...
}
```

# 附录

## XML Schema
此部分列出了与核心容器相关的XML模式。

### util Schema
util标记处理常见的实用程序配置问题，例如配置集合，引用常量等。
要在util架构中使用标签，您需要在Spring XML配置文件的顶部具有以下前导：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

        <!-- bean definitions here -->

</beans>
```

#### 使用 util：constant
考虑以下bean定义：
```
<bean id="..." class="...">
    <property name="isolation">
        <bean id="java.sql.Connection.TRANSACTION_SERIALIZABLE"
                class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean" />
    </property>
</bean>
```

前面的配置使用SpringFactoryBean实现（ FieldRetrievingFactoryBean）将isolationbean上的属性的值设置为java.sql.Connection.TRANSACTION_SERIALIZABLE常量的值。
这一切都很好，但是很冗长，并且（不必要地）向最终用户公开了Spring的内部管道。

以下基于XML Schema的版本更加简洁，清楚地表达了开发人员的意图（“注入此常量值”），并且读起来更好：
```
<bean id="..." class="...">
    <property name="isolation">
        <util:constant static-field="java.sql.Connection.TRANSACTION_SERIALIZABLE"/>
    </property>
</bean>
```

#### 使用 util：property-path
略

#### 使用 util：properties
使用PropertiesFactoryBean实例化一个java.util.Properties实例，该实例从提供的Resource位置加载值：
```
<!-- creates a java.util.Properties instance with values loaded from the supplied location -->

<bean id="jdbcConfiguration" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
    <property name="location" value="classpath:com/foo/jdbc-production.properties"/>
</bean>
```

以下示例使用一个util:properties元素来进行更简洁的表示：
```
<!-- creates a java.util.Properties instance with values loaded from the supplied location -->

<util:properties id="jdbcConfiguration" location="classpath:com/foo/jdbc-production.properties"/>
```

#### 使用 util：list
使用ListFactoryBean来创建java.util.List实例，并使用value中的值对sourceList进行初始化：
```
<!-- creates a java.util.List instance with values loaded from the supplied 'sourceList' -->

<bean id="emails" class="org.springframework.beans.factory.config.ListFactoryBean">
    <property name="sourceList">
        <list>
            <value>pechorin@hero.org</value>
            <value>raskolnikov@slums.org</value>
            <value>stavrogin@gov.org</value>
            <value>porfiry@gov.org</value>
        </list>
    </property>
</bean>
```

使用一个<util:list/>元素来进行更简洁的表示：
```
<!-- creates a java.util.List instance with the supplied values -->

<util:list id="emails">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:list>
```

还可以List通过使用元素list-class上的属性来显式控制实例化和填充的类型的确切类型。
例如，如果需要java.util.LinkedList实例化，则可以使用以下配置：
```
<util:list id="emails" list-class="java.util.LinkedList">
    <value>jackshaftoe@vagabond.org</value>
    <value>eliza@thinkingmanscrumpet.org</value>
    <value>vanhoek@pirate.org</value>
    <value>d'Arcachon@nemesis.org</value>
</util:list>
```
如果未list-class提供任何属性，则容器自己选择一个List实现。

#### 使用 util：map
使用MapFactoryBeanMapFactoryBean实现来创建一个java.util.Map实例，该实例使用从提供的中获取的键值对进行初始化'sourceMap'：
```
<!-- creates a java.util.Map instance with values loaded from the supplied 'sourceMap' -->

<bean id="emails" class="org.springframework.beans.factory.config.MapFactoryBean">
    <property name="sourceMap">
        <map>
            <entry key="pechorin" value="pechorin@hero.org"/>
            <entry key="raskolnikov" value="raskolnikov@slums.org"/>
            <entry key="stavrogin" value="stavrogin@gov.org"/>
            <entry key="porfiry" value="porfiry@gov.org"/>
        </map>
    </property>
</bean>
```

使用<util:map/>元素来进行更简洁的表示：
```
<!-- creates a java.util.Map instance with the supplied key-value pairs -->
<util:map id="emails">
    <entry key="pechorin" value="pechorin@hero.org"/>
    <entry key="raskolnikov" value="raskolnikov@slums.org"/>
    <entry key="stavrogin" value="stavrogin@gov.org"/>
    <entry key="porfiry" value="porfiry@gov.org"/>
</util:map>
```

还可以通过使用属性'map-class'来显式控制实例化和填充的类型的确切类型。
例如，如果我们确实需要java.util.TreeMap实例化，则可以使用以下配置：
```
<util:map id="emails" map-class="java.util.TreeMap">
    <entry key="pechorin" value="pechorin@hero.org"/>
    <entry key="raskolnikov" value="raskolnikov@slums.org"/>
    <entry key="stavrogin" value="stavrogin@gov.org"/>
    <entry key="porfiry" value="porfiry@gov.org"/>
</util:map>
```
如果未'map-class'提供任何属性，则容器自行选择一个Map实现。

#### 使用 util：set
使用SetFactoryBean来创建java.util.Set实例，该实例使用从提供的中获取的值进行初始化sourceSet：
```
<!-- creates a java.util.Set instance with values loaded from the supplied 'sourceSet' -->
<bean id="emails" class="org.springframework.beans.factory.config.SetFactoryBean">
    <property name="sourceSet">
        <set>
            <value>pechorin@hero.org</value>
            <value>raskolnikov@slums.org</value>
            <value>stavrogin@gov.org</value>
            <value>porfiry@gov.org</value>
        </set>
    </property>
</bean>
```

使用一个<util:set/>元素来进行更简洁的表示：
```
<!-- creates a java.util.Set instance with the supplied values -->
<util:set id="emails">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:set>
```

可以通过使用元素上的属性set-class来显式控制实例化和填充的类型的确切类型<util:set/>。
例如，如果我们确实需要java.util.TreeSet实例化a，则可以使用以下配置：
```
<util:set id="emails" set-class="java.util.TreeSet">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:set>
```

如果未set-class提供任何属性，则容器选择一个Set实现。

### aop Schema
这些aop标记涉及在Spring中配置所有AOP，包括Spring自己的基于代理的AOP框架以及Spring与AspectJ AOP框架的集成。

为了完整起见，要在aop架构中使用标记，您需要在Spring XML配置文件的顶部具有以下前导：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```

### context Schema
该context标签处理ApplicationContext的配置。

以下代码段引用了正确的架构，以便context您可以使用命名空间中的元素：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- bean definitions here -->

</beans>
```

#### 使用 property-placeholder
此元素激活${…​}占位符的替换，这些占位符针对指定的属性文件（作为Spring资源location）解析。

此元素是PropertySourcesPlaceholderConfigurer为您设置的便捷机制。

如果需要对特定PropertySourcesPlaceholderConfigurer设置进行更多控制，则可以自己将其显式定义为Bean。

#### 使用 annotation-config
此元素激活Spring基础结构以检测Bean类中的注释：
+ Spring的@Configuration模型
+ @Autowired/@Inject和@Value
+ JSR-250的@Resource，@PostConstruct和@PreDestroy（如果可用）
+ JPA@PersistenceContext和@PersistenceUnit（如果有）
+ Spring的@EventListener

或者，您可以选择BeanPostProcessors 为这些注释显式激活个人。

这个元素不会激活Spring@Transactional注释的处理，可以使用元素<tx:annotation-driven/> 启用目的。

Spring的缓存注释也需要显式地启用。

#### 使用 component-scan

#### 使用 load-time-weaver

#### 使用 spring-configured

#### 使用 mbean-export

### beans Schema
自框架诞生以来，beans元素就一直出现在春季。

以下示例显示<meta/>了周围环境中的元素<bean/> ：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="foo" class="x.y.Foo">
        <meta key="cacheName" value="foo"/> 
        <property name="name" value="Rick"/>
    </bean>

</beans>
```

## XML Schema 创作
从2.0版开始，Spring提供了一种机制，该机制可将基于架构的扩展添加到用于定义和配置Bean的基本Spring XML格式中。

为了便于编写使用支持模式的XML编辑器的配置文件，Spring的可扩展XML配置机制基于XML Schema。

要创建新的XML配置扩展，请执行以下操作：
+ 编写XML模式以描述您的自定义元素。
+ 编写自定义NamespaceHandler实现的代码。
+ 对一个或多个BeanDefinitionParser实现进行编码（这是完成实际工作的地方）。
+ 向Spring注册您的新工件。

在下面的示例中，我们创建一个XML扩展（一个自定义XML元素），该扩展使我们可以SimpleDateFormat（从java.text包中）配置类型的对象。
完成后，我们将能够SimpleDateFormat如下定义bean类型的定义：
```
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```

### 编写架构
### 编码NamespaceHandler
### 使用BeanDefinitionParser
### 注册处理程序和架构
### 在Spring XML配置中使用自定义扩展
### 更详细的例子

## 应用程序启动步骤
此部分列出了 **启动步骤** 核心容器所使用的现有资源。

核心容器中定义的应用程序启动步骤：

|名称	|描述	|标签	|
|--	|--	|--	|
|spring.beans.instantiate	|Bean及其依赖关系的实例化。	|beanNameBean的名称，beanType注入点所需的类型。	|
|spring.beans.smart-initialize	|初始化SmartInitializingSingletonbean。	|beanName 豆的名称。	|
|spring.context.annotated-bean-reader.create	|创造的AnnotatedBeanDefinitionReader。	|	|
|spring.context.base-packages.scan	|扫描基本软件包。	|packages 用于扫描的基本软件包的数组。	|
|spring.context.beans.post-process	|Bean后处理阶段。	|	|
|spring.context.bean-factory.post-process	|调用BeanFactoryPostProcessorbean。	|postProcessor 当前的后处理器。	|
|spring.context.beandef-registry.post-process	|调用BeanDefinitionRegistryPostProcessorbean。	|postProcessor 当前的后处理器。	|
|spring.context.component-classes.register	|通过注册组件类AnnotationConfigApplicationContext#register。	|classes 用于注册的给定类的数组。	|
|spring.context.config-classes.enhance	|使用CGLIB代理增强配置类。	|classCount 增强类的数量。	|
|spring.context.config-classes.parse	|配置类使用解析阶段ConfigurationClassPostProcessor。	|classCount 处理的类数。	|
|spring.context.refresh	|	|应用程序上下文刷新阶段。	|

