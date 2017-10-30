---
title: Spring概述
description: Null
categories:
 - Java
photos:
tags:
- Spring
---

> Spring的Bean装配
![Fate]({{ site.url }}/assets/images/201710/1029_top.png)

## 自动装配
```
基于java的装配
@ComponentScan(basePackages = {"springdemo.CD"})
基于xml的配置
<context:component-scan/>
```
代码中的使用
```
@Component
public class CDPlayer implements MediaPlayer{
    private CompactDisc cd;
    @Autowired
    public CDPlayer(CompactDisc cd){
        this.cd = cd;
    }
    public void Play() {
        cd.play();
    }
}
```
```
// 自动创建spring应用上下文
@RunWith(SpringJUnit4ClassRunner.class)
// 通过CDPlayerConfig加载配置
@ContextConfiguration(classes = CDPlayerConfig.class)
public class CDPlayerTest {
    @Autowired
    private CompactDisc cd;
    @Autowired
    private MediaPlayer player;
    @Test
    public void cdShouldNotBeNull(){
        Assert.assertNotNull(cd);
    }
    @Test
    public void play(){
        player.Play();
    }
}
```
Keywords:@ComponentScan,<context:component-scan/>,@Component,@Autowired

## 基于Java的装配
```
// Configuration注解表明这个类时一个配置类，包括spring上下文如何创建bean的细节
@Configuration
// 自动装配这个包下的类
// @ComponentScan(basePackages = {"springdemo.CD"})
// 但是如果涉及一些第三方包，我们无法通过隐式的方式加载，就必须要显式配置
// 通过将其他java配置导入
@Import(AnotherConfig.class)
// java配置中加载xml配置
@ImportResource("classpath:CDPlayerXmlTest-context.xml")
public class CDPlayerConfig {

    // 生命简单的bean，返回一个对象
    @Bean
    public CompactDisc sgtPeppers(){
        return new SgtPeppers();
    }
    // 如果这样写，那么每个CDPlayer构造时都要有一份独立的CompactDisc，这显然时没有必要的
    @Bean
    public CDPlayer anotherCdPlayers(){
        return new CDPlayer(sgtPeppers());
    }
    // 通过请求一个CompactDisc作为参数，自动装配一个CompactDisc到配置方法
    @Bean
    public CDPlayer cdPlayer(CompactDisc cd){
        // 这里也可以通过setter的方式
        // CDPlayer cd = new CDPlayer();
        // cd.setCompactDisc(cd);
        // return cd;
        return new CDPlayer(cd);
    }
}
```
keywords:@Configuration,@Import,@ImportResource,@Bean

## 通过xml装配
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--文件名称必须以-context.xml结尾-->
    <!--c- 开头 与<constructor-arg>功能一致-->
    <!--p- 开头与<property>功能一致-->
    <bean id="compactDisc" class="springdemo.CD.SgtPeppers" />
    <bean id="mediaPlayer" class="springdemo.CD.CDPlayer" c:cd-ref="compactDisc" />
    <!--导入其他的xmlconfg配置文件-->
    <import resource="AnotherXmlConfig-context.xml"/>
    <!--导入其他的javaconfig配置文件-->
    <bean class="springdemo.CD.AnotherConfig"/>
    <!--不管使用javaconfig或xmlconfig，一般都会创建一个根配置，这个配置将更多的装配类或xml文件组装起来-->
    <!--并且在跟配置启用组件扫描-->
</beans>
```

## 环境profile
主要是为了生产环境/测试环境/开发环境中不同环境的bean装配(条件装配的一种)
```
// 可以在类加注解
// @Profile("dev")
public class DevelopmentProfileConfig {
    @Bean(destroyMethod = "shutdown")
    @Profile("dev")
    public DataSource dataSource(){
        return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2)
                .addScript("classpath:schema.sql").build();
    }
}
```
激活Profile
```
正式的web項目中，可以在web.xml中添加
  <context-param>
    <param-name>spring.profiles.default</param-name>
    <param-value>dev</param-value>
  </context-param>
测试项目中，可以
@ActiveProfile("dev")
public class CDPlayerConfig {}
```
keywords:@Profile,@ActiveProfile

## 条件化装配bean
首先要定义类实现接口Condition
```
public class MagicExistsCondition implements Condition {
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment env = conditionContext.getEnvironment();
        System.out.println(env.getActiveProfiles());
        return env.containsProperty("magic");
    }
}
```
使用
```
    @Bean
    @Conditional(MagicExistsCondition.class)
    public CompactDisc sgtPeppers(){
        return new SgtPeppers();
    }
```
这样，只有在上下文环境中有属性magic才会装配bean<br/>
keywords:@Conditional,Conditional

## 自动装配歧义性的处理方式
当自动装配@autowired的时候，如果有多个类都实现了这个接口，那么会抛出异常
- 首选bean<br/>
```
    @Bean
    @Primary
    public CompactDisc sgtPeppers(){
        return new SgtPeppers();
    }
```
问题:只能有一个被@Primary注解，若有多个实现同一接口的类注解则抛出异常
- 限定自装配bean<br/>
```
指定想要注入进去的是哪个bean
    @Autowired
    @Qualifier("cdPlayer")
    private MediaPlayer player;
创建自定义限定符
@Component
@Qualifier("cdPlayer")
public class CDPlayer implements MediaPlayer{}
以上是通过javaconfig配置的bean，则@Bean要与@Qualifier配合使用
    @Bean
    @Qualifier("cdPlayer")
    public CDPlayer cdPlayer(CompactDisc cd){
        return new CDPlayer(cd);
    }
```
问题:如果一个类需要有多个限定符修饰，那么使用多个@Qualifier修饰是非法的，但是可以通过自定义限定符实现
```
自定义限定符，需要被@Qualifier注解
@Qualifier
public @interface Creamy{}
@Qualifier
public @interface Cold{}
使用:
@Creamy
@Cold
public class IceCream implements Dessert{}
```
keywords:@Qualifier

## bean作用域
定义了多个作用域:Singleton(单例) Prototype(原型) Session(会话) Request(请求)
```
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class CDPlayer implements MediaPlayer{}
```
注意:如果一个bean注入的另一个bean生命周期比他小的话(例如单例与会话),则需要创建一个作用域代理?
```
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE, proxyMode=ScopedProxyMode.INTERFACES)
public class CDPlayer implements MediaPlayer{}
```
xml中声明
```
<bean id="cart" class="xxx" scope="session">
	<aop:scoped-proxy/>
</bean>
注意需要在xml中声明命名空间
```
keywords:@Scope,scope

## 运行时注入
- 属性占位符
```
javaconfig配置bean的方式
@PropertySource("classpath:springdemo/app.properties")
public class CDPlayerConfig {
    @Bean
    @Qualifier("cdPlayer")
    public CDPlayer cdPlayer(CompactDisc cd){
        return new CDPlayer(cd, env.getProperty("disc.name"));
    }	
}
这里将app.properties里的disc.name加入到构造函数中
```
解析属性占位符,占位符语法(${})

```
如果不通过javaconfig配置bean
<bean id="mediaPlayer" class="springdemo.CD.CDPlayer" c:cd-ref="compactDisc" c:name="${disc.name}">
或者在定义类的时候，通过@Value
@Component
public class CDPlayer implements MediaPlayer{
    private CompactDisc cd;
    private String name;
    @Autowired
    public CDPlayer(CompactDisc cd, @Value("${disc.name}") String name){
        this.cd = cd;
        this.name = name;
    }
}
```
为了使用占位符，必须配置PropertySourcesPlaceholderConfigurer
```
javaconfig配置:
    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer(){
        return new PropertySourcesPlaceholderConfigurer();
    }
xml配置:
<context:property-placeholder/>
```
keywords:@PropertySource,env.getProperty,${},PropertySourcesPlaceholderConfigurer,<context:property-placeholder/>

- Spring表达式语言(SpEL)

