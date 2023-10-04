+++
title = "云原生Java开发框架Quarkus学习笔记"
date = "2022-07-09T22:14:10+08:00"
categories = [ "编程",]
tags = [ "code", "quarkus",]
toc = "true"
+++


## 什么是 MicroProfile

MicroProfile是一个微服务的平台定义, 目标是针对微服务架构优化企业Java开发. 由于JavaEE的标准更新越来越慢, 跟不上Web技术与K8S的发展, 于是一组供应商（包括Tomitribe）决定创建MicroProfile, 这是一个优化的微服务架构平台, 在2016年加入Eclipse基金会.
[MicroProfile](https://microprofile.io/compatible/5-0/)是一组规范, 包含如OpenTracing 、OpenAPI 、RestClient、Config、 FaultTolerance、 CDI等一组标准.当前最新标准是5.0. 各大Java厂商有很多实现, 最有名的就是红帽的Quarkus, 其他实现有Open Liberty和Payara Enterprise.
注意SpringBoot不是MicroProfile规范实现, Boot是独立于MicroProfile和JavaEE规范的, 但是功能上大同小异, Quarkus也提供了Spring注解的支持.
<!--more-->

## Quarkus

[Quarkus](https://quarkus.io/)是一个MicroProfile规范的实现, 专门为云时代打造.有：启动时间短, 内存占用小, 支持native编译（部署在GraalVM), 支持K8S特性(不仅是部署, 还包括自动生成K8S资源文件等)优势. 本质上是精选了一些优质组件, 通过扩展(extensions)模式提供快速业务开发的能力.

### 创建项目
Quarkus提供了强大Cli、Maven插件、Gradle插件支持. 以下主要使用maven.

```shell
mvn io.quarkus.platform:quarkus-maven-plugin:2.10.2.Final:create \
    -DprojectGroupId=moe.zhi \
    -DprojectArtifactId=quarkus-demo \
    -Dextensions="resteasy-reactive-jackson" 
cd quarkus-demo
```

打开IDE,可以看到自动包含pom.xml和Dockerfile(分为不同目标部署环境, 每个文件提供了详细使用说明).项目直接运行就是一个hello world api服务. 如果需要增加一个扩展, 通过maven add-extension插件的extensions参数添加, 插件自动会修改pom.xml文件, 添加对应的maven lib.

```shell
# add Caffeine cache support 
 mvn quarkus:add-extension -Dextensions="cache"
```

quarkus 目前已经有上百个extension, 可以通过`mvn quarkus:list-extensions`查看所有.也可以在[quarkus doc](https://quarkus.pro/extensions/)查看.

### 注解
Quarkus的注解符合Java CDI规范. DI部分的注解基本可以和Spring的做一一对上.

```text
//Spring       -> CDI/MicroProfile
@Autowired     -> @Inject
@Qualifier     -> @Named
@Value         -> @ConfigProperty(ConfigMap用于分组配置key的公共前缀,ConfigProperties已经废弃)
@Component     -> @Singleton
@Configuration -> @ApplicationScoped
@Bean          -> @Produces
```

### 配置

和Spring类似, Quarkus使用一个application.properties文件配置属性.
使用@ConfigProperty读取配置属性.如果属性是list, 使用逗号分隔.
配置文件中都是String和Int, MicroProfile Configuration自带了一系列的转换器：

```shell
boolean: true、1、YES、Y、ON为true, 其他为false
Byte,Short,Integer,Long,Float,Double,Character,Class(Class.forName)自动转换
目标类型有 public static T of(String) 或者public static T valueOf(String) 方法
目标类型有 public static T parse(CharSequence) 方法
目标类型有 public constructor(String)构造函数
```

自定义转换器参考`org.eclipse.microprofile.config.spi.Converter`

#### 配置属性的验证

需要`quarkus-hibernate-validator`扩展, 然后使用@Max、@Digits、@Email、@NotNull和@NotBlank等校验注解.
自定义校验器需要实现`javax.validation.ConstraintValidator`接口.

#### 自定义配置源

参考`org.eclipse.microprofile.config.spi.ConfigSource`

#### 获取环境变量

使用@Inject Config config的getPropertyNames()获得所有属性.

#### 属性来源优先级

![config property order](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/config-order.7ccnsry6sys0.webp)

#### Profile

Quarkus自带三个profile环境:dev, test, prod.
设置不同的profile属性使用`%{profile}.config.key=value`.例如

```shell
%dev.quarkus.http.port=8181
```

#### logging level

Quarkus内部使用 JBoss Logging, 如果需要使用Slf4j,添加依赖:

```xml
<dependency>
    <groupId>org.jboss.slf4j</groupId>
    <artifactId>slf4j-jboss-logmanager</artifactId>
</dependency>
```

```shell
quarkus.log.level=DEBUG
quarkus.log.file.enable=true

# 调整package下面的log level, 注意双引号
quarkus.log.category."io.undertow.request.security".level=TRACE
```

支持集中管理日志, 参考logging-json扩展.

#### quarkus生命周期事件

注入`io.quarkus.runtime.StartupEvent`和`io.quarkus.runtime.ShutdownEvent`事件即可响应.
```java
@ApplicationScoped
@Slf4j
public class AppEventListener {
    void onStart(@Observes StartupEvent event) {
        log.info("#### app started...");
    }

    void onShutdown(@Observes ShutdownEvent event) {
        log.info("### app shutdown...");
    }
}
```


#### 拦截器

首先通过`@javax.interceptor.InterceptorBinding`创建一个拦截器注解.

```java

@Inherited
@InterceptorBinding
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface Logit {
}
```

然后通过`@javax.init.AroundInvoke`和`@javax.init.AroundConstruct`两个具有相应拦截绑定功能的拦截器.

```java
@Logit
@Interceptor
public class LogitInterceptor {

    @AroundInvoke
    public Object invoke(InvocationContext ctx) throws Exception {
        System.out.println("#### interceptor" + ctx.getMethod().getName());
        return ctx.proceed();
    }
}
```

### Restful服务

Quarkus实现了JAX-RS规范, 支持@GET,@POST, @PUT等http动词注解.默认设置下, quarkus使用RESTEasy和Vertx框架, 而不是Servlet规范.如果需要使用Servlet, 则增加`quarkus-undertow`扩展, 这是JBOSS（wildfly)的Servlet服务器的引擎.
通过`@Path`注解匹配URI.
通过`@Context UriInfo uriInfo`获取url内容.
通过`@QueryParam("p")`参数注解获取QueryString中的参数.
其他参数：表单参数 (`@FormParam`)、矩阵参数(`@MatrixParam`)或cookie值 (`@CookieParam`).
此外, 使用`@Context`注解, 你还可以注入其他与 JAX-RS相关的元素, 如`javax.ws.rs.core.SecurityContext`、
`javax.ws.rs.sse.SseEventSink`或`javax.ws.rs.sse.Sse`.

json-p(processing)和json-b(binding)是JavaEE规范.如果API需要返回json,需要增加`quarkus-testeasy-jsonb`扩展.
如果data model的字段与json字段不同名, 需要使用`javax.json.bind.annotation.JsonbProperty`注解绑定.

```java
class Stu {
    @Jsonbproperty("first-name")
    String firstNm;
}
```

如果使用jackson, 则需要`quarkus-resteasy-jackon`扩展, 使用`com.fasterxml.jackson.databind.ObjectMapper`做json转换.

### Rest客户端

使用rest客户端调用外部服务接口, 需要增加"rest-client"和"resteasy-jsonb"扩展.

```java

private Client rest=ClientBuilder.newClient();

public String getTime(){
        Response resp=rest.target("http://worldclockapi.com")
        .path("/api/json/{timezone}/now")
        .resolveTemplate("timezone","GMT")
        .request(MediaType.APPLICATION_JSON)
        .get(Response.class);

        return resp.readEntity(String.class);
}

```

### 持久化

Agroal是Quarkus中首选的数据源和连接池实现,与安全、事务管 理和健康指标进行了集成. 虽然它是一个扩展程序,但如果你正在使 用Hibernate ORM或Panache,Agroal扩展会被顺带加载进来. 之后你 还需要一个数据库驱动扩展,目前,H2、PostgreSQL、MariaDB、 MySQL、Microsoft SQL Server和Derby都有支持的扩展,可以通过 Maven add-extension添加正确的数据库驱动扩展

如果你想要使用响应式编程, 也可以使用Vert.x reactive drivers.

```shell
# 配置多个数据源的话在 quarkus.datasource后面跟着一个自定义的ds name即可.例如
# configure your datasource
quarkus.datasource.url = jdbc:postgresql://localhost:5432/library-database
quarkus.datasource.driver = org.postgresql.Driver
quarkus.datasource.username = melvil
quarkus.datasource.password =  dewey
# 配置第二个数据源 指定ds name 为 orders
quarkus.datasource.orders.url = jdbc:postgresql://localhost:5432/library-database
quarkus.datasource.orders.driver = org.postgresql.Driver
quarkus.datasource.orders.username = melvil
quarkus.datasource.orders.password =  dewey

```

非默认数据源需要使用@DataSource("ds name")指定:

```java
import javax.inject.Inject;

@Inject
DataSource("orders")
AgroalDatasource ordersDs;

```

### 容错

需要添加`quarkus-smallrye-fault-tolerance`扩展. 

回退与重试：`@org.eclipse.microprofile.faulttolerance.Retry`, `@org.eclipse.microprofile.faulttolerance.Fallback`. Fallback的handler需要实现`org.eclipse.micropro file.faulttolerance.FallbackHandler`接口 

超时： `@org.eclipse.microprofile.faultttoler ance.Timeout`.

过载保护(并发请求个数)： `@org.eclipse.microprofile.faultttolerance.Bulkhead`.
`压测 >siege -r 1 -c 4 -v http:localhost:8080/hello/bulkhead`

断路器： `@CircuitBreaker`
```java
@CircuitBreaker(requestVolumeThreshold = 4, // <1> 滚动窗口
        failureRatio = 0.75, // <2>断路阈值
        delay = 2000) // <3> 重新打开时长ms
```  

- 如果使用@Fallback, 且CircuitBreakerOpenException被抛出, 回 退逻辑将被执行.
- 如果使用@Retry, 每次重试都由断路器处理, 并记录成功或失 败.
- 如果使用@Bulkhead, 则在试图进入bulkhead之前检查断路器.

### 可观察

#### health

1. health： 添加了`quarkus-smallrye-health`自动注册 `q/health/live`和`q/health/ready`两个探针
2. 自定义：实现一个 `org.eclipse.microprofile.health.HealthCheck`接口, 并加上`@org.eclipse.microprofile.health.Liveness`
   或`@org.eclipse.microprofile.health.Readiness`注解

#### 指标

1. 添加`quarkus-smallrye-metrics`扩展, 自动暴露`q/metrics`端点.
2. 自定义指标： `@Counted` `@Gauge` `@Metered` `@Timed`

### OpenTelemetry

[Quarkus Opentelemetry configuration](https://quarkus.io/guides/opentelemetry#configuration-reference)

### Reactive编程

Quarkus支持两种响应式编程方式：

- Reactive Programming with Mutiny
- Coroutines with Kotlin
  
Quarkus支持混合命令式与响应式编程, 所以不需要刻意将所有代码改造成响应式就可以享受到高性能.更多见：
[unification-of-imperative-and-reactive](https://quarkus.io/guides/quarkus-reactive-architecture#unification-of-imperative-and-reactive
)
### class loading in quarkus

production模式和native image模式下, quarkus的类加载器都是 system ClassLoader(native模式不支持多ClassLoaders)
所有quarkus app都是通过`QuarkusBootstrap` class创建.这个类解析app所有的相关依赖(编译或运行时的依赖),最终得到一个`CuratedApplication` class, 这个类包含这个app所有的类加载信息.

`CuratedApplication` 可以用于创建一个`AugmentAction` 实例, 这个实例用于创建app并启动/重启.

在dev模式下, quarkus通过classloader支持热加载, 在prod模式下, 只有 system ClassLoader.
除了热加载, 在dev模式下, 提供了`q/dev` DEV UI,支持配置应用, 查看缓存, 查看类信息, 查看/执行定时任务, 查看健康状态, 执行数据脚本迁移等等.