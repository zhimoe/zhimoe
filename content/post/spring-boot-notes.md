+++
title = "Spring Boot Notes"
date = "2019-04-14T18:33:22+08:00"
categories = [ "编程",]
tags = [ "code", "spring",]
toc = "true"
+++


一些容易忘记的 spring boot 知识要点。
> 注意，.yaml 和.yml 文件没任何区别。

## 配置
## SpringBootApplication 注解

```java
@SpringBootApplication
// <=等价=>
@Configuration
@ComponentScan
@EnableAutoConfiguration

```

<!--more-->

## 自动配置
spring 的自动配置依赖以下注解：

![spring-boot-autoconfig-conditional](https://cdn.jsdelivr.net/gh/zhimoe/picx-images-hosting@master/pic/spring-boot-autoconfig-conditional.15t8ho20v6n4.webp)

## 配置文件
任何时候硬编码的配置总是不好的，spring 支持从很多环境中读取配置：配置文件，yaml 文件，环境变量，命令参数。
配置可以在`@Value`注解中使用，也可`Environment`访问，或者通过`@ConfigurationProperties`将配置属性绑定到特定的 bean([例子](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-typesafe-configuration-properties)).

spring boot 的配置属性读取顺序为：

- Devtools global settings properties on your home directory (~/.spring-boot-devtools.properties when devtools is active).
- @TestPropertySource annotations on your tests.
- @SpringBootTest#properties annotation attribute on your tests.
- Command line arguments.
- Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property)
- ServletConfig init parameters.
- ServletContext init parameters.
- JNDI attributes from java:comp/env.
- Java System properties (System.getProperties()).
- OS environment variables.
- A RandomValuePropertySource that only has properties in random.*.
- Profile-specific application properties outside of your packaged jar (application-{profile}.properties and YAML variants)
- Profile-specific application properties packaged inside your jar (application-{profile}.properties and YAML variants)
- Application properties outside of your packaged jar (application.properties and YAML variants).
- Application properties packaged inside your jar (application.properties and YAML variants).
- @PropertySource annotations on your @Configuration classes.
- Default properties (specified using SpringApplication.setDefaultProperties).

因为 spring-boot 主要使用的`application.properties/yaml`文件，所以后面主要关注这个文件。

此外，spring 代码中使用了大约近千个 (300 多类) 默认值，这些默认值都是可以覆盖的。只需你在你的 propeties/yaml 文件中用相同的 key 即可。
所有的参考值见：[example application.properties](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/html/common-application-properties.html)

## application.properties
SpringApplication loads properties from application.properties files in the following locations and adds them to the Spring Environment:

- A /config subdirectory of the current directory
- The current directory
- A classpath /config package
- The classpath root

## application.yml
yaml 是 json 的超集，相比 properties 文件，有着简洁灵活的优势 例如可以设置数组，设置 group 概念等。
yaml 文件可以配置数组：
```yaml
# 数组功能，等价
# my.servers[0]=dev.bar.com
# my.servers[1]=foo.bar.com
my:
   servers:
       - dev.bar.com
	   - foo.bar.com

#上面的配置可以通过注解绑定到以下 bean 中，非常强大。
@ConfigurationProperties(prefix="my")
public class Config {
	private List<String> servers = new ArrayList<String>();
}


# 在一个 yaml 文件设置不同的 profile 配置，properties 文件只能通过拆分文件`application-profiles.properties`实现。
server:
    address: 192.168.1.100
---
spring:
    profiles: DEV
server:
    address: 127.0.0.1
---
spring:
    profiles: PRD
server:
    address: 192.168.1.120

```

### yaml 缺点：
> YAML files cannot be loaded by using the `@PropertySource` annotation. So, in the case that you need to load values that way, you need to use a properties file.

当然使用 properties 文件缺点也明显，不能分组 (yaml 的---功能);同时中文显示容易 unicode 码。


## 读取配置文件
除了 application.properties 文件，其他的配置属性文件需要我们自己加载读取。注意，下面的`PropertySource`无法加载 yaml 文件。
### 使用 PropertySource
```properties
cron=0/3 * * * * ?

```

```java
@Configuration
@PropertySource("classpath:foo.properties")
public class PropertiesWithJavaConfig {
    @Value(${cron})
    private String cron;
    
}

//or
@PropertySource({ 
  "classpath:persistence-${envTarget:mysql}.properties"
})

//multi files
//java 8+
@PropertySource("classpath:foo.properties")
@PropertySource("classpath:bar.properties")
public class PropertiesWithJavaConfig {
    //...
}
//java 6+
@PropertySources({
    @PropertySource("classpath:foo.properties"),
    @PropertySource("classpath:bar.properties")
})
public class PropertiesWithJavaConfig {
    //...
}

//通过 xml 加载
//register file in xml
<context:property-placeholder location="classpath:foo.properties" />
//foo.properties in src/main/resources
<context:property-placeholder location="classpath:foo.properties, classpath:bar.properties"/>

```
### 如何加载自定义的 yaml 文件
上面提到 spring 会默认加载`application.yml`文件的配置。但是其他文件名的 yml 文件无法通过`@PropertySource`加载。可以有以下方法。
- 使用 xml，然后在 Java Config 类加载 xml. 个人不推荐使用 xml 文件，脱离 spring boot 的初衷了。
- 使用 yml 加载器：The `YamlPropertiesFactoryBean` will load YAML as `Properties` and the `YamlMapFactoryBean` will load YAML as a `Map`.
- 避免使用，尽量将你的所以配置放在 application.yml 里面，因为 yml 可以有分组功能。
- 将你文件命名为`application-redis.yml`,然后在 application.yml 使用`spring.profiles.include: 'redis'` 加载。

使用 yaml 文件的加载可以通过`ConfigurationProperties`绑定到配置 bean 中。还要添加 2 个注解注册到 spring:
```java
@Configuration
@EnableConfigurationProperties
@ConfigurationProperties
public class YAMLConfig {
  
    private String name;
    private String environment;
    private List<String> servers = new ArrayList<>();
 
    // standard getters and setters
}
```

```yml
spring:
    profiles: prod
name: prod-YAML
environment: production
servers: 
    - www.abc.com
    - www.xyz.com
```


### profiles
很多配置希望基于环境，spring boot 支持`application-profile.properties`格式的配置，profile 可以是 DEV,ST,UAT,PRD,TEST 等。
例如某个 class 希望只有在`PRD`环境才有：
```java
@Profile("PRD")
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {}

```
然后在`application.yml/properties`设置 profile:
```yaml
spring:
  profiles:
    active: PRD
```
### properties 文件设置 profile

`application.properties`文件只能使用`application-DEV.properties,application-ST.properties`设置 profile.

### yml 文件设置 profile
`application.yml`既可以像 properties 文件使用`application-DEV.yml`来设置 profile，也可以使用`---`分组。如下示例，`logging.level=INFO`在所有 profile 中生效，而在生产环境中增加日志文件设置，DEV 环境则使用`DEBUG`级别日志。
```yaml
# application.yml
logging:
  level:
   root: INFO
---
spring:
  profiles: DEV
logging:
  level:
    root: DEBUG
---
spring:
  profiles: PRD
logging:
  path: /tmp/
  file: BookWorm.log
  level:
    root: WARN
```
### 激活 profiles
在`application.yml/properties`文件中激活某个 profile:
```yaml
spring:
    profiles:
        active: DEV
```
如果你设置了`SPRING_PROFILES_ACTIVE`环境变量，那么会覆盖上面的 profile 设置。当然你也可以使用自定义环境变量和默认值：

```yaml
spring:
    profiles:
        active: ${ENV_TYP:PRD}
# 读取 ENV_TYP 环境变量的值作为激活 profile，如果没用这个环境变量，那么设置为 PRD.
```

## 测试