# 参考资料
+ [Mockito 教程](https://www.baeldung.com/mockito-series)
+ [Junit Mockito 解耦合测试](https://www.cnblogs.com/0201zcr/p/5886581.html)

# maven配置
## Junit
```
<dependency>  
	<groupId>junit</groupId>  
	<artifactId>junit</artifactId>  
	<version>4.11</version>  
	<scope>test</scope>  
</dependency> 
```
## mockito
```
<dependency>  
	<groupId>org.mockito</groupId>  
	<artifactId>mockito-all</artifactId>  
	<version>1.9.5</version>  
	<scope>test</scope>  
</dependency>  
```

# Injecting Mockito Mocks into Spring Beans
## springboot中引入mockito的maven依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.2.2.RELEASE</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.21.0</version>
</dependency>
```

## 业务代码
首先，让我们创建一个我们将要测试的简单服务NameService：
```
@Service
public class NameService {
    public String getUserName(String id) {
        return "Real user name";
    }
}
```


然后把它注入到UserService：
```
@Service
public class UserService {
	@Autowired
    private NameService nameService;
	
    public String getUserName(String id) {
        return nameService.getUserName(id);
    }
}
```

## 测试代码
首先，我们必须为测试配置应用程序上下文：
```
@Profile("test")
@Configuration
public class NameServiceTestConfiguration {
    @Bean
    @Primary
    public NameService nameService() {
        return Mockito.mock(NameService.class);
    }
}
```

单元测试：
```
@ActiveProfiles("test")
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MocksApplication.class)
public class UserServiceUnitTest {
 
    @Autowired
    private UserService userService;
 
    @Autowired
    private NameService nameService;
 
    @Test
    public void whenUserIdIsProvided_thenRetrievedNameIsCorrect() {
        Mockito.when(nameService.getUserName("SomeId")).thenReturn("Mock user name");
        String testName = userService.getUserName("SomeId");
        Assert.assertEquals("Mock user name", testName);
    }
}
```