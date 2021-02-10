# 参考资料
> [Spring参考文档](https://docs.spring.io/spring-framework/docs/current/reference/html/)

--------------------------------------------------
此部分的文档涵盖对基于Servlet API构建并部署到Servlet容器的Servlet堆栈Web应用程序的支持。

# Spring Web MVC
Spring Web MVC是基于Servlet API构建的原始Web框架，从一开始就已包含在Spring框架中。

正式名称“ Spring Web MVC”来自其源模块（spring-webmvc）的名称，但更通常称为“ Spring MVC”。

Spring Framework 5.0引入了一个反应式堆栈Web框架，其名称“ Spring WebFlux”基于其源模块（spring-webflux）。

## !!!DispatcherServlet 调度器
Spring MVC围绕着前端控制器模式设计一个核心Servlet：DispatcherServlet，提供了用于请求处理的共享算法，而实际工作是由可配置的委托组件执行的。

DispatcherServlet作为Servlet，需要通过使用Java配置或web.xml进行定义声明。

另外，DispatcherServlet使用Spring配置文件来发现它所需要的请求映射，视图解析，异常处理，委托组件等配置信息。

以下示例使用Java配置注册并初始化DispatcherServlet：
```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

以下示例使用web.xml配置注册并初始化DispatcherServlet：
```
<web-app>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

**Spring Boot遵循不同的初始化顺序。**
Spring Boot并没有陷入Servlet容器的生命周期，而是使用Spring配置来引导自身和嵌入式Servlet容器。

### Context Hierarchy 上下文结构
DispatcherServlet预定义了一个WebApplicationContext，可以对自身进行扩展配置。
WebApplicationContext具有指向ServletContext和Servlet的链接。
WebApplicationContext也与ServletContext绑定，这样applications可以使用RequestContextUtils的静态方法来查找WebApplicationContext。

对于许多应用程序而言，一个WebApplicationContext是足够的。
也可以是上下文层次结构，其中 根WebApplicationContext 在多个DispatcherServlet实例之间共享，每个实例都有其自己的 子WebApplicationContext 配置。

根WebApplicationContext 通常包含基础结构bean，例如需要在多个Servlet实例之间共享的数据存储库和业务服务。
这些Bean被有效地继承，并且可以在Servlet特定的子级中重写，子WebApplicationContext 通常包含给定本地的Servlet Bean。

下图显示了这种关系：
![mvc-context-hierarchy](image/mvc-context-hierarchy.png)

以下示例配置WebApplicationContext层次结构：
```
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```
**如果不需要应用程序上下文层次结构，则可以通过从```getRootConfigClasses()```和```getServletConfigClasses()```返回null配置。**

以下示例显示了web.xml等效项：
```
<web-app>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>
</web-app>
```
**如果不需要应用程序上下文层次结构，则应用程序可以仅配置“根”上下文，并将Servlet的```contextConfigLocation```参数保留为空。**

#### Spring Web应用中三个上下文
在Spring web应用中有三个上下文：spring root上下文（WebApplicationContext）、spring mvc上下文（WebApplicationContext）和web应用上下文(ServletContext)。

要想很好理解这三个上下文的关系，需要先熟悉spring是怎样在web容器中启动起来的。

spring的启动过程其实就是其IoC容器的启动过程，对于web程序，IoC容器启动过程即是建立上下文的过程。

spring的启动过程：
一个web应用，其部署在web容器中，web容器提供其一个全局的上下文环境，这个上下文就是ServletContext，其为后面的spring IoC容器提供宿主环境。
在常见的应用web.xml中，有提供一个```org.springframework.web.context.ContextLoaderListener```。
在web容器启动时，会触发容器初始化事件，此时contextLoaderListener会监听到这个事件，其contextInitialized方法会被调用。
在这个方法中，spring会初始化一个启动上下文，这个上下文被称为根上下文，即```WebApplicationContext```。
这是一个接口类，实际的实现类是```XmlWebApplicationContext```。
这个就是spring的IoC容器，其对应的Bean定义的配置由web.xml中的context-param标签定义的```contextConfigLocation```参数指定。
在这个IoC容器初始化完毕后，spring以```WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE```为属性Key，将其存储到ServletContext中，便于获取。

contextLoaderListener监听器初始化完毕后，开始初始化web.xml中配置的Servlet，这个servlet可以配置多个。
以Spring web应用中的DispatcherServlet为例，这个servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个servlet请求。
DispatcherServlet上下文在初始化的时候会建立自己的IoC上下文，用以持有spring mvc相关的bean。
DispatcherServlet在建立自己的IoC上下文时，会利用```WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE```先从```ServletContext```中获取之前的**根上下文**(即```WebApplicationContext```)作为自己上下文的```parent```上下文。有了这个parent上下文之后，再初始化自己持有的上下文。
DispatcherServlet初始化自己上下文的工作在其initStrategies方法中可以看到，大概的工作就是**初始化处理器映射、视图解析**等。
这个servlet自己持有的上下文默认实现类也是```XmlWebApplicationContext```。
初始化完毕后，spring以与**servlet的名字相关**(此处不是简单的以servlet名为Key，而是通过一些转换，具体可自行查看源码)的属性为属性Key，也将其存到ServletContext中，以便后续使用。
这样每个servlet就持有自己的上下文，即拥有自己独立的bean空间，同时各个servlet共享相同的bean，即**根上下文**定义的那些bean。

#### ServletContext
ServletContext实例可以做很多事情，例如通过调用getResourceAsStream（）方法来访问WEB-INF资源（xml配置等）。通常，在Servlet Spring应用程序的web.xml中定义的所有应用程序上下文都是Web应用程序上下文，这既适用于根Webapp上下文，也适用于Servlet的应用程序上下文。

另外，取决于Web应用程序上下文的功能，可能会使你的应用程序更难测试，并且可能需要使用MockServletContext类进行测试。

#### root WebApplicationContext 与 servlet WebApplicationContext
servlet上下文和root上下文之间的关系：
Spring允许你构建多级应用程序上下文层次结构，因此，如果当前应用程序上下文中不存在所需的bean，则会从父上下文中获取所需的bean。
默认情况下，在Web应用程序中，有两个层次结构级别。
这使你可以将某些服务作为整个应用程序的单例运行（Spring Security Bean和基本数据库访问服务通常位于此处），
而另一项则作为相应服务中的单独服务运行，以避免Bean之间发生名称冲突。
例如，**一个Servlet上下文将为网页提供服务，而另一个将实现无状态Web服务**。

### !!!Special Bean Types 特殊Bean类型
```DispatcherServlet```委托```特殊Bean```来处理请求以及提供适当响应。

这里所说的```特殊Bean```，指的是Spring管理的实现了框架约定的对象实例。
它们通常带有内置的约定，但您可以自定义它们的属性并扩展或替换它们。

下表列出了```DispatcherServlet```会检测到的特殊Bean：

|Bean类型	|说明	|
|--	|--	|
|HandlerMapping	|将一个```request```映射到```处理器```以及```拦截器```，映射规则因```HandlerMapping```实现而异。```HandlerMapping```的两个主要实现是```RequestMappingHandlerMapping```(用于支持@RequestMapping注解)和```SimpleUrlHandlerMapping```(用于向程序注册URI路径)。|
|HandlerAdapter	|帮助```DispatcherServlet```调用映射到请求的处理程序	，而不管实际如何调用该处理程序。例如，调用带注解的控制器需要解析注解。```HandlerAdapter``` 的主要目的是保护DispatcherServlet不受这些细节影响。	|
|HandlerExceptionResolver	|解决异常的策略，可能会将它们映射到处理程序、HTML错误视图或其他目标。	|
|ViewResolver	|将处理程序返回的基于逻辑字符串的视图名称解析为用于呈现响应的实际视图	|
|LocaleResolver，LocaleContextResolver	|解析客户端正在使用的区域设置，可能还包括它们的时区，以便能够提供国际化视图。	|
|ThemeResolver	|解析您的Web应用程序可以使用的主题，提供个性化布局。	|
|MultipartResolver	|用于解析multi-part request。例如，文件上传。|
|FlashMapManager	|存储和检索“输入”和“输出”FlashMap，这些FlashMap可用于将属性从一个请求传递到另一个请求，通常是通过重定向。	|

### !!!Web MVC Config MVC配置
Applications 可以声明```Special Bean Types```中用于处理请求的Bean。DispatcherServlet 首先在WebApplicationContext中查找【Special Bean】，如果没有匹配的Bean类型，它将使用中```DispatcherServlet.properties```列出的默认Bean类型。

【MVC Config】使用Java或XML声明所需的bean，并提供更高级别的配置回调API来定制它。

Spring Boot 使用Java来配置 Spring MVC，并提供了许多额外方便的选项。

!!!```DispatcherServlet.properties```如下：
```
#DispatcherServlet策略接口的默认实现类.
#DispatcherServlet上下文中找不到匹配的Bean时作为备选方案.
#不会由开发人员自定义.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping,\
	org.springframework.web.servlet.function.support.RouterFunctionMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter,\
	org.springframework.web.servlet.function.support.HandlerFunctionAdapter


org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

### Servlet Config Servlet配置
在Servlet 3.0+环境中，您可以选择以编程方式配置Servlet容器，以替代方式或与web.xml文件结合使用。以下示例注册一个DispatcherServlet：
```
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```
WebApplicationInitializer是Spring MVC提供的接口，可确保检测到您的实现并将其自动用于初始化任何Servlet3容器。
通过WebApplicationInitializer命名方法 的抽象基类实现，AbstractDispatcherServletInitializer可以DispatcherServlet通过重写方法来指定servlet映射和DispatcherServlet配置位置，从而更加轻松地进行注册 。

### !!!Processing 请求处理过程
DispatcherServlet的请求处理过程如下：
+ 在WebApplicationContext被搜索并在请求的一个属性，在过程控制器和其它元件可以使用的约束。默认情况下，它是在DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE键下绑定的。
+ 语言环境解析器绑定到请求，以使流程中的元素解析在处理请求（呈现视图，准备数据等）时要使用的语言环境。如果不需要语言环境解析，则不需要语言环境解析器。
+ 主题解析器绑定到请求，以使诸如视图之类的元素确定要使用的主题。如果不使用主题，则可以将其忽略。
+ 如果指定多部分文件解析器，则将检查请求中是否有多部分。如果找到多部分，则将请求包装在中，以MultipartHttpServletRequest供流程中的其他元素进一步处理。有关多部分处理的更多信息，请参见Multipart Resolver。
+ 搜索适当的处理程序。如果找到处理程序，则将运行与该处理程序（预处理器，后处理器和控制器）关联的执行链，以准备要渲染的模型。或者，对于带注解的控制器，可以在（之内HandlerAdapter）呈现响应，而不返回视图。
+ 如果返回模型，则呈现视图。如果未返回任何模型（可能是由于预处理器或后处理器拦截了该请求，可能出于安全原因），则不会呈现任何视图，因为该请求可能已被满足。

BeanWebApplicationContext中声明的HandlerExceptionResolver用于解决在请求处理期间引发的异常。这些异常解析器允许定制逻辑以解决异常。

DispatcherServlet还支持last-modification-date Servlet API指定的的返回。
确定特定请求的最后修改日期的过程很简单：DispatcherServlet查找适当的处理程序映射并测试找到的处理程序是否实现了LastModified接口。如果是这样，则将接口long getLastModified(request)方法的值 LastModified返回给客户端。

### !!!Interception 拦截器
所有的```HandlerMapping```实现都支持处理程序拦截器，当您想要将特定功能应用于特定的请求 (例如检查主体)时，这些拦截器非常有用。

拦截器需要实现org.springframework work.web.servlet包的HandlerInterceptor接口，并实现三种方法：
+ preHandle(..)：在handler运行之前
+ postHandle(..)：在handler运行之后
+ afterCompletion(..)：在完整请求完成后

```preHandle```方法返回一个布尔值，可以使用此方法来中断或继续执行链的处理。
当方法返回true时，处理程序执行链继续继续执行。
当方法返回false时，```DispatcherServlet```会假定拦截器本身已经处理了请求，并且不会继续执行执行链中的其他拦截器和处理程序。

```postHandle```对于使用了```@ResponseBody```和```ResponseEntity```方法不太有用，这些方法的响应是在```HandlerAdapter```中和```postHandle```之前提交的，这意味着此时要对响应进行任何更改（例如添加额外的报头）都为时已晚。对于这样的场景，可以实现```ResponseBodyAdvices```并将其声明为一个```Controller Advice```bean，或者直接在```RequestMappingHandlerAdapter```上配置它。

### !!!Exceptions 异常处理
如果在请求映射期间发生异常或从请求处理程序抛出异常，则将DispatcherServlet委托给 一系列 HandlerExceptionResolver 以解决异常并提供替代处理，通常是错误响应。

下表列出了可用的HandlerExceptionResolver实现：
|HandlerExceptionResolver	|描述	|
|--	|--	|
|SimpleMappingExceptionResolver	|异常类名称和错误视图名称之间的映射。对于在浏览器应用程序中呈现错误页面很有用。	|
|DefaultHandlerExceptionResolver	|解决Spring MVC引发的异常，并将其映射到HTTP状态代码。	|
|ResponseStatusExceptionResolver	|使用@ResponseStatus注解解决异常，并根据注解中的值将它们映射到HTTP状态代码。	|
|ExceptionHandlerExceptionResolver	|通过调用@ExceptionHandler、@Controller或@ControllerAdvice中的方法来解决异常。	|

#### 解析器链
在Spring配置中，通过声明多个 HandlerExceptionResolver并设置它们的 order 属性来形成异常解析器链。
order属性越高，异常解析器的定位就越晚。

HandlerExceptionResolver可以返回以下ModelAndView：
+ 一个指向错误视图的ModelAndView。
+ 一个空的（empty）ModelAndView，如果异常是在解析程序中处理的。
+ null：如果该异常仍未解决，则供后续的解析器尝试；如果没有后续解析器，则允许异常冒泡到Servlet容器。
	
#### 容器错误
如果所有HandlerExceptionResolver都无法解决异常，或者响应状态设置为错误状态（即4xx，5xx），则Servlet容器可以使用HTML呈现默认错误页面。

要自定义容器的默认错误页面，可以web.xml在中声明错误页面映射。以下示例显示了如何执行此操作：
```
<error-page>
    <location>/error</location>
</error-page>
```

进而将其映射到@Controller：
```
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }
}
```

### !!!ViewResolver 视图解析器
Spring MVC定义了ViewResolver和View接口，使您可以在浏览器中呈现模型，而无需将您绑定到特定的视图技术。
ViewResolver 提供视图名称和实际视图之间的映射。
View在移交给特定的视图技术之前，先解决数据准备问题。

有关ViewResolver的具体实现：
|ViewResolver	|描述	|
|--	|--	|
|AbstractCachingViewResolver	|AbstractCachingViewResolver解析缓存视图实例子类，缓存可以提高某些视图技术的性能。	|
|UrlBasedViewResolver	|ViewResolver接口的简单实现，将逻辑视图名称直接解析为URL而没有显式映射定义。	|
|InternalResourceViewResolver	|UrlBasedViewResolver的子类，支持InternalResourceView（Servlet和JSP）。	|
|FreeMarkerViewResolver	|UrlBasedViewResolver的子类，支持FreeMarkerView。	|
|ContentNegotiatingViewResolver	|ViewResolver基于请求文件名或Accept头解析视图的接口的实现。	|
|BeanNameViewResolver	|ViewResolver在当前应用程序上下文中将视图名称解释为Bean名称的接口的实现。	|

#### 处理方式
可以通过声明多个解析器bean以及必要时通过设置order属性以指定顺序来链接视图解析器。
order属性越高，视图解析器在链中的定位就越晚。

对于一个ViewResolver，可以指定它返回null来指示找不到该视图。
但是，对于JSP和InternalResourceViewResolver，确定JSP是否存在的唯一方法是通过进行调度 RequestDispatcher。
因此，您必须始终将InternalResourceViewResolver View解析器的总体顺序配置为末尾。

配置**视图解析器**，就像将ViewResolver bean添加到Spring配置中一样简单。

#### 重定向
视图名称中的特殊前缀 redirect: 可以执行重定向。
UrlBasedViewResolver 和它的子类会识别需要重定向的指令。
视图名称的其余部分是重定向URL。

最终效果与控制器返回相同RedirectView，但是现在控制器本身可以根据逻辑视图名称进行操作。
逻辑视图名称（例如redirect:/myapp/some/resource）相对于当前Servlet上下文redirect:https://myhost.com/some/arbitrary/path 重定向，而名称（例如）重定向到绝对URL。

请注意，如果使用注解控制器方法@ResponseStatus，则注解值优先于设置的响应状态RedirectView。

#### 转发
您还可以forward:对视图名称使用特殊的前缀，这些视图名称最终由UrlBasedViewResolver和子类解析。
这将创建一个 InternalResourceView，并执行一个RequestDispatcher.forward()。

此前缀在InternalResourceViewResolver和 InternalResourceView（对于JSP）中没有用，但是如果您使用另一种视图技术，但仍然希望强制转发由Servlet / JSP引擎处理的资源，则该前缀很有用。

#### 内容协商
ContentNegotiatingViewResolver 不会解析视图本身，而是委托其他视图解析器，并选择类似于客户端请求的表示形式的视图。可以根据Accept标题或查询参数（例如"/path?format=pdf"）来确定表示形式。

### Locale 地域信息
Spring Web MVC框架支持国际化。
DispatcherServlet使您可以使用客户端的语言环境自动解析消息。
这是通过**LocaleResolver**对象完成的。

收到请求时，将DispatcherServlet查找语言环境解析器，如果找到一个，则尝试使用它来设置语言环境。
通过使用该RequestContext.getLocale() 方法，您始终可以检索由语言环境解析器解析的语言环境。

除了自动的语言环境解析之外，您还可以在处理程序映射上附加一个拦截器，以在特定情况下（例如，基于请求中的参数）更改语言环境。

语言环境解析器和拦截器在org.springframework.web.servlet.i18n程序包中定义，并以常规方式在应用程序上下文中配置。

Spring包含以下选择的语言环境解析器：
+ 时区
+ 标头解析器
+ Cookie解析器
+ 会话解析器
+ 区域拦截器

#### 时区
除了获取客户的语言环境外，了解其时区通常也很有用。
LocaleContextResolver接口扩展了LocaleResolver，使解析程序可以提供更丰富的内容LocaleContext，其中可能包含时区信息。

可以使用RequestContext.getTimeZone()方法获得用户的TimeZone信息。
Spring的注册的任何日期/时间Converter和Formatter对象 都会自动使用时区信息ConversionService。

#### Header解析器
Header解析器 检查accept-language客户端（例如，Web浏览器）发送的请求中的标头。
通常，此头字段包含客户端操作系统的语言环境。

请注意，此解析器不支持时区信息。

#### Cookie解析器
Cookie解析器 检查Cookie客户端上可能存在的，以查看是否 指定Locale或TimeZone。如果是这样，它将使用指定的详细信息。

通过使用此语言环境解析器的属性，您可以指定cookie的名称以及最长期限。

以下示例定义了一个CookieLocaleResolver：
```
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

    <property name="cookieName" value="clientlanguage"/>

    <!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
    <property name="cookieMaxAge" value="100000"/>

</bean>
```

#### Session解析器
Session解析器 从可能与用户的请求相关的会话检索Locale和TimeZone，。

CookieLocaleResolver，此策略将本地选择的语言环境设置存储在Servlet容器的中HttpSession。
结果，这些设置对于每个会话都是临时的，因此在每个会话结束时会丢失。

#### Locale拦截器
可以通过向HandlerMapping添加LocaleChangeInterceptor来启用语言环境的更改。
它可以检测请求中的参数，并通过调用LocaleResolver中的setLocale方法，更改应用程序上下文中相应地语言环境。

下一个示例显示，包含siteLanguage参数的所有资源都会更改语言环境。
例如，对URL的请求https://www.sf.net/home.view?siteLanguage=nl将站点语言更改为荷兰语。
以下示例显示如何拦截语言环境：
```
<bean id="localeChangeInterceptor"
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
    <property name="paramName" value="siteLanguage"/>
</bean>

<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/>

<bean id="urlMapping"
        class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="localeChangeInterceptor"/>
        </list>
    </property>
    <property name="mappings">
        <value>/**/*.view=someController</value>
    </property>
</bean>
```

### Themes 主题
略

### MultipartResolver Multipart解析器
MultipartResolver是一种从org.springframework.web.multipart软件包中提取的多部分请求（包括文件上传）的策略。
有一种基于Commons FileUpload的实现，另一种基于Servlet 3.0多部分请求解析。

要启用 MultipartResolver，需要MultipartResolver在DispatcherServletSpring配置中声明一个名为 multipartResolver 的bean。
在DispatcherServlet检测到它，并将其应用于传入请求。

当内容类型为multipart/form-data的POST被接收时，解析器解析该内容并把当前HttpServletRequest封装为MultipartHttpServletRequest，以提供对请求参数的访问。

#### Apache Commons FileUpload
要使用Apache Commons FileUpload，可以配置类型为CommonsMultipartResolver名称为multipartResolver的Bean。
还需要在类路径添加commons-fileupload依赖。

#### Servlet 3.0
要启用Servlet 3.0解析。需要如下配置：
+ 在Java中，在Servlet上设置一个MultipartConfigElement。
+ 在web.xml中，为Servlet声明添加一个"<multipart-config>"选项。

以下示例显示了如何在Servlet上设置一个MultipartConfigElement：
```
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }
}
```

### Logging 日志
DEBUG级别的日志被设计为紧凑，最少且人性化的。它侧重于高价值的信息，这些信息有用，而其他信息则仅在调试特定问题时才有用。

TRACE级别的日志记录通常遵循与DEBUG相同的原则，但可用于调试任何问题。此外，某些日志消息在TRACE和DEBUG上可能显示不同级别的详细信息。

#### 敏感数据
调试和跟踪日志记录可能会记录敏感信息。
默认情况下屏蔽请求参数和标头，需要通过设置 DispatcherServlet 的 enableLoggingRequestDetailson属性显式启用。

以下示例显示了如何通过使用Java配置来执行此操作：
```
public class MyInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return ... ;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return ... ;
    }

    @Override
    protected String[] getServletMappings() {
        return ... ;
    }

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setInitParameter("enableLoggingRequestDetails", "true");
    }
}
```

## Filters 过滤器
### Form Data
浏览器只能通过 HTTP GET或HTTP POST 提交数据，而非浏览器客户端还可以使用HTTP PUT，PATCH和DELETE方式提交数据。
Servlet API 的 ServletRequest.getParameter*() 方法仅支持HTTP POST的表单字段访问。

spring-web 模块提供FormContentFilter，可以拦截 content type 为 application/x-www-form-urlencoded 的HTTP PUT，PATCH和DELETE请求，并从请求主体读取表单数据，并包装ServletRequest使其通过ServletRequest.getParameter*()读取表单数据。

### Forwarded Headers
当请求通过代理（例如负载平衡器）进行处理时，主机、端口和协议可能会更改，这给从客户端角度指向正确的主机，端口和方案的链接创建带来了挑战。

RFC 7239定义了HTTP header Forwarded，可以用来提供有关原始请求的信息。
还有其他一些非标HTTP header，包括X-Forwarded-Host，X-Forwarded-Port， X-Forwarded-Proto，X-Forwarded-Ssl，和X-Forwarded-Prefix。

ForwardedHeaderFilter是一个Servlet过滤器，用于修改请求。
1基于Forwarded标头更改主机，端口和方案
2删除那些标头以消除进一步的影响。
这个过滤器用于包装请求，因此必须先于其他过滤器（例如RequestContextFilter，该过滤器应适用于修改后的请求，而不适用于原始请求）。

对于转发的标头，出于安全方面的考虑，应用程序无法知道标头是由代理添加的，还是由恶意客户端添加的。
这就是为什么应配置位于信任边界的代理以删除Forwarded 来自外部的不受信任的标头的原因。
还可以配置ForwardedHeaderFilter with removeOnly=true，在这种情况下，它会删除但不使用标题。

为了支持异步请求和错误请求，此过滤器应与DispatcherType.ASYNC和DispatcherType.ERROR关联。
如果使用Spring Framework的AbstractAnnotationConfigDispatcherServletInitializer，则会为所有调度类型自动注册所有过滤器。
如果通过web.xml注册该过滤器 或者在 Spring Boot启动FilterRegistrationBean时，一定要包括DispatcherType.ASYNC和 DispatcherType.ERROR。

### Shallow ETag
ShallowEtagHeaderFilter过滤器通过缓存写入响应的内容，并从它计算MD5哈希创建一个Shallow ETag。
客户端下一次发送时，将执行相同的操作，但还会将计算出的值与If-None-Match 请求标头进行比较，如果二者相等，则返回304（NOT_MODIFIED）。

这种策略可以节省网络带宽，但不能节省CPU，因为必须为每个请求计算完整响应。

为了支持异步请求，必须对此过滤器进行映射，DispatcherType.ASYNC以便过滤器可以延迟并成功生成ETag到最后一个异步调度的末尾。

### CORS
Spring MVC通过控制器上的注解为CORS配置提供了细粒度的支持。
但是，当与Spring Security一起使用时，我们建议使用CorsFilter，CorsFilter要在Spring Security的过滤器链之前订购的内置组件。

## !!!Annotated Controllers 控制器注解
Spring MVC提供了基于注解的编程模型，其中 @Controller和 @RestController 组件使用注解来表达请求映射，请求输入，异常处理等。
带注解的控制器具有灵活的方法签名，无需扩展基类或实现特定的接口。

以下示例显示了由注解定义的控制器：
```
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

### 控制器定义
您可以通过使用 WebApplicationContext 中定义的标准 Spring bean 来定义控制器bean。

@Controller原型允许自动检测，使用Spring的@Component对类路径中类和自动注册bean定义他们。
它还充当带注解类的构造型，表明其作为Web组件的作用。

要启用对此类@Controllerbean的自动检测，可以将组件扫描添加到Java配置中，如以下示例所示：
```
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

以下示例显示了与先前示例等效的XML配置：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>
    <!-- ... -->

</beans>
```

@RestController是一个组合式注解，它本身带有元注解，@Controller并@ResponseBody指示一个控制器。
该控制器的每个方法都继承了类型级别的@ResponseBody注解。

#### AOP代理
在某些情况下，可能需要在运行时用AOP代理装饰控制器。例如，直接在控制器上使用@Transactional。

对于控制器，建议使用基于类的代理，如果一个控制器必须实现一个接口，则需要显式配置基于类的代理。
例如，使用<tx:annotation-driven/>可以更改为<tx:annotation-driven proxy-target-class="true"/>，
使用 @EnableTransactionManagement可以更改为 @EnableTransactionManagement(proxyTargetClass = true)。

### !!!Request Mapping 请求映射
@RequestMapping注解将请求映射到控制器方法。
@RequestMapping具有各种属性设置，可以通过**URL，HTTP方法，请求参数，标头和媒体类型**进行匹配。
可以在类级别使用它来表示共享的映射，也可以在方法级别使用它来缩小到特定的端点映射。

@RequestMapping还有特定HTTP方法的快捷方式变体：
+ @GetMapping
+ @PostMapping
+ @PutMapping
+ @DeleteMapping
+ @PatchMapping

以下示例具有类型和方法级别的映射：
```
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

#### URI Pattern
@RequestMapping可以使用URL模式映射方法。这有两种方式：
+ PathPattern —与URL路径匹配的预解析模式，该路径也预解析为 PathContainer。该解决方案专为Web使用而设计，可有效处理编码和路径参数，并有效匹配。
+ AntPathMatcher —将字符串模式与字符串路径匹配。这是在Spring配置中还用于在类路径，文件系统和其他位置上选择资源的原始解决方案。它效率较低，并且字符串路径输入对于有效处理URL的编码和其他问题是一个挑战。

PathPattern是Web应用程序的推荐解决方案，它是Spring WebFlux的唯一选择。
在5.3之前的版本中，AntPathMatcher是Spring MVC中的唯一选择，并且继续是默认设置。PathPattern可以在MVC配置中启用 。

PathPattern支持与AntPathMatcher相同的模式语法。
另外，它还支持捕获模式，例如{*spring}，用于匹配路径末端的0个或更多路径段。
PathPattern还限制了**用于匹配多个路径段的用法，以使其仅在模式末尾才允许使用。

一些PathPattern示例模式：
+ "/resources/ima?e.png" -匹配路径段中的一个字符
+ "/resources/*.png" -匹配路径段中的零个或多个字符
+ "/resources/**" -匹配多个路径段
+ "/projects/{project}/versions" -匹配路径段并将其捕获为变量
+ "/projects/{project:[a-z]+}/versions" -使用正则表达式匹配并捕获变量

使用@PathVariable可以获取的URI变量。例如：
```
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

可以在类和方法级别声明URI变量，如以下示例所示：
```
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI变量会自动转换为适当的类型或TypeMismatchException 引发。
简单类型（int，long，Date，等）默认支持，你可以注册任何其它数据类型的支持。

#### Pattern比较
当多个模式与URL匹配时，必须选择最佳匹配。根据是否启用了已解析的`PathPattern's，使用以下方法之一完成此操作：
+ PathPattern.SPECIFICITY_COMPARATOR
+ AntPathMatcher.getPatternComparator(String path)

#### 后缀匹配
略

#### 后缀匹配 与 RFD
略

#### 请求媒体类型匹配
可以根据请求的Content-Type来缩小请求映射，如以下示例所示：
```
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

onsumes属性还支持否定表达式。例如，!text/plain表示以外的任何内容类型text/plain。

consumes属性可以在类级别声明共享属性。在类级使用时，方法级consumes属性将覆盖而不是扩展类级声明。

MediaType提供常用媒体类型（例如APPLICATION_JSON_VALUE和）的常量 APPLICATION_XML_VALUE。

#### 响应媒体类型匹配
可以根据Accept请求头和控制器方法生成的媒体类型列表来缩小请求映射，如以下示例所示：
```
@GetMapping(path = "/pets/{petId}", produces = "application/json") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

媒体类型可以指定字符集，支持否定表达式。例如， !text/plain表示除“文本/纯文本”之外的任何内容类型。

produces属性可以在类级别声明共享。在类级使用时，方法级produces属性将覆盖而不是扩展类级声明。

#### 参数和请求头匹配
可以根据请求参数条件来缩小请求映射。
可以测试是否存在请求参数（myParam），是否存在请求参数（!myParam）或特定值（myParam=myValue）。

以下示例显示如何测试特定值：
```
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

可以将其与请求标头条件一起使用，如以下示例所示：
```
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

#### HTTP HEAD, OPTIONS
@GetMapping（和@RequestMapping(method=HttpMethod.GET)）透明地支持HTTP HEAD以进行请求映射。控制器方法不需要更改。应用于的响应包装器javax.servlet.http.HttpServlet确保将Content-Length 标头设置为写入的字节数（实际上未写入响应）。

@GetMapping（和@RequestMapping(method=HttpMethod.GET)）被隐式映射到并支持HTTP HEAD。像处理HTTP GET一样处理HTTP HEAD请求，不同之处在于，不是写入正文，而是计算字节数并设置Content-Length 标头。

默认情况下，通过将Allow响应标头设置为所有@RequestMapping具有匹配URL模式的方法中列出的HTTP方法列表来处理HTTP OPTIONS 。

对于@RequestMapping不使用HTTP方法声明的情况，Allow标头设置为 GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS。控制器方法应该总是声明支持HTTP方法（例如，通过使用HTTP方法具体变体： @GetMapping，@PostMapping，及其他）。

您可以将@RequestMapping方法显式映射到HTTP HEAD和HTTP OPTIONS，但这在通常情况下不是必需的。

#### 自定义注解
Spring MVC支持将组合注解用于请求映射。

Spring MVC还支持带有自定义请求匹配逻辑的自定义请求映射属性。这是一个更高级的选项，它需要RequestMappingHandlerMapping对getCustomMethodCondition方法进行子类化 和覆盖，您可以在其中检查custom属性并返回您自己的方法RequestCondition。

#### 编程方式注册
可以以编程方式注册处理程序方法，这些方法可用于动态注册或高级案例。例如同一处理程序在不同URL下的不同实例。

下面的示例注册一个处理程序方法：
```
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }
}
```
+ 注入目标处理程序和控制器的处理程序映射。
+ 准备请求映射元数据。
+ 获取处理程序方法。
+ 添加注册。

### !!!Methods 方法
```@RequestMapping``` 注解的方法具有灵活的签名，可以从一系列的方法参数和返回值中进行选择。

#### !!!Method Arguments 参数
控制器方法支持的参数：

|控制器方法参数	|说明	|
|--	|--	|
|```javax.servlet.ServletRequest```， ```javax.servlet.ServletResponse```	|选择任何特定的请求或响应类型，例如，```ServletRequest```、```HttpServletRequest```或Spring的```MultipartRequest```, ```MultipartHttpServletRequest```。	|
|```javax.servlet.http.HttpSession```	|强制会话存在，这样的参数永远不会是空的。请注意，会话访问不是线程安全的，如果需要多个请求并发访问一个会话，请考虑将```RequestMappingHandlerAdapter```实例的```SynchronizeOnSession```标志设置为true。	|
|```javax.servlet.http.PushBuilder```	|用于程序化HTTP/2资源推送的Servlet 4.0推送构建器API。请注意，根据Servlet规范，如果客户端不支持HTTP/2特性，则注入的PushBuilder实例可以为空。	|
|```java.security.Principal```	|当前通过身份验证的用户。可能是特定的Principal实现类(如果已知)。	|
|```java.io.InputStream```，```java.io.Reader```	|用于访问Servlet API公开的原始请求正文。	|
|```java.io.OutputStream```， ```java.io.Writer```	|用于访问Servlet API公开的原始响应正文。	|
|```java.util.Locale```	|当前请求区域设置，由可用的最具体的```LocaleResolver```(实际上是配置的```LocaleResolver```或```LocaleContextResolver```)确定。	|
|```java.util.TimeZone``` + ```java.time.ZoneId```	|与当前请求关联的时区，由```LocaleContextResolver```确定。	|
|```WebRequest```， ```NativeWebRequest```	|访问请求参数以及请求和会话的属性，代替直接使用Servlet API。	|
|```HttpMethod```	|请求的HTTP方法	|
|**```@RequestParam```**	|用于访问Servlet请求参数，包括multipart参数，参数值将转换为声明的方法参数类型。**对于简单类型的参数，@RequestParam是可选的**。	|
|**```@ModelAttribute```**	|用于访问应用了数据绑定和验证的模型中的现有属性(如果不存在，则实例化)。使用@ModelAttribute是可选的。	|
|```@PathVariable```	|用于访问URI模板变量。	|
|```@MatrixVariable```	|用于访问URI路径段中的name-value变量。	|
|```@RequestPart```	|用于访问```multipart/form-data```请求中的内容，使用```HttpMessageConverter```转换部件的正文。	|
|**```@RequestHeader```**	|用于访问请求头。Header值转换为声明的方法参数类型。	|
|**```@CookieValue```**	|用于访问Cookies。Cookies值将转换为声明的方法参数类型。	|
|**```@RequestBody```**	|用于访问HTTP请求正文。正文内容通过使用```HttpMessageConverter```实现转换为声明的方法参数类型。。	|
|**```HttpEntity<B>```**	|用于访问请求头和正文。正文使用HttpMessageConverter进行转换。	|
|**```@RequestAttribute```**	|用于访问request属性	|
|**```@SessionAttribute```**	|用于访问任何session属性	|
|```java.util.Map```，```org.springframework.ui.Model```，```org.springframework.ui.ModelMap```	|用于访问在HTML控制器中使用并作为视图呈现的一部分公开给模板的模型。	|
|```RedirectAttributes```	|指定在重定向时使用的属性(即追加到查询字符串中)和在重定向后请求之前临时存储的闪存属性。	|
|```Errors```， ```BindingResult```	|用于访问命令对象（即，@ModelAttribute自变量）的验证和数据绑定中的错误，@RequestBody或访问a或自 @RequestPart变量的验证中的错误。您必须在经过验证的方法参数之后立即声明Errors或BindingResult参数。	|
|```SessionStatus``` + ```@SessionAttributes```		|为了标记表单处理完成，将触发清除通过类级@SessionAttributes注解声明的会话属性。	|
|```UriComponentsBuilder```		|用于准备相对于当前请求的主机，端口，方案，上下文路径以及servlet映射的文字部分的URL。	|
|**其他参数**	|如果方法参数与该表中的任何值都不匹配。如果参数为简单类型（由BeanUtils＃isSimpleProperty确定），则将其解析为```@RequestParam```。否则，将其解析为```@ModelAttribute```。	|

任何参数都不支持反应类型。

JDK8的```java.util.Optional```可以与具有```required```属性的注解(例如@RequestParam、@RequestHeader等)结合使用，等同于```required=false```。

#### !!!Return Values 返回值
下表描述了受支持的控制器方法返回值，所有返回值都支持反应性类型。

|控制器方法返回值	|描述	|
|--	|--	|
|**```@ResponseBody```**	|返回值通过```HttpMessageConverter```进行转换并写入响应。	|
|**```HttpEntity<B>```**， **```ResponseEntity<B>```**	|指定完整响应(包括HTTP头和正文)的返回值将通过HttpMessageConverter实现进行转换并写入响应。	|
|```HttpHeaders```	|用于返回有头而无正文的响应。	|
|**```String```**	|一个视图名称，将通过```ViewResolver```实现来解析，并与隐式模型一起使用。通过命令对象和@ModelAttribute方法确定	|
|```View```	|View实例以使用用于与所述隐式模型一起渲染。通过命令对象和@ModelAttribute方法确定。	|
|```java.util.Map```， ```org.springframework.ui.Model```	|要添加到隐式模型的属性，视图名称通过RequestToViewNameTranslator隐式确定	|
|```@ModelAttribute```	|要添加到模型的属性，视图名称通过RequestToViewNameTranslator隐式确定。	|
|```ModelAndView``` object	|要使用的视图和模型属性，以及响应状态（可选）	|
|**```void```**	|如果具有void返回类型(或NULL返回值)的方法，还具有ServletResponse、OutputStream参数或@ResponseStatus注释，则认为该方法已完全处理了响应。如果控制器进行了肯定的ETag或lastModified时间戳检查，则也是如此。如果上述情况都不成立，则void返回类型还可以为REST控制器指示“无响应正文”，或者为HTML控制器指示默认的视图名称选择。	|
|```DeferredResult<V>```	|从任何线程异步生成任何上述返回值-例如，由于某些事件或回调的结果。	|
|```Callable<V>```	|在SpringMVC管理的线程中异步产生上述任何返回值。	|
|```ListenableFuture<V>```， ```java.util.concurrent.CompletionStage<V>```， ```java.util.concurrent.CompletableFuture<V>```	|为了方便起见(例如，当底层服务返回其中之一时)，可以替代DeferredResult。	|
|```ResponseBodyEmitter```， ```SseEmitter```	|使用HttpMessageConverter实现异步发出要写入响应的对象流。也支持作为ResponseEntity的主体。	|
|```StreamingResponseBody```	|异步写入响应OutputStream。也支持作为ResponseEntity的主体	|
|```Reactive types```	|DeferredResult的替代方法，将多值流(例如，Flux、Observable)收集到列表中。对于流场景(例如，文本/事件流、应用程序/json+流)，改为使用SseEmitter和ResponseBodyEmitter，其中ServletOutputStream阻塞I/O在Spring MVC管理的线程上执行，并在每次写入完成时施加背压。。	|
|```其他返回值```	|任何返回值如果与此表中任何较早的值都不匹配，并且是字符串或空，则被视为视图名称。	|

#### Type Conversion 类型转换
一些控制器方法的参数是以String为基础的输入（如 @RequestParam，@RequestHeader，@PathVariable，@MatrixVariable，和@CookieValue），如果参数被声明为非String类型，则可以进行类型转换。

默认情况下，已经支持简单的类型（int，long，Date等）。
可以自定义类型转换，通过使用一个WebDataBinder，或者通过注册Formatters与FormattingConversionService来实现。

#### Matrix变量
矩阵变量，也可以称为URI路径参数。

矩阵变量可以出现在任何路径段中，每个变量用分号分隔，多个值用逗号分隔（例如/cars;color=red,green;year=2012）。
也可以通过重复的变量名称（例如color=red;color=green;color=blue）来指定多个值 。

如果希望URL包含矩阵变量，则控制器方法的请求映射必须使用URI变量来屏蔽该变量内容，并确保可以成功地匹配请求，与矩阵变量的顺序和状态无关。

以下示例使用矩阵变量：
```
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

鉴于所有路径段都可能包含矩阵变量，因此有时需要消除矩阵变量应位于哪个路径变量的歧义。
以下示例显示了如何做到这一点：
```
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

可以将矩阵变量定义为可选变量，并指定默认值，如以下示例所示：
```
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

要获取所有矩阵变量，可以使用MultiValueMap，如以下示例所示：
```
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

**如果需要启用矩阵变量：**
在Java配置中，设置一个 UrlPathHelper，并配置removeSemicolonContent=false。
在XML名称空间中，设置<mvc:annotation-driven enable-matrix-variables="true"/>。

#### @RequestParam
使用@RequestParam将Servlet请求参数绑定到控制器中的方法参数。

以下示例显示了如何执行此操作：
```
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

默认情况下，使用此注解的方法参数是必需的。可以将@RequestParam注解的required设置为 false或使用java.util.Optional包装器声明参数来指定方法参数是可选的。

如果方法参数类型不是String，则类型将自动转换。

将参数类型声明为**数组或列表**，可以为同一参数名称解​​析多个参数值。

如果将@RequestParam注解声明为Map<String, String>或 MultiValueMap<String, String>，而未在注解中指定参数名称，则将使用每个给定参数名称的请求参数值填充映射。

@RequestParam是可选的。**默认情况下，任何简单值类型的参数且未由任何其他参数解析器解析，就如同使用注解一样@RequestParam。**

#### @RequestHeader
使用@RequestHeader注解将请求头绑定到控制器中的方法参数。

考虑以下的请求：
```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

以下示例获取Accept-Encoding和Keep-Alive标头的值：
```
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

如果目标方法的参数类型不是String，则将自动应用类型转换。

当@RequestHeader注解上的使用Map<String, String>， MultiValueMap<String, String>或 HttpHeaders参数，则填充有所有header值。

#### @CookieValue
您可以使用@CookieValue注解将HTTP cookie的值绑定到控制器中的方法参数。

考虑带有以下cookie的请求：
```
JSESSIONID = 415A4AC178C59DACE0B2C9CA727CDD84
```

以下示例显示如何获取cookie值：
```
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

如果目标方法的参数类型不是String，则类型转换将自动应用。

#### @ModelAttribute
可以在方法参数上使用@ModelAttribute注解，从模型访问属性，或者将其实例化（如果不存在）。
model属性还覆盖了名称与字段名称匹配的HTTP Servlet请求参数中的值。
这称为数据绑定，它使您不必处理解析和转换单个查询参数和表单字段的工作。

以下示例显示了如何执行此操作：
```
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) {
	
} 
```

虽然通常使用模型来用属性填充模型，但另一种替代方法是依赖于Converter<String, T>URI路径变量约定的组合。
在下面的示例中，模型属性名称 account匹配URI路径变量account，并且Account通过将String帐号传递给已注册的来加载Converter<String, Account>：
```
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

数据绑定可能导致错误。默认情况下，引发一个BindException。
要检查controller方法中的此类错误，可以在@ModelAttribute后面添加一个BindingResult参数：
```
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

在某些情况下，您可能希望访问没有数据绑定的模型属性。
可以将Model注入到控制器中并直接访问它，或者设置@ModelAttribute(binding=false)，如以下示例所示：
```
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) { 
    // ...
}
```

您可以在数据绑定之后通过添加javax.validation.Valid注解或Spring的@Validated注解（ Bean Validation和 Spring validate）自动应用验证 。以下示例显示了如何执行此操作：
```
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

@ModelAttribute是可选的。**默认情况下，任何不是简单值类型且未被其他任何参数解析器解析的参数都将被视为@ModelAttribute。**

#### @SessionAttributes
@SessionAttributes用于在请求之间的HTTP Servlet会话中存储模型属性。
**它是类型级别的注解，用于声明特定控制器使用的会话属性。**
这通常列出应透明地存储在会话中以供后续访问请求的模型属性的名称或模型属性的类型。

以下示例使用@SessionAttributes注解：
```
@Controller
@SessionAttributes("pet") 
public class EditPetForm {
    // ...
}
```

在第一个请求上，将名称为的模型属性pet添加到模型时，该属性会自动提升到HTTP Servlet会话并保存在该会话中。它会一直保留在那里，直到另一个控制器方法使用SessionStatus方法参数来清除存储，如以下示例所示：
```
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
            status.setComplete(); 
            // ...
        }
    }
}
```

#### @SessionAttribute
如果您需要访问全局存在（即，在控制器外部，例如，通过过滤器）可能存在或不存在的预先存在的会话属性，则可以@SessionAttribute在方法参数上使用注解，如下所示示例显示：
```
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

对于用例需要添加或删除会话属性，可以考虑注入 org.springframework.web.context.request.WebRequest或 javax.servlet.http.HttpSession到控制器的方法。

#### @RequestAttribute
可以使用@RequestAttribute注解来访问先前创建的预先存在的请求属性（例如，通过ServletFilter 或HandlerInterceptor）：
```
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```

#### Redirect变量
默认情况下，所有模型属性均被视为在重定向URL中作为URI模板变量公开。
在其余属性中，那些属于原始类型或原始类型的集合或数组的属性会自动附加为查询参数。

如果专门为重定向准备了模型实例，则将原始类型属性作为查询参数附加可能是理想的结果。但是，在带注解的控制器中，模型可以包含为渲染目的添加的其他属性（例如，下拉字段值）。为避免此类属性出现在URL中的可能性，@RequestMapping方法可以声明类型的自变量，RedirectAttributes并使用它来指定要提供给的确切属性RedirectView。如果该方法确实重定向，RedirectAttributes则使用的内容。否则，将使用模型的内容。

在RequestMappingHandlerAdapter提供了一个名为标志 ignoreDefaultModelOnRedirect，你可以用它来表示默认的内容 Model不应该，如果一个控制器方法重定向使用。相反，控制器方法应该声明一个类型的属性，RedirectAttributes或者如果没有声明，则不应将任何属性传递给RedirectView。MVC命名空间和MVC Java配置都将此标志设置为false，以保持向后兼容性。但是，对于新应用程序，我们建议将其设置为true。

请注意，展开重定向URL时，本请求中的URI模板变量会自动变为可用，并且您无需通过Model或显式添加它们RedirectAttributes。

以下示例显示了如何定义重定向：
```
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

#### Flash变量
Flash属性为一个请求提供了一种存储打算在另一个请求中使用的属性的方式。重定向时最常需要此功能，例如Post-Redirect-Get模式。Flash属性在重定向之前（通常在会话中）被临时保存，以便在重定向之后可供请求使用，并立即被删除。

Spring MVC有两个主要的抽象来支持Flash属性。FlashMap用于保存Flash属性，而FlashMapManager用于存储，检索和管理 FlashMap实例。

Flash属性支持始终处于“打开”状态，无需显式启用。但是，如果不使用它，则永远不会导致HTTP会话创建。在每个请求上，都有一个“输入” FlashMap，该属性具有从前一个请求（如果有）传递过来的属性，而“输出”则FlashMap具有为后续请求保存的属性。FlashMap 可以通过Spring中的静态方法从Spring MVC中的任何位置访问这两个实例 RequestContextUtils。

带注解的控制器通常不需要FlashMap直接使用。取而代之的是， @RequestMapping方法可以接受类型的参数，RedirectAttributes并使用它为重定向方案添加Flash属性。通过添加的Flash属性将 RedirectAttributes自动传播到“输出” FlashMap。同样，重定向后，来自“输入”的属性FlashMap会自动添加到 Model服务于目标URL的控制器的。

#### Multipart
当一个MultipartResolver启用后，应用程序可以对POST的multipart/form-data内容进行解析。
以下示例访问一个常规表单字段和一个上载文件：
```
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```
将参数类型声明为List<MultipartFile>，则可以解析相同参数名称的多个文件。
如果将@RequestParam注解声明为Map<String, MultipartFile>或 MultiValueMap<String, MultipartFile>，而未在注解中指定参数名称，则将使用每个给定参数名称的多部分文件来填充映射。

还可以将数据绑定到对象的一部分。例如，前面示例中的表单字段和文件可以是表单对象上的字段，如以下示例所示：
```
class MyForm {

    private String name;

    private MultipartFile file;

    // ...
}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

**在RESTful服务方案中，也可以从非浏览器客户端提交多部分请求。**
以下示例显示了带有JSON的文件：
```
POST /someUrl
Content-Type: multipart/mixed

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="meta-data"
Content-Type: application/json; charset=UTF-8
Content-Transfer-Encoding: 8bit

{
    "name": "value"
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="file-data"; filename="file.properties"
Content-Type: text/xml
Content-Transfer-Encoding: 8bit
... File Data ...
```

可以通过@RequestParam访问参数“meta-data”。在使用HttpMessageConverter将multipart转换后，可以使用注解@RequestPart来访问"file-data"：
```
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

可以将@RequestPart与@Validated注解结合使用或一起使用，这两种注解都会导致应用标准Bean验证。
默认情况下，验证错误会导致MethodArgumentNotValidException，并变成400（BAD_REQUEST）响应。
可以通过Errors或BindingResult参数在控制器内本地处理验证错误，如以下示例所示：
```
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```


#### !!!@RequestBody
使用@RequestBody注解，通过HttpMessageConverter读取请求体并反序列化到一个Object。
下面的示例使用一个@RequestBody参数：
```
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

@RequestBody可以与java的@Valid注解（javax.validation.Valid）或Spring的@Validated注解结合使用，这两种注解都会应用标准Bean验证。
默认情况下，验证错误会导致MethodArgumentNotValidException，并变成400（BAD_REQUEST）响应。
或者，您可以通过Errors或BindingResult参数在控制器内本地处理验证错误，如以下示例所示：
```
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```

#### !!!HttpEntity
HttpEntity与@RequestBody大致相同，但基于公开请求标头和正文的容器对象。
以下显示了一个示例：
```
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```

#### !!!@ResponseBody
@ResponseBody注解在方法上使用，通过HttpMessageConverter将返回序列化为响应主体。
以下显示了一个示例：
```
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```
@ResponseBody在类级别也受支持，在这种情况下，它由所有控制器方法继承。

#### !!!ResponseEntity
ResponseEntity使用和@ResponseBody类似，但带有状态和标题。例如：
```
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

使用ResponseEntity<Resource>进行文件下载。例如：
```
@GetMapping("/files/{filename:.+}")
@ResponseBody
public ResponseEntity<Resource> serveFile(@PathVariable String filename) {

	Resource file = storageService.loadAsResource(filename);
	return ResponseEntity
			.ok()
			.header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getFilename() + "\"")
			.body(file);
}

public Resource loadAsResource(String filename) {
		try {
			Path file = load(filename);
			Resource resource = new UrlResource(file.toUri());
			if(resource.exists() || resource.isReadable()) {
				return resource;
			}
			else {
				throw new StorageFileNotFoundException("Could not read file: " + filename);

			}
		} catch (MalformedURLException e) {
			throw new StorageFileNotFoundException("Could not read file: " + filename, e);
		}
	}
```

使用ResponseEntity<Resource>进行文件下载。例如：
```
@RequestMapping(value = "/file", method = RequestMethod.GET)
	public ResponseEntity<Resource> downloadFile() {
		ByteArrayOutputStream bos = null;
		String filename = "测试.xlsx";
		try {
			Workbook workbook = createExcel();
			bos = new ByteArrayOutputStream();
			workbook.write(bos);
			workbook.close();

			HttpHeaders headers = new HttpHeaders();
			headers.add("Cache-Control", "no-cache, no-store, must-revalidate");
			headers.add("Pragma", "no-cache");
			headers.add("Expires", "0");
			headers.add("charset", "utf-8");
			//设置下载文件名
			filename = URLEncoder.encode(filename, "UTF-8");
			headers.add("Content-Disposition", "attachment;filename=\"" + filename + "\"");

			Resource resource = new InputStreamResource(new ByteArrayInputStream(bos.toByteArray()));
			return ResponseEntity.ok().headers(headers).contentType(MediaType.parseMediaType("application/x-msdownload")).body(resource);
		} catch (IOException e) {
			if (null != bos) {
				try {
					bos.close();
				} catch (IOException e1) {
					e1.printStackTrace();
				}
			}
		}
		return null;
	}
```

可以绕过消息转换并直接流式传输到响应（用于文件下载）。可以使用StreamingResponseBody 返回值类型来执行此操作：
```
@GetMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```
可以将StreamingResponseBody用作ResponseEntity的body，来自定义响应的状态和标题。

#### Jackson JSON
Spring提供了对 Jackson JSON库的支持。

#### JSON视图
Spring MVC为Jackson的序列化视图提供了内置支持，该视图支持只呈现一个Object所有字段中的一部分。
要将其与 @ResponseBody或ResponseEntity 一起使用，可以使用Jackson的 @JsonView注解来激活序列化视图类。
如以下示例所示：
```
public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```
```
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}
```

@JsonView允许一组视图类，但是每个控制器方法只能指定一个。如果需要激活多个视图，则可以使用复合界面。

如果要以编程方式执行上述操作，而不是声明@JsonView注解，则将返回值包装为MappingJacksonValue并用于提供序列化视图：
```
@RestController
public class UserController {

    @GetMapping("/user")
    public MappingJacksonValue getUser() {
        User user = new User("eric", "7!jd#h23");
        MappingJacksonValue value = new MappingJacksonValue(user);
        value.setSerializationView(User.WithoutPasswordView.class);
        return value;
    }
}
```

对于依赖视图的控制器，可以将序列化视图类添加到模型中，如以下示例所示：
```
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

### Model 模型
您可以使用@ModelAttribute注解：
+ 在方法中的方法参数@RequestMapping上创建或访问Object来自模型的，并通过绑定到请求 WebDataBinder。
+ 作为@Controller或@ControllerAdvice类中的方法级注解，可在任何@RequestMapping方法调用之前帮助初始化模型。
+ @RequestMapping标记其返回值的方法是模型属性。

控制器可以有多种@ModelAttribute方法。
@RequestMapping在同一个控制器中，所有这些方法均在方法之前被调用。
@ModelAttribute 也可以通过跨控制器共享一种方法@ControllerAdvice。

@ModelAttribute方法具有灵活的方法签名。它们支持许多与@RequestMapping方法相同的参数，除了@ModelAttribute自身或与请求主体相关的任何东西。

以下示例显示了一种@ModelAttribute方法：
```
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

以下示例仅添加一个属性：
```
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

还可以@ModelAttribute在方法上用作方法级注解@RequestMapping，在这种情况下，方法的返回值将@RequestMapping解释为模型属性。通常不需要这样做，因为这是HTML控制器的默认行为，除非返回值是a String，否则它将被解释为视图名称。 @ModelAttribute还可以自定义模型属性名称，如以下示例所示：
```
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```

### DataBinder 数据绑定
@Controller或@ControllerAdvice类可以有@InitBinder注解的方法，这个方法用于初始化WebDataBinder实例。

WebDataBinder实例的功能如下：
+ 将请求参数（即表单或查询数据）绑定到模型对象。
+ 将基于字符串的请求值（例如请求参数，路径变量，标头，Cookie等）转换为控制器方法参数的目标类型。
+ String呈现HTML表单时，将模型对象值格式化为值。

@InitBinder方法可以注册控制器特异性java.beans.PropertyEditor或SpringConverter和Formatter组件。
此外，您可以使用 MVC配置 在全局共享中注册Converter和Formatter键入FormattingConversionService。

@InitBinder@RequestMapping除了@ModelAttribute（命令对象）参数外，方法还支持许多与方法相同的参数。
通常，它们使用一个WebDataBinder参数（用于注册）和一个void返回值进行声明。
以下清单显示了一个示例：
```
@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

或者，当您Formatter通过shared使用基于设置时 FormattingConversionService，可以重新使用相同的方法并注册特定于控制器的Formatter实现，如以下示例所示：
```
@Controller
public class FormController {

    @InitBinder 
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```

### !!!Exceptions 异常
@Controller和@ControllerAdvice类可以声明使用@ExceptionHandler注解的方法，用于处理控制器方法的异常。

如以下示例所示：
```
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

异常可以匹配顶级异常（即IOException），也可以与顶级序异常的子类异常匹配（例如IllegalStateException）。

对于匹配的异常类型，最好将目标异常声明为方法参数。当多个异常方法匹配时，根源异常匹配通常比原因异常匹配更可取。

另外，注解声明可以缩小异常类型以使其匹配，如以下示例所示：
```
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(IOException ex) {
    // ...
}
```

可以使用带有非常通用的参数签名的特定异常类型的列表，如以下示例所示：
```
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(Exception ex) {
    // ...
}
```

我们通常建议您在参数签名中尽可能具体，以减少根类型和原因异常类型之间不匹配的可能性。
考虑将多重匹配方法分解为单个@ExceptionHandler 方法，每个方法都通过其签名匹配单个特定的异常类型。

在多种@ControllerAdvice安排中，我们建议声明@ControllerAdvice优先级并以相应顺序声明您的主根异常映射。
尽管根异常匹配优先于原因，但这是在给定控制器或@ControllerAdvice类的方法之间定义的。
这意味着优先级较高的@ControllerAdviceBean上的原因匹配优于优先级 较低的@ControllerAdviceBean上的任何匹配（例如，根） 。

最后但并非最不重要的一点是，@ExceptionHandler方法实现可以选择通过以原始形式重新抛出异常来退出处理给定异常实例。
在仅对根级别匹配或无法静态确定的特定上下文中的匹配感兴趣的情况下，这很有用。
重新抛出的异常会在其余的解决方案链中传播，就像给定的@ExceptionHandler方法最初不会匹配一样。

**Spring MVC对@ExceptionHandler的支持使用了HandlerExceptionResolver机制，建立在DispatcherServlet级别上。**

#### !!!Method Arguments 方法参数
@ExceptionHandler 方法支持以下参数：

|方法参数	|描述	|
|--	|--	|
|异常类型	|用于访问引发的异常。	|
|HandlerMethod	|用于访问引发异常的控制器方法。	|
|WebRequest， NativeWebRequest	|对请求参数以及请求和会话属性的一般访问，而无需直接使用Servlet API。	|
|javax.servlet.ServletRequest， javax.servlet.ServletResponse	|选择任何特定的请求或响应类型（例如ServletRequest或 HttpServletRequest或Spring的MultipartRequest或MultipartHttpServletRequest）。	|
|javax.servlet.http.HttpSession	|强制会话的存在。结果，这种论据永远不会null。
请注意，会话访问不是线程安全的。考虑将RequestMappingHandlerAdapter实例的synchronizeOnSession标志设置 为true是否允许多个请求同时访问会话。	|
|java.security.Principal	|当前经过身份验证的用户-可能是特定的Principal实现类（如果已知）。	|
|HttpMethod	|请求的HTTP方法。	|
|java.util.Locale	|当前请求的语言环境，取决于最具体的LocaleResolver可用语言（实际上是配置的LocaleResolver或）LocaleContextResolver。	|
|java.util.TimeZone， java.time.ZoneId	|与当前请求关联的时区，由决定LocaleContextResolver。	|
|java.io.OutputStream， java.io.Writer	|用于访问原始响应主体，如Servlet API所公开。	|
|java.util.Map，org.springframework.ui.Model，org.springframework.ui.ModelMap	|用于访问模型以进行错误响应。永远是空的。	|
|RedirectAttributes	|指定在重定向的情况下要使用的属性（将附加到查询字符串中）和flash属性，这些属性将临时存储直到重定向后的请求。	|
|@SessionAttribute	|与访问由于类级@SessionAttributes声明而存储在会话中的模型属性相反，用于访问任何会话属性。	|
|@RequestAttribute	|用于访问请求属性。	|

#### !!!Return Values 返回值
@ExceptionHandler 方法支持以下返回值：

|返回值	|描述	|
|--	|--	|
|@ResponseBody	|返回值通过HttpMessageConverter实例转换并写入响应。	|
|HttpEntity<B>， ResponseEntity<B>	|返回值指定完整的响应（包括HTTP标头和正文）将通过HttpMessageConverter实例转换并写入响应。	|
|String	|一个视图名称，将通过ViewResolver实现来解析，并与隐式模型一起使用-通过命令对象和@ModelAttribute方法确定。该处理程序方法还可以通过声明一个Model 参数（如前所述）以编程方式丰富模型。	|
|View	|View实例以使用用于与所述隐式模型一起渲染-通过命令对象和确定@ModelAttribute方法。该处理程序方法还可以通过声明一个自Model变量（如前所述）以编程方式丰富模型。	|
|java.util.Map， org.springframework.ui.Model	|要添加到隐式模型的属性，其视图名称是通过隐式确定的RequestToViewNameTranslator。	|
|@ModelAttribute	|要添加到模型的属性，其视图名称通过隐式确定RequestToViewNameTranslator。	|
|ModelAndView object	|要使用的视图和模型属性，以及响应状态（可选）。	|
|void	|方法返回void（或null返回值）被认为已经完全处理的响应，如果它也具有ServletResponse一个OutputStream参数，或@ResponseStatus注解。如果控制器进行了肯定ETag或lastModified时间戳检查，则情况也是如此 	|
|任何其他返回值	|如果返回值与上述任何一个都不匹配且不是简单类型，则默认情况下会将其视为要添加到模型的模型属性。如果是简单类型，则仍然无法解析。	|

#### REST API exceptions RESTAPI异常
REST服务的常见要求是在响应正文中包含错误详细信息。

Spring框架不会自动执行此操作，因为响应主体中错误详细信息的表示是特定于应用程序的。

@RestController可以**使用@ExceptionHandler带有ResponseEntity返回值的方法来设置响应的状态和主体**。
也可以在@ControllerAdvice类中声明此类方法以将其全局应用。

实现在响应主体中具有错误详细信息的全局异常处理的应用程序，可以考虑继承 ResponseEntityExceptionHandler，它提供对Spring MVC引发的异常的处理，并提供用于自定义响应主体的钩子。
要使用此功能，请创建一个的子类 ResponseEntityExceptionHandler，用对其进行注解@ControllerAdvice，覆盖必要的方法，然后将其声明为Spring bean。

### !!!Controller Advice
通常@ExceptionHandler，@InitBinder和@ModelAttribute方法适用于@Controller声明的类。
如果要使此类方法应用于全局，则可以在带有@ControllerAdvice或@RestControllerAdvice的类中声明它们。

@ControllerAdvice带有注解@Component，这意味着可以通过组件扫描将此类注册为Spring Bean。
@RestControllerAdvice是用@ControllerAdvice和@ResponseBody注解组合的注解。
从本质上讲@ExceptionHandler方法是通过消息转换（视图解析或模板渲染）呈现给响应体的。

默认情况下，@ControllerAdvice方法适用于每个请求（即所有控制器），可以通过使用批注上的属性将其范围缩小到控制器的子集，如以下示例所示：
```
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

## Functional Endpoints 函数节点
Spring Web MVC包含WebMvc.fn，这是一个轻量级的函数编程模型，其中的函数用于路由和处理请求，而契约则是为不变性而设计的。

它是基于注解的编程模型的替代方案，但可以在同一DispatcherServlet上运行。

略

## URI Links URI链接
Spring框架中可用于URI的各种选项。

### UriComponents
UriComponentsBuilder 有助于从带有变量的URI模板构建URI，如以下示例所示：
```
UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")  
        .queryParam("q", "{q}")  
        .encode() 
        .build(); 

URI uri = uriComponents.expand("Westin", "123").toUri();  
```

可以将前面的示例合并为一个链，并通过进行缩短buildAndExpand，如以下示例所示：
```
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("Westin", "123")
        .toUri();
```

可以通过直接转到URI（这意味着编码）来进一步缩短它，如以下示例所示：
```
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

可以使用完整的URI模板进一步缩短它，如以下示例所示：
```
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}?q={q}")
        .build("Westin", "123");
```

### UriBuilder
UriComponentsBuilder实现了UriBuilder。

使用UriBuilder可以创建一个UriBuilderFactory。

UriBuilderFactory 与 UriBuilder提供一个可插入的机构以从URI模板，基于共享的配置，构建的URI诸如基本URL等。

可以为RestTemplate和WebClient配置一个UriBuilderFactory，从而使用自定义的URI。

DefaultUriBuilderFactory是UriBuilderFactory使用UriComponentsBuilder并公开共享配置选项的默认实现。

以下示例显示了如何配置RestTemplate：
```
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);
```

以下示例配置了WebClient：
```
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

另外，您也可以直接使用DefaultUriBuilderFactory。
它类似于使用， UriComponentsBuilder但不是静态工厂方法，它是一个包含配置和首选项的实际实例，如以下示例所示：
```
String baseUrl = "https://example.com";
DefaultUriBuilderFactory uriBuilderFactory = new DefaultUriBuilderFactory(baseUrl);

URI uri = uriBuilderFactory.uriString("/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

### URI编码
UriComponentsBuilder 在两个级别公开编码选项：
+ UriComponentsBuilder＃encode（）：首先对URI模板进行预编码，然后在扩展时严格对URI变量进行编码。
+ UriComponents＃encode（）：扩展URI变量后，对URI组件进行编码。

这两个选项都使用八位转义字节替换非ASCII和非法字符。但是，第一个选项还会替换出现在URI变量中的具有保留含义的字符。
例如“;”，这在路径上是合法的，但具有保留的含义。第一个选项代替“;” URI变量中带有“％3B”，但URI模板中没有。第二个选项永远不会替换“;”，因为它是路径中的合法字符。
在大多数情况下，第一个选项可能会产生预期的结果，因为它将URI变量视为要完全编码的不透明数据，而第二个选项仅在URI变量有意包含保留字符的情况下才有用。

以下示例使用第一个选项：
```
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("New York", "foo+bar")
        .toUri();

// Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```

可以通过直接转到URI（这意味着编码）来缩短前面的示例，如以下示例所示：
```
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .build("New York", "foo+bar")
```

可以使用完整的URI模板进一步缩短它，如以下示例所示：
```
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}?q={q}")
        .build("New York", "foo+bar")
```

在WebClient与RestTemplate扩大和编码URI通过内部模板UriBuilderFactory策略。两者都可以使用自定义策略进行配置。如下例所示：
```
String baseUrl = "https://example.com";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl)
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

// Customize the RestTemplate..
RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);

// Customize the WebClient..
WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

DefaultUriBuilderFactory实现在UriComponentsBuilder内部使用以扩展和编码URI模板。
作为工厂，它提供了一个位置，可以根据以下一种编码模式来配置编码方法：
+ TEMPLATE_AND_VALUES：使用UriComponentsBuilder#encode()，对应于先前列表中的第一个选项，对URI模板进行预编码，并在扩展时严格编码URI变量。
+ VALUES_ONLY：不对URI模板进行编码，而是在将URI变量UriUtils#encodeUriUriVariables扩展到模板之前对其进行严格编码。
+ URI_COMPONENT：使用UriComponents#encode()，对应于先前列表中的第二个选项，在扩展URI变量后使用URI组件值进行编码。
+ NONE：未应用编码。

### 相对请求
使用ServletUriComponentsBuilder来创建**相对于当前请求**的URI，如以下示例所示：
```
HttpServletRequest request = ...

// 省略 host, scheme, port, path and query string...
ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}").build()
        .expand("123")
        .encode();
```

可以创建**相对于上下文路径**的URI，如以下示例所示：
```
// 省略 host, port and context path...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromContextPath(request)
        .path("/accounts").build()
```

可以创建**与Servlet相关**的URI（例如/main/*），如以下示例所示：
```
// 省略 host, port, context path, and Servlet prefix...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
        .path("/accounts").build()
```

### 控制器连接
Spring MVC提供了一种链接到控制器方法的的机制。例如，以下MVC控制器允许创建链接：
```
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

    @GetMapping("/bookings/{booking}")
    public ModelAndView getBooking(@PathVariable Long booking) {
        // ...
    }
}
```

使用 MvcUriComponentsBuilder 可以通过按名称引用方法来准备链接。如以下示例所示：
```
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

在前面的示例中，我们提供了实际的方法参数值（在本例中为long值21：）用作路径变量并插入到URL中。此外，我们提供了值，42以填充所有剩余的URI变量，例如hotel从类型级别请求映射继承的变量。如果该方法具有更多参数，则可以为URL不需要的参数提供null。通常，只有@PathVariable和@RequestParam参数与构造URL有关。

还有其他使用方法MvcUriComponentsBuilder。例如，您可以使用类似于代理的测试技术来避免按名称引用控制器方法，如以下示例所示（该示例假定静态导入MvcUriComponentsBuilder.on）：
```
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

前面的示例在中使用静态方法MvcUriComponentsBuilder。
实际上，它们依靠ServletUriComponentsBuilder获取当前请求的方案，主机，端口，上下文路径和Servlet路径准备基本URL。

### 视图链接
在Thymeleaf，FreeMarker或JSP之类的视图中，可以通过引用每个请求映射的隐式或显式分配的名称来构建到带注解的控制器的链接。

以下示例：
```
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity<PersonAddress> getAddress(@PathVariable String country) { ... }
}
```

给定前面的控制器，您可以按照以下步骤准备来自JSP的链接：
```
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```

## !!!Asynchronous Requests 异步请求
Spring MVC与Servlet3.0异步请求处理进行了广泛的集成：
+ 控制器方法中的```DeferredResult```和```Callable```返回值，并为单个异步返回值提供基本支持。
+ 控制器可以流式传输多个值，包括```SSE```和```原始数据```。
+ 控制器可以使用反应式客户端并返回```reactive types```以进行响应处理。

### DeferredResult
一旦在Servlet容器中启用了异步请求处理功能，控制器方法就可以使用DeferredResult来包装任何受支持的控制器方法返回值。
如以下示例所示：
```
@GetMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// From some other thread...
deferredResult.setResult(result);
```

控制器可以从另一个线程异步生成返回值，例如，响应外部事件（JMS消息），计划任务或其他事件。

### Callable
控制器可以使用java.util.concurrent.Callable来包装任何受支持的返回值，如以下示例所示：
```
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}
```

然后，可以通过配置 TaskExecutor 来运行给定任务来获取返回值 。

### 调用流程
Servlet异步请求处理的非常简洁的概述：
+ ServletRequest可以通过调用将A置于异步模式request.startAsync()。这样做的主要作用是可以退出Servlet（以及所有过滤器），但是响应保持打开状态，以便以后完成处理。
+ 要调用request.startAsync()的回报AsyncContext，您可以使用超过异步处理的进一步控制。例如，它提供了dispatch与Servlet API中的转发类似的方法，不同之处在于，它允许应用程序恢复Servlet容器线程上的请求处理。
+ 在ServletRequest提供对电流DispatcherType，它可以使用处理该初始请求，异步调度之间进行区分，前向，以及其他的调度类型。

DeferredResult 处理工作如下：
+ 控制器返回aDeferredResult并将其保存在一些可以访问它的内存队列或列表中。
+ Spring MVC调用request.startAsync()。
+ 同时，DispatcherServlet和所有配置的过滤器退出请求处理线程，但响应保持打开状态。
+ 应用程序DeferredResult从某个线程设置，Spring MVC将请求分派回Servlet容器。
+ 将DispatcherServlet被再次调用，并且处理与异步生产返回值恢复。

Callable 处理工作如下：
+ 控制器返回Callable。
+ Spring MVC调用request.startAsync()并将其提交Callable到TaskExecutor一个单独的线程中进行处理。
+ 同时，DispatcherServlet和所有过滤器退出Servlet容器线程，但是响应保持打开状态。
+ 最终Callable产生一个结果，Spring MVC将请求分派回Servlet容器以完成处理。
+ 将DispatcherServlet被再次调用，并且处理从所述异步生产返回值恢复Callable。

#### 异常处理
使用时DeferredResult，可以选择呼叫setResult还是setErrorResult。
这两种情况下，Spring MVC都将请求分派回Servlet容器以完成处理。然后将其视为控制器方法返回了给定值，或者好像它产生了给定的异常。
然后，异常将通过常规的异常处理机制（如调用@ExceptionHandler方法）进行处理。

当您使用时Callable，会发生类似的处理逻辑，主要区别是从中返回了结果，Callable或者引发了异常。

#### 拦截器
HandlerInterceptor实例可以是AsyncHandlerInterceptor类型，在初始请求启动异步处理时，接收afterConcurrentHandling回调。

HandlerInterceptor实例也可以是CallableProcessingInterceptor 或 DeferredResultProcessingInterceptor，可以更深入地与异步请求的生命周期集成。

DeferredResult提供onTimeout(Runnable)和onCompletion(Runnable)回调。
Callable可以替换为WebAsyncTask暴露超时和完成回调的其他方法。

#### 与WebFlux对比
Servlet API最初是为通过Filter-Servlet链进行一次传递而构建的。Servlet 3.0中添加了异步请求处理，使应用程序可以退出Filter-Servlet链，但保留响应以进行进一步处理。Spring MVC异步支持围绕该机制构建。当控制器返回a时DeferredResult，退出Filter-Servlet链，并释放Servlet容器线程。稍后，当DeferredResult设置了时，将进行一次ASYNC调度（到相同的URL），在此期间再次映射控制器，但是DeferredResult使用该值（就像控制器返回了它一样）而不是调用它来恢复处理。

相比之下，Spring WebFlux既不是基于Servlet API构建的，也不需要这种异步请求处理功能，因为它在设计上是异步的。异步处理已内置在所有框架协定中，并在请求处理的所有阶段得到内在支持。

从编程模型的角度来看，Spring MVC和Spring WebFlux都支持异步和响应类型作为控制器方法中的返回值。
Spring MVC甚至支持流，包括反应背压。、但是，与WebFlux不同，WebFlux依赖于非阻塞I / O，并且每次写入都不需要额外的线程，因此对响应的单个写入仍然处于阻塞状态（并在单独的线程上执行）。

另一个根本区别在于Spring MVC的不支持在控制器方法参数异步或反应性类型（例如，@RequestBody，@RequestPart，和其它物质），也不会具有用于异步和反应类型作为模型属性的任何显式支持。Spring WebFlux确实支持所有这些。

### !!!HTTP流
您可以将DeferredResult和Callable用于**单个异步返回值**。
如果要产生多个异步值并将那些值写入响应，该怎么办？本节介绍如何执行此操作。

#### Objects
您可以使用ResponseBodyEmitter返回值生成一个对象流，其中每个对象都使用HttpMessageConverter进行序列化并写入响应，如以下示例所示：
```
@GetMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

还可以将ResponseBodyEmitter作为ResponseEntity中的主体，得以自定义响应的状态和标题。

当emitter引发异常时IOException（例如，如果远程客户端消失了），应用程序不负责清理连接，因此不应调用emitter.complete 或emitter.completeWithError。
取而代之的是，Servlet容器自动启动 AsyncListener错误通知，Spring MVC在其中进行completeWithError调用。
依次，此调用ASYNC对应用程序执行最后的调度，在此期间，Spring MVC调用已配置的异常解析器并完成请求。

#### SSE
SseEmitter（ResponseBodyEmitter的子类）提供对服务器发送事件的支持 ，其中从服务器发送的事件根据W3C SSE规范进行格式化。
要从控制器生成SSE流，请返回SseEmitter，如以下示例所示：
```
@GetMapping(path="/events", produces=MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter handle() {
    SseEmitter emitter = new SseEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

虽然SSE是流式传输到浏览器的主要选项，但请注意Internet Explorer不支持服务器发送事件。
考虑将Spring的WebSocket消息与 针对各种浏览器的SockJS后备传输（包括SSE）一起使用。

#### !!!Raw Data
有时，绕过消息转换并直接流式传输到响应很有用（例如，用于文件下载）。
您可以使用```StreamingResponseBody```返回值类型来执行此操作，如以下示例所示：
```
@GetMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```

您可以将StreamingResponseBody用作ResponseEntity的正文，来自定义响应的状态和标题。

### Reactive Types 反应类型
Spring MVC支持在控制器中使用反应式客户端库。这包括WebClient和例如Spring Data反应数据存储库等。
在这种情况下，能够从控制器方法返回反应类型是很方便的。

反应性返回值的处理方式如下：
+ 单值类型与使用DeferredResult相似。示例包括Mono（Reactor）或Single（RxJava）。
+ 流媒体类型（例如 application/x-ndjson 或 text/event-stream）与使用ResponseBodyEmitter or SseEmitter 相似。
+ 其他流媒体类型（例如 application/json）与使用DeferredResult<List<?>>相似。

为了流式传输到响应，支持了反应性背压，但响应的写仍处于阻塞状态，并通过configure TaskExecutor，在单独的线程上运行，以避免阻塞上游源。
默认情况下，SimpleAsyncTaskExecutor用于阻止写操作，但是在负载下不适合使用。如果计划使用响应类型进行流传输，则应使用 MVC配置来配置任务执行程序。

### Disconnects 连接断开
当远程客户端离开时，Servlet API不提供任何通知。

因此，在通过SseEmitter 或 反应性类型流式传输 到响应时，定期发送数据非常重要，因为如果客户端断开连接，写入将失败。
发送可以采取空（仅评论）SSE事件或另一端必须将其解释为心跳和忽略的任何其他数据的形式。

或者，考虑使用具有内置心跳机制的Web消息传递解决方案（例如，基于WebSocket的STOMP或具有SockJS的WebSocket ）。

### Configuration 配置项
必须在Servlet容器级别启用异步请求处理功能。MVC配置为异步请求提供了多个选项。

#### Servlet容器
Filter和Servlet声明具有一个asyncSupported标志，需要将其设置true，启用异步请求处理。
此外，过滤器映射应声明为ASYNC javax.servlet.DispatchType。

在Java配置中，当使用AbstractAnnotationConfigDispatcherServletInitializer 初始化Servlet容器时，这是自动完成的。

在web.xml配置中，您可以添加<async-supported>true</async-supported>到DispatcherServlet和Filter声明以及添加 <dispatcher>ASYNC</dispatcher>到过滤器映射。

#### Spring MVC
MVC配置公开了以下与异步请求处理相关的选项：
+ Java配置：在configureAsyncSupport上使用回调WebMvcConfigurer。
+ XML名称空间：使用下的<async-support>元素<mvc:annotation-driven>。

可以配置以下内容：
+ 异步请求的默认超时值（如果未设置）取决于基础Servlet容器。
+ AsyncTaskExecutor用于在使用响应类型进行流式传输时阻止写操作 以及用于执行Callable从控制器方法返回的实例。我们强烈建议您配置此属性，如果您使用响应类型进行流式处理或具有返回的控制器方法，则Callable默认情况下为a SimpleAsyncTaskExecutor。
+ DeferredResultProcessingInterceptor实施和CallableProcessingInterceptor实施。

请注意，可以在DeferredResult，ResponseBodyEmitter 和 SseEmitter 上设置默认超时值。
对于Callable，可以使用 WebAsyncTask提供超时值。

## !!!CORS 跨域
Spring MVC可以处理CORS（Cross-Origin Resource Sharing 跨源资源共享）。

### 简介
出于安全原因，浏览器禁止AJAX调用当前来源以外的资源。
例如，您可以在一个标签页中拥有银行帐户，在另一个标签页中拥有evil.com。来自evil.com的脚本不应使用您的凭据向您的银行API发出AJAX请求，例如从您的帐户中提取资金！

跨域资源共享（CORS）是 由大多数浏览器实现的W3C规范，可让您指定授权哪种类型的跨域请求。
这优于使用基于IFRAME或JSONP的安全性较低且功能较弱的办法。

### 调用流程
CORS规范定义了三种请求：预检请求、简单请求和实际请求。

Spring MVC的HandlerMapping为CORS实现提供了内置支持。
HandlerMapping 将请求映射到处理程序后，会根据给定CORS配置，并采取进一步的措施。
预检请求直接处理，而简单请求和实际请求则被拦截，验证并设置必需的CORS响应标头。

为了启用跨域请求（即，Origin标头存在并且与请求的主机不同），您需要具有一些显式声明的CORS配置。
如果找不到匹配的CORS配置，则预检请求请求将被拒绝。没有将CORS标头添加到简单和实际CORS请求的响应中，因此，浏览器拒绝了它们。

每个HandlerMapping都可以使用基于URL模式的映射进行单独配置CorsConfiguration。
在大多数情况下，应用程序使用MVC Java配置或XML名称空间声明此类映射，这将单个全局映射传递给所有HandlerMappping实例。

可以将全局CORS配置HandlerMapping与更细粒度的处理程序级CORS配置结合使用。
例如，带注解的控制器可以使用类或方法级的@CrossOrigin注解（其他处理程序可以实现 CorsConfigurationSource）。

组合全局和本地配置的规则通常是相加的，例如，所有全局和所有本地来源。
对于那些只能接受单个值的属性，例如allowCredentials和maxAge，局部变量将覆盖全局值。

### @CrossOrigin
@CrossOrigin 注解能够对带注解的控制器方法跨域请求，如下面的示例所示：
```
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

默认情况下，@CrossOrigin允许：
+ 所有origins。
+ 所有headers。
+ 控制器方法映射到的所有HTTP方法。

allowCredentials默认情况下不会启用，因为这会建立一个信任级别，以公开敏感的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。
启用后，allowOrigins必须将其设置为一个或多个特定域（而不是特殊值"*"），或者可以使用allowOriginPatterns属性来匹配动态的一组原点。

maxAge 设置为30分钟。

@CrossOrigin 在类级别也受支持，并且被所有方法继承，如以下示例所示：
```
@CrossOrigin(origins = "https://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

可以@CrossOrigin在类级别和方法级别上使用，如以下示例所示：
```
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com")
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

### 全局配置
除了细粒度的控制器方法级别配置之外，您可能还想定义一些全局CORS配置。
您可以在任何HandlerMapping中分别设置基于URL的CorsConfiguration映射。
但是，大多数应用程序都使用Java配置或XML名称空间来做到这一点。

默认情况下，全局配置启用以下功能：
+ 所有origins。
+ 所有headers。
+ GET，HEAD和POST方法。

allowCredentials默认情况下不会启用，因为这会建立一个信任级别，以公开敏感的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。
启用后，allowOrigins必须将其设置为一个或多个特定域（而不是特殊值"*"），或者allowOriginPatterns可以使用该属性来匹配动态的一组原点。

maxAge 设置为30分钟。

#### Java配置
要在Java配置中启用CORS，可以使用CorsRegistry回调，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

#### XML配置
要在XML名称空间中启用CORS，可以使用<mvc:cors>元素，如以下示例所示：
```
<mvc:cors>
    <mvc:mapping path="/api/**"
        allowed-origins="https://domain1.com, https://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="true"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="https://domain1.com" />
</mvc:cors>
```

### CORS过滤器
可以通过内置过滤器CorsFilter来应用CORS支持。

要配置过滤器，请传递一个CorsConfigurationSource给其构造函数，如以下示例所示：
```
CorsConfiguration config = new CorsConfiguration();

// Possibly...
// config.applyPermitDefaultValues()

config.setAllowCredentials(true);
config.addAllowedOrigin("https://domain1.com");
config.addAllowedHeader("*");
config.addAllowedMethod("*");

UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
source.registerCorsConfiguration("/**", config);

CorsFilter filter = new CorsFilter(source);
```

## Web Security 网络安全
在Spring Security的项目提供了保护Web应用程序免受恶意攻击的支持。
请参阅[Spring Security参考文档](https://spring.io/projects/spring-security)。

## HTTP Caching HTTP缓存
HTTP缓存可以显着提高Web应用程序的性能。

HTTP缓存围绕Cache-Control响应标头以及随后的条件请求标头（例如Last-Modified和ETag）展开。
Cache-Control为私有（例如浏览器）和公共（例如代理）缓存提供有关如何缓存和重用响应的建议。
一个ETag头用于使没有身体可能导致一个304（NOT_MODIFIED）一个条件请求，如果内容没有改变。
ETag可以看作是Last-Modified标题的更复杂的后继者。

### CacheControl
CacheControl提供对配置与Cache-Control标头相关的设置的支持，并在许多地方作为参数被接受：
+ WebContentInterceptor
+ WebContentGenerator
+ Controllers
+ Static Resources

尽管RFC 7234描述了Cache-Control响应标头的所有可能的指令，但该CacheControl类型采用面向用例的方法，重点关注常见方案：
```
// 缓存1个小时 - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// 防止缓存 - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// 在公共和私有缓存中缓存十天,公共缓存不应转换响应
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```

WebContentGenerator还接受如下更简单的cachePeriod属性（以秒为单位定义）：
+ -1：不产生Cache-Control响应头。
+  0：可以防止通过使用缓存'Cache-Control: no-store'指令。
+ >0：通过使用 'Cache-Control: max-age=n'指令，缓存指定响应n秒。

### Controllers
控制器可以添加对HTTP缓存的显式支持。

建议这样做，因为需要先计算资源的lastModified 或 ETag值，然后才能将其与条件请求标头进行比较。

控制器可以将ETag标头和Cache-Control 设置添加到中ResponseEntity，如以下示例所示：
```
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
}
```

如果与条件请求标头的比较表明内容未更改，则前面的示例发送带有空正文的304（NOT_MODIFIED）响应。
否则， ETag和Cache-Control标头将添加到响应中。

还可以在控制器中针对条件请求标头进行检查，如以下示例所示：
```
@RequestMapping
public String myHandleMethod(WebRequest request, Model model) {

	// 基于实际业务的计算
    long eTag = ... 

    if (request.checkNotModified(eTag)) {
		// 没有变化，响应设置为304（NOT_MODIFIED）-无需进一步处理
        return null; 
    }
	
	// 发生变化，继续进行请求处理
    model.addAttribute(...); 
    return "myViewName";
}
```

有三种变体，用于根据eTag值和lastModified /或值检查条件请求。
对于条件GET和HEAD要求，可以设置为304（NOT_MODIFIED）的响应。
对于有条件的POST，PUT和DELETE，可以改为将响应设置为412（PRECONDITION_FAILED），以防止并发修改。

### Static Resources
您应该为静态资源提供Cache-Control和条件响应标头，以实现最佳性能。

### ETag Filter
您可以使用ShallowEtagHeaderFilter来添加eTag根据响应内容计算得出的“浅”值，从而节省带宽，但不节省CPU时间。

## View Technologies 视图技术
### Thymeleaf
Thymeleaf是一种现代的服务器端Java模板引擎，它强调可以在浏览器中预览的自然HTML模板，而无需使用正在运行的服务器，这对于独立处理UI模板（例如，由设计人员）非常有用。

如果要替换JSP，Thymeleaf提供了最广泛的功能集之一，以使这种过渡更加容易。

Thymeleaf是积极开发和维护的。

详情参考 [Thymeleaf项目主页](https://www.thymeleaf.org/)。

### FreeMarker
[Apache FreeMarker](https://freemarker.apache.org/)是一个模板引擎，用于生成从HTML到电子邮件等的任何类型的文本输出。

Spring框架具有内置的集成，可以将Spring MVC与FreeMarker模板一起使用。

#### View配置
以下示例显示了如何将FreeMarker配置为一种视图技术：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();
    }

    // Configure FreeMarker...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/WEB-INF/freemarker");
        return configurer;
    }
}
```

以下示例显示了如何在XML中进行配置：
```
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:freemarker/>
</mvc:view-resolvers>

<!-- Configure FreeMarker... -->
<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/WEB-INF/freemarker"/>
</mvc:freemarker-configurer>
```

另外，您也可以声明FreeMarkerConfigurerBean以完全控制所有属性，如以下示例所示：
```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>
```

模板需要存储在FreeMarkerConfigurer 上例所示的目录中。
依照上述配置，如果您的控制器返回的视图名称为welcome，则解析器将查找 /WEB-INF/freemarker/welcome.ftl模板。

#### FreeMarker配置
您可以通过Configuration在bean上设置适当的bean属性，将FreeMarker的“Settings”和“SharedVariables”直接传递给FreeMarker 对象（由Spring管理）FreeMarkerConfigurer。

freemarkerSettings属性需要一个java.util.Properties对象，而该freemarkerVariables属性需要一个 java.util.Map。

以下示例显示了如何使用FreeMarkerConfigurer：
```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
    <property name="freemarkerVariables">
        <map>
            <entry key="xml_escape" value-ref="fmXmlEscape"/>
        </map>
    </property>
</bean>

<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```

#### 表单处理
略

### Groovy Markup
略

### Script Views
### JSP and JSTL
### Tiles
略

### RSS and Atom
略

### PDF and Excel
Spring提供了返回HTML以外的输出的方法，包括PDF和Excel电子表格。

#### 文档视图简介
HTML页面并非始终是用户查看模型输出的最佳方法，而Spring使从模型数据动态生成PDF文档或Excel电子表格变得简单。
该文档是视图，并从服务器以正确的内容类型进行流传输，以（希望）使客户端PC能够运行其电子表格或PDF查看器应用程序作为响应。

为了使用Excel视图，您需要将Apache POI库添加到您的类路径中。

为了生成PDF，您需要添加（最好是）OpenPDF库。

#### PDF视图
单词列表的简单PDF视图可以扩展 org.springframework.web.servlet.view.document.AbstractPdfView和实现该 buildPdfDocument()方法，如以下示例所示：
```
public class PdfWordList extends AbstractPdfView {

    protected void buildPdfDocument(Map<String, Object> model, Document doc, PdfWriter writer,
            HttpServletRequest request, HttpServletResponse response) throws Exception {

        List<String> words = (List<String>) model.get("wordList");
        for (String word : words) {
            doc.add(new Paragraph(word));
        }
    }
}
```

控制器可以从外部视图定义（按名称引用）或作为View处理程序方法的实例返回这种视图。

#### Excel视图
从Spring Framework 4.2开始， org.springframework.web.servlet.view.document.AbstractXlsView它作为Excel视图的基类提供。它基于Apache POI，具有取代过时类的专用子类（AbstractXlsxView 和AbstractXlsxStreamingView）AbstractExcelView。

编程模型类似于AbstractPdfView，buildExcelDocument() 作为中央模板方法，控制器能够从外部定义（按名称）或View从处理程序方法作为实例返回这种视图。

### Jackson
Spring提供了对Jackson JSON库的支持。

#### 基于Jackson的JSON MVC视图
MappingJackson2JsonView 使用Jackson库的ObjectMapper将响应内容渲染为JSON。
默认情况下，模型映射的所有内容（特定于框架的类除外）均编码为JSON。

对于需要过滤地图内容的情况，可以使用属性指定一组特定的模型属性进行编码modelKeys。
还可以使用该extractValueFromSingleKeyModel 属性来直接提取和序列化单键模型中的值，而不是将其作为模型属性的映射。

可以根据需要使用Jackson提供的注解来自定义JSON映射。
当需要进一步控制时，可以为需要为特定类型提供自定义JSON序列化器和反序列化器的情况，ObjectMapper 通过ObjectMapper属性注入自定义。

#### 基于Jackson的XML视图
MappingJackson2XmlView 使用 Jackson的XML扩展XmlMapper 将响应内容渲染为XML。

如果模型包含多个条目，则应使用modelKeybean属性显式设置要序列化的对象。如果模型包含单个条目，则会自动序列化。

可以根据需要使用JAXB或Jackson提供的注解自定义XML映射。
当需要进一步控制时，可以XmlMapper 通过ObjectMapper属性注入自定义，对于自定义XML的情况，您需要为特定类型提供序列化器和反序列化器。

### XML Marshalling
略

### XSLT Views
略

## !!!MVC Config MVC配置
MVC的Java配置和MVC的XML命名空间，提供了适用于大多数应用程序的默认配置以及用于自定义它的配置API。

### Enable MVC Config 启用MVC配置
在Java配置中，可以使用```@EnableWebMvc```注解启用MVC配置，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

在XML配置中，可以使用```<mvc:annotation-driven>```元素来启用MVC配置，如以下示例所示：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

### MVC Config API MVC配置API
在Java配置中，可以实现该WebMvcConfigurer接口，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```

在XML中，您可以查看```<mvc：Annotation-Driven/>```的属性和子元素。相关XML协议如下所示：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

### Type Conversion 类型转换
默认情况下，将安装各种数字和日期类型的格式化程序，并支持通过@NumberFormat和@DateTimeFormat在字段上进行自定义。

要在Java配置中注册自定义格式器和转换器，请使用以下命令：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

要在XML配置中执行相同的操作，请使用以下命令：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```

默认情况下，Spring MVC在解析和格式化日期值时会考虑请求区域设置。
这适用于使用“输入”表单字段将日期表示为字符串的表单。
但是，对于“日期”和“时间”表单字段，浏览器使用HTML规范中定义的固定格式。
在这种情况下，日期和时间格式可以按以下方式自定义：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}
```

### Validation 参数验证
默认情况下，如果Bean验证存在于类路径中（例如，Hibernate Validator），则将LocalValidatorFactoryBean其注册为全局验证器，以@Valid与 Validated控制器方法参数一起使用。

在Java配置中，您可以自定义全局Validator实例，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        // ...
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven validator="globalValidator"/>

</beans>
```

Validator还可以在本地注册实现，如以下示例所示：
```
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }
}

```

### Interceptors 拦截器
在Java配置中，您可以注册拦截器以应用于传入的请求，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

### Content Types
您可以配置Spring MVC如何根据请求确定请求的媒体类型（例如，Accept标头，URL路径扩展，查询参数等）。

URL路径扩展首先检查-有json，xml，rss，并atom 注册为已知扩展名（视路径依赖），然后检查Accept的报头。

考虑将这些默认值更改为Accept仅标头，并且，如果必须使用基于URL的内容类型解析，请考虑对路径扩展使用查询参数策略。

在Java配置中，您可以自定义请求的内容类型解析，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```

### Message Converters
您可以在Java配置中自定义```HttpMessageConverter```。
通过重写方法```configureMessageConverters()```，替换Spring MVC创建的默认转换器。
通过重写方法```extendMessageConverters()```，向默认转换器添加额外的转换器。

以下示例通过自定义的```ObjectMapper```，添加了一个的XML和JSON转换器来代替默认的转换器：
```
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
				
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```

在前面的示例中，```Jackson2ObjectMapperBuilder```用于为```MappingJackson2HttpMessageConverter```和```MappingJackson2XmlHttpMessageConverter```创建通用配置。配置包括：启用缩进、定制日期格式以及注册Jackson模块参数名称，

此构建器自定义Jackson的默认属性如下所示：
+ DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES 被禁用。
+ MapperFeature.DEFAULT_VIEW_INCLUSION 被禁用。

如果在类路径中检测到以下知名模块，它将自动注册以下知名模块：
+ jackson-datatype-joda：支持Joda-Time类型。
+ jackson-datatype-jsr310：支持Java 8日期和时间API类型。
+ jackson-datatype-jdk8：支持其他Java 8类型，例如Optional。
+ jackson-module-kotlin：支持Kotlin类和数据类。

其他有趣的Jackson模块也可用：
+ jackson-datatype-money：支持javax.money类型（非官方模块）。
+ jackson-datatype-hibernate：支持特定于Hibernate的类型和属性（包括延迟加载方面）。

### View Controllers
这是定义一个ParameterizableViewController的快捷方式，该快捷方式在调用时立即转发到视图。
当视图生成响应之前没有Java控制器逻辑要运行时，可以在静态情况下使用它。

以下Java配置示例将请求转发/到名为的视图home：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

以下示例通过使用<mvc:view-controller>元素，实现了与上一示例相同的操作，但使用XML ：
```
<mvc:view-controller path="/" view-name="home"/>
```

如果将@RequestMapping方法映射到任何HTTP方法的URL，则视图控制器不能用于处理相同的URL。
这是因为通过URL与带注释的控制器的匹配被视为端点所有权的足够有力的指示，因此可以将405（METHOD_NOT_ALLOWED），415（UNSUPPORTED_MEDIA_TYPE）或类似的响应发送给客户端，以帮助进行调试。
因此，建议避免在带注释的控制器和视图控制器之间拆分URL处理。

### View Resolvers
MVC配置简化了视图解析器的注册。

以下Java配置示例通过使用JSP和Jackson作为ViewJSON呈现的默认配置来配置内容协商视图解析：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```

MVC命名空间提供了专用元素。以下示例适用于FreeMarker：
```
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

在Java配置中，您可以添加相应的Configurerbean，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/freemarker");
        return configurer;
    }
}
```

### Static Resources
此选项提供了一种从基于位置的列表中提供静态资源的便捷的方法。

在下一个示例中，给定一个以/resources开头的请求，相对路径用为Web应用程序根目录下/public或类路径下/static。
这些资源的使用期限为一年，以确保最大程度地利用浏览器缓存并减少浏览器发出的HTTP请求。
从Resource#lastModified中推导出Last-Modified信息，以便HTTP条件请求支持"Last-Modified"标头。

以下清单显示了如何使用Java配置进行操作：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCacheControl(CacheControl.maxAge(Duration.ofDays(365)));
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:resources mapping="/resources/**"
    location="/public, classpath:/static/"
    cache-period="31556926" />
```

资源处理程序还支持一系列 ResourceResolver实现和 ResourceTransformer实现，您可以使用它们来创建用于处理优化资源的工具链。

您可以VersionResourceResolver根据从内容，固定应用程序版本或其他版本计算出的MD5哈希值，使用for版本资源URL。
 ContentVersionStrategy（MD5哈希）是一个不错的选择-有一些明显的例外，如与模块加载程序使用的JavaScript资源。

以下示例显示了如何VersionResourceResolver在Java配置中使用：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }
}

```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:resources mapping="/resources/**" location="/public/">
    <mvc:resource-chain resource-cache="true">
        <mvc:resolvers>
            <mvc:version-resolver>
                <mvc:content-version-strategy patterns="/**"/>
            </mvc:version-resolver>
        </mvc:resolvers>
    </mvc:resource-chain>
</mvc:resources>
```

然后，您可以用ResourceUrlProvider来重写URL并应用完整的解析器和转换器链，例如，插入版本。
MVC配置提供了一个ResourceUrlProvider bean，以便可以将其注入其他对象。
您也可以使用 Thymeleaf，JSP，FreeMarker以及其他依赖于URL标签的ResourceUrlEncodingFilter使重写 HttpServletResponse#encodeURL透明。

请注意，在同时使用和EncodedResourceResolver（例如，用于提供压缩或brotli编码的资源）和时VersionResourceResolver，必须按此顺序注册它们。
这样可以确保始终基于未编码文件可靠地计算基于内容的版本。

WebJarsResourceResolver当org.webjars:webjars-locator-core类路径中存在库时，也会通过 自动注册 WebJars。
解析程序可以重写URL以包括jar的版本，还可以与没有版本的传入URL进行匹配，例如从/jquery/jquery.min.js到 /jquery/1.2.0/jquery.min.js。

### Default Servlet
Spring MVC允许映射DispatcherServlet到/（从而覆盖了容器默认Servlet的映射），同时仍允许容器默认Servlet处理静态资源请求。
它配置了一个DefaultServletHttpRequestHandlerURL映射为/**，相对于其他URL映射具有最低的优先级。

该处理程序将所有请求转发到默认Servlet。因此，它必须按所有其他URL的顺序保留在最后HandlerMappings。
如果使用<mvc:annotation-driven>，就是这种情况。
如果设置定制的HandlerMapping，一定要设置其order属性值比DefaultServletHttpRequestHandler的值（Integer.MAX_VALUE）低。

下面的示例演示如何使用默认设置启用功能：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:default-servlet-handler/>
```

覆盖Servlet映射 / 需要的注意是，RequestDispatcher必须通过名称而不是通过路径来检索默认Servlet的。

在 DefaultServletHttpRequestHandler尝试自动检测默认的Servlet在启动时的容器，使用大多数主要的Servlet容器（包括软件Tomcat, Jetty, GlassFish, JBoss, Resin, WebLogic, and WebSphere）已知名称的列表。
如果已使用其他名称自定义配置了默认Servlet，或者在默认Servlet名称未知的情况下使用了不同的Servlet容器，则必须显式提供默认Servlet的名称，如以下示例所示：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```

### Path Matching
您可以自定义与路径匹配和URL处理有关的选项。

以下示例显示了如何在Java配置中自定义路径匹配：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setPatternParser(new PathPatternParser())
            .addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
    }

    private PathPatternParser patternParser() {
        // ...
    }
}
```

以下示例显示了如何在XML中实现相同的配置：
```
<mvc:annotation-driven>
    <mvc:path-matching
        trailing-slash="false"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```

### Advanced Java Config
@EnableWebMvc引入了DelegatingWebMvcConfiguration，其中：
+ 为Spring MVC应用程序提供默认的Spring配置
+ 检测并委托给WebMvcConfigurer实现以自定义该配置。

对于高级模式，您可以直接删除@EnableWebMvc，并且实现展DelegatingWebMvcConfiguration而不是实现WebMvcConfigurer，如以下示例所示：
```
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...
}
```

您可以将现有方法保留在WebConfig中，但是现在您还可以覆盖基类中的bean声明，并且在类路径上仍然可以具有许多其他WebMvcConfigurer实现。

### Advanced XML Config
MVC命名空间没有高级模式。
如果您需要在bean上自定义一个不能更改的属性，则可以使用BeanPostProcessor与Spring的ApplicationContext生命周期挂钩，如以下示例所示：
```
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        // ...
    }
}
```

请注意，您需要以MyPostProcessorXML形式显式声明或通过<component-scan/>声明使其被检测为bean 。

## HTTP/2
需要Servlet 4容器支持HTTP / 2，并且Spring Framework 5与Servlet API 4兼容。
从编程模型的角度来看，应用程序不需要做任何特定的事情。
但是，有一些与服务器配置有关的注意事项。

Servlet API确实公开了一种与HTTP / 2相关的构造。
您可以使用 javax.servlet.http.PushBuilder主动将资源推送到客户端，并且它作为方法的方法参数而受支持@RequestMapping。

# REST客户端
客户端对REST端点的访问。

## RestTemplate
RestTemplate是执行HTTP请求的同步客户端。
它是原始的Spring REST客户端，在基础HTTP客户端库上公开了简单的模板方法API。

从5.0版本开始，RestTemplate它处于维护模式，以后只有很少的更改和错误请求被接受。
请考虑使用提供更现代API并支持同步，异步和流传输方案的 WebClient。

## WebClient
WebClient是执行HTTP请求的非阻塞，反应式客户端。
它是在5.0中引入的，提供了的现代替代方案RestTemplate，并有效支持同步和异步以及流方案。

与相比RestTemplate，WebClient支持以下内容：
+ 非阻塞I / O。
+ 反应性产生背压。
+ 高并发，硬件资源更少。
+ 利用Java 8 lambda的功能风格，流畅的API。
+ 同步和异步交互。
+ 从服务器流向上或向下流。

# 测试
本节总结了 Spring MVC应用程序中spring-test可用的选项：
+ Servlet API Mocks：用于单元测试控制器，过滤器和其他Web组件的Servlet API合约的模拟实现。
+ TestContext Framework：支持在JUnit和TestNG测试中加载Spring配置，包括跨测试方法高效地缓存已加载的配置，并支持通过MockServletContext加载WebApplicationContexta。
+ Spring MVC Test：一种框架，也称为MockMvc，用于通过DispatcherServlet（即支持注释）测试带注释的控制器，该框架具有Spring MVC基础结构，但没有HTTP服务器。
+ Client-side REST：spring-test提供一个MockRestServiceServer，您可以用作模拟服务器来测试内部使用的客户端代码RestTemplate。
+ WebTestClient：专为测试WebFlux应用程序而设计，但也可以用于通过HTTP连接到任何服务器的端到端集成测试。它是一个无阻塞的反应式客户端，非常适合测试异步和流传输方案。

# WebSockets
此部分涵盖对Servlet堆栈的支持，包括原始WebSocket交互的WebSocket消息传递，通过SockJS进行WebSocket仿真以及通过STOMP作为WebSocket的子协议进行发布-订阅消息传递。

## WebSocket介绍
WebSocket协议 (RFC 6455) 提供了一种标准化的方法，可以通过单个TCP连接在客户端和服务器之间建立全双工的双向通信通道。
它是与HTTP不同的TCP协议，但旨在通过端口80和443在HTTP上工作，并允许重复使用现有的防火墙规则。

WebSocket交互始于一个HTTP请求，该请求使用HTTP Upgrade 标头进行升级，或者在这种情况下切换到WebSocket协议。
以下示例显示了这种交互：
```
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```
具有WebSocket支持的服务器代替通常的200状态代码，返回类似于以下内容的输出：
```
HTTP/1.1 101 Switching Protocols 
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```
成功握手后，HTTP升级请求的基础TCP套接字将保持打开状态，客户端和服务器均可继续发送和接收消息。

请注意，如果WebSocket服务器在Web服务器（例如nginx）后面运行，则可能需要对其进行配置，以将WebSocket升级请求传递到WebSocket服务器。
同样，如果应用程序在云环境中运行，请检查与WebSocket支持相关的云提供商的说明。

### HTTP与WebSocket
尽管WebSocket被设计为与HTTP兼容并以HTTP请求开头，但重要的是要了解这两个协议导致了截然不同的体系结构和应用程序编程模型。

在HTTP和REST中，应用程序被建模为许多URL。
为了与应用程序交互，客户端访问那些URL，即请求-响应样式。服务器根据HTTP URL，方法和标头将请求路由到适当的处理程序。

相比之下，在WebSockets中，初始连接通常只有一个URL。
随后，所有应用程序消息都在同一TCP连接上流动。这指向了一个完全不同的异步，事件驱动的消息传递体系结构。

WebSocket也是一种低级传输协议，与HTTP不同，它不对消息的内容规定任何语义。
这意味着除非客户端和服务器就消息语义达成一致，否则就无法路由或处理消息。

WebSocket客户端和服务器可以通过Sec-WebSocket-ProtocolHTTP握手请求上的标头协商使用更高级别的消息传递协议（例如STOMP）。
在这种情况下，他们需要提出自己的约定。

### 何时使用WebSocket
WebSockets可以使网页具有动态性和交互性。
但是，在许多情况下，结合使用Ajax和HTTP流或长时间轮询可以提供一种简单有效的解决方案。

例如，新闻，邮件和社交订阅源需要动态更新，但是每几分钟进行一次更新可能是完全可以的。
另一方面，协作，游戏和金融应用程序需要更接近实时。

仅延迟并不是决定因素。
如果消息量相对较少（例如，监视网络故障），则HTTP流或轮询可以提供有效的解决方案。
低延迟，高频率和高音量的结合才是使用WebSocket的最佳案例。

还要记住，在Internet上，控件之外的限制性代理可能会阻止WebSocket交互，这可能是因为未将它们配置为传递 Upgrade标头，或者是因为它们关闭了长期处于空闲状态的连接。
这意味着与面向公众的应用程序相比，将WebSocket用于防火墙内部的应用程序是一个更直接的决定。

## WebSocket API
Spring框架提供了一个WebSocket API，可用于编写处理WebSocket消息的客户端和服务器端应用程序。

### WebSocketHandler
创建WebSocket服务器就像WebSocketHandler实现一样简单，或者扩展TextWebSocketHandler或BinaryWebSocketHandler。
以下示例使用TextWebSocketHandler：
```
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.TextMessage;

public class MyHandler extends TextWebSocketHandler {

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        // ...
    }

}
```

有专用WebSocket的 Java配置和XML名称空间 支持，用于将前面的WebSocket处理程序映射到特定的URL，如以下示例所示：
```
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

以下示例显示了与先前示例等效的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

前面的示例用于Spring MVC应用程序，并且应包含在的DispatcherServlet配置中。
但是Spring的WebSocket并不依赖于Spring MVC。
在WebSocketHttpRequestHandler的帮助下，可以简单地将WebSocketHandler集成到其他HTTP服务环境中 。

当WebSocketHandler直接使用API或间接使用API（例如通过 STOMP消息传递）时，由于基础标准WebSocket会话（JSR-356）不允许并发发送，因此应用程序必须同步消息的发送。一种选择是使用ConcurrentWebSocketSessionDecorator将WebSocketSession封装起来 。

### WebSocket Handshake
定制初始WebSocket握手请求的最简单方法是通过HandshakeInterceptor，它公开了“before”和“after”的方法。
可以使用此类拦截器来阻止握手或使用任何WebSocketSession可用属性。

以下示例使用内置的拦截器将HTTP会话属性传递到WebSocket会话：
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler")
            .addInterceptors(new HttpSessionHandshakeInterceptor());
    }

}
```

以下示例显示了与先前示例等效的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:handshake-interceptors>
            <bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor"/>
        </websocket:handshake-interceptors>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

一个更高级的选项是继承DefaultHandshakeHandler，它执行WebSocket握手的步骤包括验证客户端来源，协商子协议以及其他详细信息。

如果应用程序需要配置自定义项RequestUpgradeStrategy以适应尚不支持的WebSocket服务器引擎和版本，则它可能还需要使用此选项。

### 部署方式
易于将Spring WebSocket API集成到Spring MVC应用程序中，在该应用程序中DispatcherServlet可以同时服务HTTP WebSocket握手和其他HTTP请求。通过调用也很容易集成到其他HTTP处理方案中WebSocketHttpRequestHandler。这是方便且易于理解的。但是，对于JSR-356运行时，需要特别注意。

Java WebSocket API（JSR-356）提供了两种部署机制。首先涉及启动时的Servlet容器类路径扫描（Servlet 3功能）。另一个是在Servlet容器初始化时使用的注册API。这两种机制都无法将单个“前端控制器”用于所有HTTP处理（包括WebSocket握手和所有其他HTTP请求）（例如Spring MVC）DispatcherServlet。

这是对JSR-356的重大限制，RequestUpgradeStrategy即使在JSR-356运行时中运行，Spring的WebSocket支持也可以通过服务器特定的实现来解决。Tomcat，Jetty，GlassFish，WebLogic，WebSphere和Undertow（以及WildFly）目前存在此类策略。

### 服务配置
每个基础的WebSocket引擎都公开配置属性，这些属性控制运行时特征，例如消息缓冲区大小的大小，空闲超时等。

对于Tomcat，WildFly和GlassFish，可以将一个ServletServerContainerFactoryBean添加到WebSocket Java配置中，如以下示例所示：
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }

}
```

以下示例显示了与先前示例等效的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <bean class="org.springframework...ServletServerContainerFactoryBean">
        <property name="maxTextMessageBufferSize" value="8192"/>
        <property name="maxBinaryMessageBufferSize" value="8192"/>
    </bean>

</beans>
```

对于Jetty，您需要提供一个预先配置的Jetty，WebSocketServerFactory然后DefaultHandshakeHandler通过WebSocket Java配置将其插入Spring 。以下示例显示了如何执行此操作：
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(),
            "/echo").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }

}
```

以下示例显示了与先前示例等效的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/echo" handler="echoHandler"/>
        <websocket:handshake-handler ref="handshakeHandler"/>
    </websocket:handlers>

    <bean id="handshakeHandler" class="org.springframework...DefaultHandshakeHandler">
        <constructor-arg ref="upgradeStrategy"/>
    </bean>

    <bean id="upgradeStrategy" class="org.springframework...JettyRequestUpgradeStrategy">
        <constructor-arg ref="serverFactory"/>
    </bean>

    <bean id="serverFactory" class="org.eclipse.jetty...WebSocketServerFactory">
        <constructor-arg>
            <bean class="org.eclipse.jetty...WebSocketPolicy">
                <constructor-arg value="SERVER"/>
                <property name="inputBufferSize" value="8092"/>
                <property name="idleTimeout" value="600000"/>
            </bean>
        </constructor-arg>
    </bean>

</beans>
```

### 跨域支持
从Spring Framework 4.1.5开始，WebSocket和SockJS的默认行为是仅接受同源请求。
也可以允许所有或指定的来源列表。
此检查主要用于浏览器客户端。

三种可能的行为是：
+ 仅允许同源请求（默认）：在此模式下，启用SockJS时，Iframe HTTP响应标头X-Frame-Options设置为SAMEORIGIN，并且JSONP传输被禁用，因为它不允许检查请求的来源。因此，启用此模式时，不支持IE6和IE7。
+ 允许指定来源列表：每个允许的来源必须以http:// 或开头https://。在此模式下，启用SockJS时，将禁用IFrame传输。因此，启用此模式时，不支持IE6到IE9。
+ 允许所有原点：要启用此模式，应提供*作为允许的原点值。在这种模式下，所有传输都可用。

可以配置WebSocket和SockJS允许的来源，如以下示例所示：
```
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("https://mydomain.com");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

以下示例显示了与先前示例等效的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers allowed-origins="https://mydomain.com">
        <websocket:mapping path="/myHandler" handler="myHandler" />
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

## SockJS Fallback
在公共Internet上，控件外部的限制性代理可能会阻止WebSocket交互，这可能是因为未将它们配置为传递Upgrade标头，或者是因为它们关闭了长期处于空闲状态的连接。

解决此问题的方法是WebSocket仿真，即先尝试使用WebSocket，然后再尝试使用基于HTTP的技术来模拟WebSocket交互并公开相同的应用程序级API。

在Servlet堆栈上，Spring框架为SockJS协议提供服务器（和客户端）支持。

### 总览
SockJS的目标是让应用程序使用WebSocket API，但在运行时必要时使用非WebSocket替代方案，而无需更改应用程序代码。

SockJS包括：
+ 所述SockJS协议 以可执行的形式定义的 解说的测试。
+ 该SockJS JavaScript客户端 -在浏览器中使用客户端库。
+ SockJS服务器实现，包括Spring Frameworkspring-websocket模块中的一个。
+ spring-websocket模块中的SockJS Java客户端（从4.1版开始）。

SockJS设计用于浏览器。它使用多种技术来支持各种浏览器版本。
传输分为三大类：WebSocket，HTTP流和HTTP长轮询。

SockJS客户端从发送开始以GET /info从服务器获取基本信息。
在那之后，它必须决定使用哪种交通工具。如果可能，请使用WebSocket。如果没有，在大多数浏览器中，至少有一个HTTP流选项。如果不是，则使用HTTP（长）轮询。

所有传输请求都具有以下URL结构：
```
https：// host：port / myApp / myEndpoint / {server-id} / {session-id} / {transport}
```
+ {server-id} 在路由集群中的请求时很有用，但否则不使用。,
+ {session-id} 关联属于SockJS会话的HTTP请求。
+ {transport}指示传输类型（例如，websocket，xhr-streaming，和其它物质）。

WebSocket传输仅需要单个HTTP请求即可进行WebSocket握手。此后所有消息在该套接字上交换。
HTTP传输需要更多请求。
例如，Ajax/XHR流依赖于对服务器到客户端消息的一个长时间运行的请求，以及对客户端到服务器消息的其他HTTP POST请求。
长轮询与此类似，不同之处在于长轮询在每次服务器到客户端发送后结束当前请求。

SockJS添加了最少的消息框架。例如：
+ 字母 o(open)：服务器初始化。
+ JSON数组 a["message1","message2"]：发送消息。
+ 字母 h(heartbeat)：如果在25秒（默认）内没有消息流。
+ 字母 c(close)：以关闭会话。

### 启用S​​ockJS
通过Java配置启用SockJS，如以下示例所示：
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").withSockJS();
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

以下示例显示了与先前示例等效的XML配置：
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:sockjs/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

### IE 8 and 9

### Heartbeats

### Client Disconnects

### SockJS and CORS

### SockJsClient

## STOMP
WebSocket协议定义了两种消息类型（文本消息和二进制消息），但是其内容未定义。

该协议定义了一种机制，供客户端和服务器协商用于在WebSocket之上使用的子协议（即高级消息协议），以定义每种协议可以发送的消息类型，格式，内容。每个消息，依此类推。

子协议的使用是可选的，但是无论哪种方式，客户端和服务器都需要就定义消息内容的某种协议达成共识。

### Overview
### Benefits
### Enable STOMP
### WebSocket Server
### Flow of Messages
### Annotated Controllers
### Sending Messages
### Simple Broker
### External Broker
### Connecting to a Broker
### Dots as Separators
### Authentication
### Token Authentication
### User Destinations
### Order of Messages
### Events
### Interception
### STOMP Client
### WebSocket Scope
### Performance
### Monitoring
### Testing

# 其他Web框架
本章详细介绍了Spring与第三方Web框架的集成。

Spring框架的核心价值主张之一就是**支持多种选择**。

从一般意义上讲，Spring不会强迫您使用或购买任何特定的体系结构，技术或方法（尽管它肯定比其他建议更重要）。
可以自由选择与开发人员及其开发团队最相关的架构，技术或方法，这在Web领域最明显。
在Web领域，Spring提供了自己的Web框架（Spring MVC和Spring WebFlux），同时支持与许多流行的第三方Web框架集成。

## 通用配置
在深入研究每个受支持的Web框架的集成细节之前，让我们首先看一下并非特定于任何Web框架的通用Spring配置。

Spring的轻量级应用程序模型拥护的一个概念是分层体系结构的概念。
在“经典”分层体系结构中，Web层只是众多层中的一层。
它充当服务器端应用程序的入口点之一，并且委派服务层中定义的服务对象（外观），以满足特定于业务（与表示技术无关）的用例。
在Spring中，这些服务对象，任何其他特定于业务的对象，数据访问对象和其他对象都存在于不同的“业务上下文”中，其中不包含Web或表示层对象（表示对象，例如Spring MVC控制器，通常是在不同的“展示环境”中进行配置）。
 
具体要做的是，在Web应用程序的web.xml文件中声明一个 ContextLoaderListener，并在<context-param />添加 contextConfigLocation部分，该部分定义了如何加载Spring XML配置文件。

首先声明以下<listener/>配置：
```
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
然后声明以下<context-param/>配置：
```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext*.xml</param-value>
</context-param>
```
如果未指定 contextConfigLocation 参数，ContextLoaderListener 将查找并加载名为/WEB-INF/applicationContext.xmlload的文件。
Context文件加载完成后，Spring将创建一个WebApplicationContext对象，并将其存储在Web应用程序的ServletContext中。

所有Java Web框架都建立在Servlet API的基础上，因此可以使用以下代码段来访问 ContextLoaderListener 创建的 ApplicationContext。
以下示例显示了如何获取WebApplicationContext：
```
WebApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext);
```

一旦有了对WebApplicationContext的引用，就可以按其名称或类型检索bean。
大多数开发人员都按名称检索bean，然后将其转换为实现的接口之一。

幸运的是，本节中的大多数框架都具有更简单的查找bean的方法。
它们不仅使从Spring容器中获取bean变得容易，而且还使您可以在其控制器上使用依赖项注入。

## JSF
JavaServer Faces（JSF）是基于JCP标准组件、Web事件驱动的用户界面框架。
它是Java EE的正式组成部分，但也可以单独使用，例如通过将Mojarra或MyFaces嵌入Tomcat中。

JSF的最新版本已与应用程序服务器中的CDI基础结构紧密联系在一起，其中一些新的JSF功能仅在这种环境下有效。

Spring对JSF支持不再活跃，主要是在现代化较旧的基于JSF的应用程序时出于迁移目的而存在。
Spring的JSF集成中的关键元素是JSFELResolver机制。

### Spring Bean解析器
SpringBeanFacesELResolver是符合JSF的ELResolver实现，与JSF和JSP使用的标准Unified EL集成。

它首先委派给Spring的上下文 WebApplicationContext，然后委派给底层JSF实现的默认解析器。

可以在JSF的配置文件faces-context.xml文件中定义SpringBeanFacesELResolver，如以下示例所示：
```
<faces-config>
    <application>
        <el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
        ...
    </application>
</faces-config>
```

### FacesContextUtils
在faces-config.xml中将属性映射到的bean时，自定义效果ELResolver很好。

有时可能需要显式地获取bean，使用FacesContextUtils，与WebApplicationContextUtils相似，它采用FacesContext参数而不是ServletContext参数。
以下示例显示如何使用FacesContextUtils：
```
ApplicationContext ctx = FacesContextUtils.getWebApplicationContext(FacesContext.getCurrentInstance());
```

## Apache Struts 2.x
Struts由Craig McClanahan发明，是由Apache Software Foundation托管的一个开源项目。

当时，它极大地简化了JSP / Servlet编程范例，并赢得了许多使用专有框架的开发人员的青睐。
它简化了编程模型，它是开源的，并且拥有庞大的社区，这使该项目得以发展并在Java Web开发人员中广受欢迎。

查看[Struts 2.x和Struts提供的Spring插件](https://struts.apache.org/plugins/spring/)提供内置的Spring集成。

## Apache Tapestry 5.x
Tapestry是一个“面向组件的框架，用于在Java中创建动态，健壮，高度可伸缩的Web应用程序。”

尽管Spring具有自己强大的Web层，但通过将Tapestry用于Web用户界面并将Spring容器用于较低层，构建企业Java应用程序具有许多独特的优势。

参考Tapestry的[Spring专用集成模块](https://tapestry.apache.org/integrating-with-spring-framework.html)。

## 更多资源
以下链接提供了有关本章中描述的各种Web框架的更多资源。
+ [JSF](https://www.oracle.com/technetwork/java/javaee/javaserverfaces-139869.html)主页
+ [Struts](https://struts.apache.org/)主页
+ [Tapestry](https://tapestry.apache.org/)主页