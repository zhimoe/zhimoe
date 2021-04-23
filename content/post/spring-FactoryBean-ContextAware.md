---
title: "Spring FactoryBean and ContextAware"
date: "2019-05-16T22:44:38+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - spring
---

## FactoryBean
一句话就是FactoryBean用于返回其他对象实例的,而不是自身类型的实例.
例如:

```java

public class Tool {
 
    private int id;
 
    // standard constructors, getters and setters
}

public class ToolFactory implements FactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    @Override
    public Tool getObject() throws Exception {
        return new Tool(toolId);
    }
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    public boolean isSingleton() {
        return false;
    }
 
    // standard setters and getters
}
```
注册Tool:
```xml
<!-- factorybean-spring-ctx.xml -->
<beans>
 
    <bean id="tool" class="com.baeldung.factorybean.ToolFactory">
        <property name="factoryId" value="9090"/>
        <property name="toolId" value="1"/>
    </bean>
</beans>

```
使用注解注册:
```java

@Bean(name = "tool")
ToolFactory toolFactory() {
    ToolFactory factory = new ToolFactory();
    factory.setFactoryId(7070);
    factory.setToolId(2);
    return factory;
}

```

使用Tool:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
    @Autowired
    private Tool tool;

    @Test
    public void testConstructWorkerByXml() {
        assertThat(tool.getId(), equalTo(1));
    }
}
```

访问ToolFactory,在bean id前面添加 `&`:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
 
    @Resource(name = "&tool")
    private ToolFactory toolFactory;
 
    @Test
    public void testConstructWorkerByXml() {
        assertThat(toolFactory.getFactoryId(), equalTo(9090));
    }
}
```

## 和BeanFactory的区别
除了FactoryBean,还有一个BeanFactory的接口及其实现.



