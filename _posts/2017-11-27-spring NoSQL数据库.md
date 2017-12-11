---
title: Spring NoSQL
description: Null
categories:
 - Java
photos:
tags:
- Spring
- Sql
---

> Spring NoSQL
关于spring data mongodb/redis的基础使用
![TOP]({{ site.url }}/assets/images/201711/1127_top.png)

## 简介
文档数据库<br>
适合做：数据不需要分散到多个表or实体中，将信息收集到非规范化的结构更有意义；文档是独立的实体<br>
eg：学生成绩单，不同学生相互独立，其相互的成绩也没有必要关联<br>
不适合做：具有明显关联关系的数据<br>

## 使用mongodb
spring data mongodb提供了三种方式在spring中使用mongodb<br>
通过注解实现对象-文档映射；<br>
使用MongoTemplate实现基于模板的数据库访问；<br>
自动化的运行时Repository生成功能。<br>

- 启用mongodb<br>
```
编写mongodb配置类
@Configuration
@EnableMongoRepositories(
        basePackages = "spitter.db"
)
/**
* MongoTemplate:数据库与代码之间的接口，所有操作在这里面
* 通过继承AbstractMongoConfiguration，其回隐式的创建MongoTemplate
*/
public class MongoConfig{
    @Bean
    public MongoClientFactoryBean mongoClientFactoryBean(){
        MongoClientFactoryBean mongoClientFactoryBean = new MongoClientFactoryBean();
        mongoClientFactoryBean.setHost("localhost");
        return mongoClientFactoryBean;
    }
    @Bean
    public MongoOperations mongoTemplate(Mongo mongo){
        return new MongoTemplate(mongo, "OrdersDB");
    }
}
```

- 为模型添加注解，持久化文档
```
/**
* Document：借助MongoTemplate自动生成Repository持久化
*
*/
@Document
public class Order implements Serializable{
    @Field(value = "username")
    private String username;
    @PersistenceConstructor
    public Order(String username, String id) {
        this.username = username;
        this.id = id;
    }
    @Id
    private String id;
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
}
```

- 使用Mongotemplate访问
```
@Repository
public class OrderRepository {
    @Autowired
    private MongoOperations template;
    public List<Order> getOrders(){
        int id = (new Random().nextInt(11000));
        template.save(new Order("mike", String.valueOf(id)));
        return template.findAll(Order.class);
    }
}
```
注：<br>
1：遇到问题，spring data mongodb与spring的版本兼容问题<br>
2：还需要引入mongo-java-driver<br>

## 使用redis操作key-value数据
提供了4种客户端连接工厂<br>
JedisConnectionFactory<br>
JredisConnectionFactory<br>
LettuceConnectionFactory<br>
SrpConnectionFactory<br>
- 连接redis<br>
```
    @Bean
    public JedisConnectionFactory redisConnectionFactory(JedisPoolConfig jedisPoolConfig){
        JedisConnectionFactory jf = new JedisConnectionFactory();
        jf.setPoolConfig(jedisPoolConfig);
        jf.setHostName("localhost");
        jf.setPort(6379);
        jf.afterPropertiesSet();
        return jf;
    }
```
- 使用redisTemplate<br>
提供了两个模板<br>
RedisTemplate<br>
StringRedisTemplate<br>
```
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<String, String>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
```
- 使用key value序列器<br>
stringRedisTemplate默认使用String序列化策略
redisTemplate默认使用JdkSerializationRedisSerializer序列化策略<br>
指定序列化策略：
```
默认提供的序列化策略：
GenericToStringSerializer：使用Spring转换服务进行序列化；
JacksonJsonRedisSerializer：使用Jackson 1，将对象序列化为JSON；
Jackson2JsonRedisSerializer：使用Jackson 2，将对象序列化为JSON；
JdkSerializationRedisSerializer：使用Java序列化；
OxmSerializer：使用Spring O/X映射的编排器和解排器（marshaler和unmarshaler）实现序列化，用于XML序列化；
StringRedisSerializer：序列化String类型的key和value。
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<String, String>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<Order>(Order.class));
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
```
否则如果使用默认的redisTemplate可能会乱码