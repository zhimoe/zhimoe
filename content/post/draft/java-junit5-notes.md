+++
title = 'Junit5 单元测试完整指南'
date = '2023-11-25T21:38:57+08:00'
categories = ['编程']
tags = ['junit5','java']
toc = true
draft = true
+++

本文包含三部分：Junit5 的基本使用和 Junit4 迁移、Mockito 与 Junit5 的集成、在 SpringBoot3 中使用 Junit5 与 Mockito

<!--more-->

## Junit5 的基本使用与 Junit4 迁移

### Core Principles: Prefer extension points over features
[Core Principles](https://github.com/junit-team/junit5/wiki/Core-Principles)

### 基本注解
除了`@Test`外最常用的是`@ParameterizedTest`，配合`@CsvSource`等注解可以简化准备测试案例的代码，详细的教程可以看[JUnit 5 Parameterized Tests](https://baeldung.com/parameterized-tests-junit-5)

`@RepeatedTest`一般用于测试缓存或者副作用等功能。

`@TestFactory`用于生成运行时测试。被注解的方法返回值必须是`Stream<DynamicTest>`、`Iterator<DynamicTest>`、`Iterable<DynamicTest>`、`Collection<DynamicTest>`中的一种：
```java
@TestFactory
Stream<DynamicTest> dynamicTestsFromStreamInJava8() {
        
    DomainNameResolver resolver = new DomainNameResolver();
        
    List<String> domainNames = Arrays.asList(
      "www.somedomain.com", "www.anotherdomain.com", "www.yetanotherdomain.com");
    List<String> outputList = Arrays.asList(
      "154.174.10.56", "211.152.104.132", "178.144.120.156");
        
    return inputList.stream()
      .map(dom -> DynamicTest.dynamicTest("Resolving: " + dom, 
        () -> {int id = inputList.indexOf(dom);
 
      assertEquals(outputList.get(id), resolver.resolveDomain(dom));
    }));       
}
```

`@ParameterizedTest`只是用不同参数多次执行一个测试方法。而 Test Templates 则支持使用不同的配置条件多次执行一个测试方法。这里的配置条件包含不同的 Mock 依赖，不同的 callback 等。本质上，Repeated Tests and Parameterized Tests are built-in specializations of test templates:
```java
@TestTemplate
@ExtendWith({ParameterizedTestExtension.class})
public @interface ParameterizedTest {}

class ParameterizedTestExtension implements TestTemplateInvocationContextProvider 
```
下面看一个 TestTemplate 具体的例子
```java
// TODO:
https://www.programcreek.com/java-api-examples/?code=domaframework%2Fdoma%2Fdoma-master%2Fdoma-processor%2Fsrc%2Ftest%2Fjava%2Forg%2Fseasar%2Fdoma%2Finternal%2Fapt%2Fprocessor%2Fdomain%2FExternalDomainProcessorTest.java

```


### 修改测试 display name
`src/test/resources/junit-platform.properties`中配置
``` yaml
junit.jupiter.displayname.generator.default = \
    org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```


### 常用 assert
```java
import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTimeout;
import static org.junit.jupiter.api.Assertions.assertTimeoutPreemptively;
import static org.junit.jupiter.api.Assertions.assertTrue;
```
### Junit4 迁移
看官方文档最直接[migrating-from-junit4](https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4)
| Junit4          | Junit5   | 说明      |
| --------------- | -------- | -------- |
| @RunWith  | @ExtendWith  |    |
| @Before @After  | @BeforeEach @AfterEach  |     |
| @BeforeClass @AfterClass | @BeforeAll  @AfterAll   |    |
| @Ignore | @Disabled   | @DisabledOn...  |
| @Category | @Tag @IncludeTags   |    |
| @Rule @ClassRule | @ExtendWith @RegisterExtension  |    |
| @Test(expected = …​) ExpectedException | Assertions.assertThrows(…​)   |   |


### Mockito notes

1. If you are using argument matchers, **all** arguments have to be provided by matchers.
   
```java
verify(mock).someMethod(anyInt(), anyString(), "third argument");
//above is incorrect - exception will be thrown because third argument is given without an argument matcher.
```
2. Matcher methods like any(), eq() do not return matchers. Internally, they record a matcher on a stack and return a dummy value (usually null). This implementation is due to static type safety imposed by the java compiler. The consequence is that you cannot use any(), eq() methods outside of verified/stubbed method.
   
3.  MockitoAnnotations.openMocks(testClass);
 
You can use built-in runner: MockitoJUnitRunner or a rule: MockitoRule. For JUnit5 tests, refer to the JUnit5 extension described in section 45.
Read more here: MockitoAnnotations


4. when(mock.someMethod("some arg"))
   .thenThrow(new RuntimeException())
   .thenReturn("foo");

5. doReturn(Object)

doThrow(Throwable...)

doThrow(Class)

doAnswer(Answer)

doNothing()

doCallRealMethod()

6. 

   List list = new LinkedList();
   List spy = spy(list);

   //Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
   when(spy.get(0)).thenReturn("foo");

   //You have to use doReturn() for stubbing
   doReturn("foo").when(spy).get(0);
 
Mockito *does not* delegate calls to the passed real instance, instead it actually creates a copy of it. So if you keep the real instance and interact with it, don't expect the spied to be aware of those interaction and their effect on real instance state. The corollary is that when an *un-stubbed* method is called *on the spy* but *not on the real instance*, you won't see any effects on the real instance.
Watch out for final methods. Mockito doesn't mock final methods so the bottom line is: when you spy on real objects + you try to stub a final method = trouble. Also you won't be able to verify those method as well.

7. Car boringStubbedCar = when(mock(Car.class).shiftGear()).thenThrow(EngineNotStarted.class).getMock();
8. assertEquals("foo", Foo.method());
 try (MockedStatic mocked = mockStatic(Foo.class)) {
 mocked.when(Foo::method).thenReturn("bar");
 assertEquals("bar", Foo.method());
 mocked.verify(Foo::method);
 }
 assertEquals("foo", Foo.method());
 