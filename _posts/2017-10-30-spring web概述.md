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
主要内容有：如何启动springweb；如何建立简单的控制器且返回视图；如何接收处理来自客户端的请求
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

- 传递模型到视图中<br/>
```
DataConfig:
@Configuration
public class DataConfig {
    @Bean
    public DataSource dataSource(){
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScripts("schema.sql", "data.sql")
                .build();
    }
    @Bean
    public JdbcOperations jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
JDBCSpittleRepository:
@Repository
public class JDBCSpittleRepository implements SpittleRepository {
    @Autowired
    private JdbcOperations jdbc;
    public List<Spittle> getSpittles() {
        return jdbc.query("select * from Spittle", new SpittleRowMapper());
    }
    private static class SpittleRowMapper implements RowMapper<Spittle> {
        public Spittle mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new Spittle(
                    rs.getLong("id"),
                    rs.getString("message"),
                    rs.getDate("created_at"),
                    rs.getDouble("longitude"),
                    rs.getDouble("latitude"));
        }
    }
}
Controller:
@Controller
@RequestMapping("/spittles")
public class SpittleController {
    @Autowired
    private SpittleRepository spittleRepository;
    @RequestMapping(method = RequestMethod.GET)
    public String spittles(Model model){
        model.addAttribute("spittleList", spittleRepository.getSpittles());
        return "spittles";
    }
}
```
注:@Repository：DAO组件类<br/>
需要通过maven导入h2，spring-jdbc的依赖<br/>

## 后端接受请求输入
springmvc接收的请求有<br/>
查询参数（Query Parameter）<br/>
表单参数（Form Parameter）<br/>
路径变量（Path Variable）<br/>

- 查询参数
```
    @RequestMapping(method = RequestMethod.GET)
    public List<Spittle> spittles(Model model, @RequestParam(defaultValue = MAX_LONG, value = "max") Long max){
        System.out.println(max);
        return spittleRepository.getSpittles();
    }
```
- 路径参数
```
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public String spittlesByPath(Model model, @PathVariable("id") Long id){
        System.out.println(id);
        model.addAttribute("spittleList", spittleRepository.getSpittles());
        return "spittles";
    }
    注：这里不能直接返回List<Spittle>,因为url最后一个是id,逻辑视图的名称将会根据请求路径推断得出
```
Spring MVC允许我们在@RequestMapping路径中添加占位符。占位符的名称要用大括号（“{”和“}”）括起来。路径中的其他部分要与所处理的请求完全匹配，但是占位符部分可以是任意的值<br/>

keywords:DispatcherServlet,@EnableWebMvc,ViewResolver,@Repository,@Controller,@RequestMapping,@RequestParam,@PathVariable,Model

## 处理表单
```
    @RequestMapping(value = "register", method = RequestMethod.POST)
    // 当处理注册表单的POST请求时，控制器需要接受表单数据并将表单数据保存为Spitter对象
    public String register(Spitter spitter){
        spitterRepository.save(spitter);
        // 当InternalResourceViewResolver看到视图格式中的“redirect:”前缀时，它就知道要将其解析为重定向的规则，而不是视图的名称
        return "redirect:/spitter/" + spitter.getUsername();
    }
    @RequestMapping(value = "/{username}", method = RequestMethod.GET)
    public String showSpitter(@PathVariable("username") String username, Model model){
        model.addAttribute(spitterRepository.findByUsername(username));
        return "profile";
    }
registerForm:
<form method="POST">
    First Name: <input type="text" name="firstName" /><br/>
    Last Name: <input type="text" name="lastName" /><br/>
    Email: <input type="email" name="email" /><br/>
    Username: <input type="text" name="username" /><br/>
    Password: <input type="password" name="password" /><br/>
    <input type="submit" value="Register" />
</form>
```
keywords:redirect:















