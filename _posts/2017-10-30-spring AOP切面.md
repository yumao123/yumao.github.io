---
title: Spring Aop切面
description: Null
categories:
 - Java
photos:
tags:
- Spring
---

> Spring的Aop切面
散布于应用中多处的功能被称为横切关注点，且从概念上与应用的业务逻辑分离，将横切关注点与业务逻辑分离就是面向切面编程要解决的问题
![Fate]({{ site.url }}/assets/images/201710/1030_top.png)

## AOP概述
一个被划分为模块的应用
![截图]({{ site.url }}/assets/images/201710/aop1.png)
如果要重用通用功能的话，最常见是继承 或 委托，然而两种方式都可能导致系统复杂化<br/>
切面提供了取代这些的另一种方案，更清晰简洁，通过声明的方式定义功能以何种方式在何处应用，无需修改受影响的类。横切关注点可以被模块化为特殊类，这些类就被称为切面。

- 术语<br/>
![截图]({{ site.url }}/assets/images/201710/aop2.png)
**通知(advice)**<br/>
AOP中，切面的工作称为通知，定义了切面是什么以及何时使用(调用前?调用后?抛出异常?)<br/>
分为:前置通知(before) 后置通知(after) 返回通知(after-returning) 异常通知(after-throwing) 环绕通知(around)<br/>
**切点(pointcut)**<br/>
一个切点不需要通知应用的所有连接点，切点有助于缩小切面所通知的连接点范围，通常使用明确类、方法，或者利用正则定义锁匹配的类、方法指定切点<br/>
**连接点(join point)**<br/>
连接点是应用执行过程中能够插入切面的一个点，可以是调用方法时，抛出异常，或者修改一个字段时；切面代码可以利用这些点插入到应用的正常流程，添加新行为<br/>
**切面(aspect)**<br/>
是通知和切点的结合，包括何时以及何处完成功能<br/>
**引入(introduction)**<br/>
引入允许我们向现有类添加新方法或属性，在不修改现有类的情况下，具有新行为和状态<br/>
**织入(weaving)**<br/>
是把切面应用到目标对象并创建新代理对象的过程；在目标对象生命周期里有多个点可以织入:编译期(需要特殊的编译器，Aspectj是已这种方式织入)、类加载期(需要特殊的类加载器，Aspectj支持这种方式织入)、运行期(Spring aop是已这种方式织入)<br/>

- Spring对AOP支持<br/>
1.基于代理的经典Spring AOP<br/>
2.纯POJO切面<br/>
3.@AspectJ注解驱动的切面<br/>
4.注入式AspectJ切面（适用于Spring各版本）<br/>
关于Spring在运行时通知对象
![截图]({{ site.url }}/assets/images/201710/aop2.png)

- spring只支持方法级别的连接点<br/>
keywords:通知,切点,连接点,切面,引入,织入

## 通过切点选择连接点
- 编写切点<br/>
```
package concert;
public interface Performance {
    public void perform();
}
```
使用AspectJ切点表达式
![截图]({{ site.url }}/assets/images/201710/aop4.png)
表达式*开始表示不关心方法返回值，如果需要匹配切点仅匹配concert包，可使用within()指示器
![截图]({{ site.url }}/assets/images/201710/aop5.png)

- 切点中选择bean<br/>
```
执行beanPerformance.perform应用通知，但是bean id限定为woodstock
execution(* concert.Performance.perform() and bean('woodstock'))
```

## 使用注解创建切面
```
@Component
public class TestPerformance implements Performance{
    public void perform() {
        System.out.println("TestPerformance perform");
    }
}
// 通过@Aspect进行标注该类是一个切面(通知类)
// AspectJ提供了5个注解定义通知
// @After @AfterReturning @AfterThrowing @Around @Before
@Aspect
public class Audience {
	// 通过@Pointcut可以在切面内定义可重用的切点
    @Pointcut("execution(* demo.Performance.perform(..))")
    public void performance(){}
    @Before("performance()")
    public void silenceCellPhones(){
        System.out.println("silenceCellPhones");
    }
}
```
即使使用AspectJ注解，但是该类也不会被视为切面，所以需要启用AspectJ注解
```
javaconfig:
@Configuration
@ComponentScan
// 启用自动代理功能
@EnableAspectJAutoProxy
public class AopConfig {
	// 声明切面Bean
    @Bean
    public Audience audience(){
        return new Audience();
    }
}
xmlconfig:
<aop:aspectj-autoproxy/>
<bean class="concert.Audience"/>
``
遇到的问题:<br/>
1.AspectJ使用maven的artifactId应该是aspectjweaver<br/>
2.aspectjweaver的版本需要与当前使用jdk版本一致,即jdk1.8对应的版本号是1.8.x,否则报错<br/>

- 创建环绕通知
```
    @Around("execution(* springdemo.CD.Performance.perform(..))")
    // ProceedingJoinPoint这个对象是必须要有的，因为要在通知中通过它来调用被通知的方法
    public void watchPerformance(ProceedingJoinPoint jp){
        try{
            System.out.println("before...");
            // 当要将控制权交给被通知的方法时，它需要调用ProceedingJoinPoint的proceed()方法
            // 如果不调用proceed方法，方法就会被阻塞
            jp.proceed();
            System.out.println("after...");
        }catch (Throwable e){
            System.out.println("Exception...");
        }
    }
```

- 处理通知中的参数 154

- 通过注解引入新功能

## 在xml中声明切面
如果要声明切面，但是又不能为类添加注解时，必须转向xml配置
配置方式:
```
<aop:advisor>定义AOP通知器
<aop:after>定义AOP后置通知（不管被通知的方法是否执行成功）
<aop:afterreturning>定义AOP返回通知
<aop:afterthrowing>定义AOP异常通知
<aop:around>定义AOP环绕通知
<aop:aspect>定义一个切面
<aop:aspectjautoproxy>启用@AspectJ注解驱动的切面
<aop:before>定义一个AOP前置通知
<aop:config>顶层的AOP配置元素。大多数的<aop:*>元素必须包含在<aop:config>元素内
<aop:declareparents>以透明的方式为被通知的对象引入额外的接口
<aop:pointcut>定义一个切点
```
```
    <bean id="audience" class="springdemo.CD.Audience"/>
    <bean id="testPerformance" class="springdemo.CD.TestPerformance"/>
    <aop:config>
        <aop:aspect ref="audience">
            <aop:pointcut id="perform" expression="execution(* springdemo.CD.TestPerformance.perform(..))"/>
            <aop:before method="silenceCellPhones" pointcut-ref="perform"/>
        </aop:aspect>
    </aop:config>
```
