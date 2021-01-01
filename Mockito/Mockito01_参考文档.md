# 参考资料
+ [Mockito 官网](https://site.mockito.org/)

# Mockito概述
Mockito，用于Java中单元测试的mocking框架。

Mockito 文档： [javadoc.io](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)。

# 快速开始
## 引入依赖
配置依赖参考： 
[Central Repository](https://search.maven.org/artifact/org.mockito/mockito-core) 
或 
[Bintray](https://bintray.com/mockito/maven/mockito/)。

Gradle：
```
repositories { jcenter() }
dependencies { testCompile "org.mockito:mockito-core:2.+" }
```

Maven：
```
<dependency>
  <groupId>org.mockito</groupId>
  <artifactId>mockito-core</artifactId>
  <version>2.28.2</version>
</dependency>
```

## verify interactions 验证交互
```
import static org.mockito.Mockito.*;

// mock creation
// 创建mock
List mockedList = mock(List.class);

// using mock object - it does not throw any "unexpected interaction" exception
// 使用模拟对象-它不会抛出任何“意外交互”异常
mockedList.add("one");
mockedList.clear();

// selective, explicit, highly readable verification
// 选择性、明确性、可读性强的验证
verify(mockedList).add("one");
verify(mockedList).clear();
```

## stub method calls 使用存根犯法，模拟数据
```
// you can mock concrete classes, not only interfaces
// 您可以模拟具体的类，而不仅仅是接口
LinkedList mockedList = mock(LinkedList.class);

// stubbing appears before the actual execution
// 在实际执行之前拦截
when(mockedList.get(0)).thenReturn("first");

// the following prints "first"
// 此处打印"first"
System.out.println(mockedList.get(0));

// 因为get（999）没有存根，所以下面打印“null”
System.out.println(mockedList.get(999));
```



# 相关知识
## stub和mock
Stub：关注状态验证。粗粒度的测试，在某个依赖系统不存在或者还没实现或者难以测试的情况下使用，例如访问文件系统，数据库连接，远程协议等。

Mock：关注行为验证。细粒度的测试，即代码的逻辑，多数情况下用于单元测试。

### Stub和Mock的相同处
stub和mock都是为了配合测试，对被测程序所依赖的单元的模拟。简单说，为了测函数A，但A有引用到了函数B，通过模拟B的一些状态或行为测试A。
Stub和Mock都是模拟外部依赖，以便我们能控制。

### Stub和Mock的区别
stub基于状态，mock基于行为。
stub难于维护。
mock有对本身的调用验证。
stub是基于状态，mock是基于行为。
Stub是完全模拟一个外部依赖， 而Mock用来判断测试通过还是失败。