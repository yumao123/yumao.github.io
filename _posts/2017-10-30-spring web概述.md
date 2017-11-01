---
title: Spring web概述
description: Null
categories:
 - Java
photos:
tags:
- Spring
---

> Spring web概述
![unkown]({{ site.url }}/assets/images/201711/1101_top.png)

## 关于流程
![截图]({{ site.url }}/assets/images/201711/springweb1.png)
1.请求 -> spring前端控制器DispatcherServlet
2.DispatcherServlet -> 查询handlermapping -> controller(将请求发送给对应的控制器)
3.conroller -> 服务对象处理业务逻辑
4.服务对象产生信息模型(model) -> view -> DispatcherServlet -> 视图解析器view resolver匹配视图名为一个特定视图的实现(例如jsp) -> 浏览器

```
SpitterWebInit:
/**
 * 在servlet3.0下，容器回去类路径中查找实现javax.servlet.servletcontainerinitializer接口的类
 * spring提供实现这个接口的实现类：springservletcontainerinitializer，反查实现了webapplicationinitializer类将配置任务交给他们完成
 * spring引入了WebApplicationInitializer基础实现，也就是AbstractAnnotationConfigDispatcherServletInitializer
 * 所以只要我们实现了AbstractAnnotationConfigDispatcherServletInitializer，也就同时实现了WebApplicationInitializer
 * 当部署在servlet3.0容器中时，容器会自动发现它，配置servlet上下文
 */
// 扩展AbstractAnnotation-ConfigDispatcherServletInitializer的任意类都会自动地 配置Dispatcher-Servlet和Spring应用上下文，Spring的应用上下
// 文会位于应用程序的Servlet上下文之中
public class SpitterWebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }
    @Override
    // 指定配置类 bean
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }
    @Override
    // DispatcherServlet映射到"/"
    /**
     * DispatcherServlet 和 ContextLoaderListener(servlet监听器)
     * DispatcherServlet启动时 - 创建spring上下文，加载bean(getServletConfigClasses)
     * 还有一个上下文，由ContextLoaderListener创建，加载应用中其他bean(驱动应用后端中间层和数据层组件)
     * GetServlet-ConfigClasses()会返回带有@Configuration注解的类，定义DispatcherServlet应用上下文中的bean
     * getRootConfigClasses()方法返回的带有@Configuration注解的类将会用来配置ContextLoaderListener创建的应用上下文中的bean
     */
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
WebConfig:
@Configuration
// 启用spring mvc
@EnableWebMvc
@ComponentScan(basePackages = {"spittr"})
public class WebConfig extends WebMvcConfigurationSupport{
    @Bean
    // 配置jsp视图解析器
    public ViewResolver viewResolver(){
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }
    @Override
    // 静态资源处理，将对静态资源的请求转发到servlet容器默认的servlet上，而不是使用DispatcherServlet
    protected void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
RootConfig:
@Configuration
@ComponentScan(basePackages = {"spittr"}, excludeFilters = {@ComponentScan.Filter(type= FilterType.ANNOTATION, value= EnableWebMvc.class)})
public class RootConfig {
}
```