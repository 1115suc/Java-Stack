# 🚀 快速集成 Spring Cache

## 🌟 缓存层选择

![image-20210710235405378.png](img/image-20210710235405378.png)

**选择Face的理由：**

- controller层功能过于粗狂、组装数据返回前端，不易缓存的维护；
- service的功能过于细腻，切关联甚广；
- 使用face处理缓存等一些特殊场景，与开发服务逻辑隔离，方便维护；

## 🔧 1. 添加 Maven 依赖

```xml
    <!--不要将缓存放在中间common层，因为如果引用common的第三方不适用缓存，会导致因为场景依赖自动装配的机制导致启动失败-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <!--引入redis的starter依赖-->
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- redis创建连接池，默认不会创建连接池 -->
    <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    </dependency>
```


## ⚙️ 2. 配置 Redis 连接

在 `application.yml` 中添加 Redis 配置：

```yaml
spring:
  redis:
    host: 192.168.88.128 # 默认localhost
    port: 6379   #默认是6379
    password:    #如果redis服务没有配置密码，则可不写
    database: 0  #默认操纵redis的0分片的数据 ，可省略不写
    lettuce:
      pool:
        max-active: 8  # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8    # 连接池中的最大空闲连接
        min-idle: 1    # 连接池中的最小空闲连接
    timeout: PT10S     # 连接超时时间
```


## 🔧 3. 启用缓存功能

创建缓存配置类并启用缓存：

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {
    @Bean
    public StringRedisSerializer stringRedisSerializer(){
        return new StringRedisSerializer();
    }
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        //定义redis数据序列化的对象
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        //jackson序列化方式对象
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        //设置被序列化的对象的属性都可访问：暴力反射
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        //仅仅序列化对象的属性，且属性不可为final修饰
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        serializer.setObjectMapper(objectMapper);
        // 配置key value序列化
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer))
                //关闭控制存储--》禁止缓存value为null的数据
                .disableCachingNullValues()
                //修改前缀与key的间隔符号，默认是::  eg:name:findById
                .computePrefixWith(cacheName->cacheName+":");

        //设置特有的Redis配置
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        //定制化的Cache 设置过期时间 eg:以role：开头的缓存存活时间为10s
        //cacheConfigurations.put("role",customRedisCacheConfiguration(config,Duration.ofSeconds(20)));
        cacheConfigurations.put(StockConstant.STOCK,customRedisCacheConfiguration(config, Duration.ofHours(24)));
        //构建redis缓存管理器
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                //Cache事务支持,保证reids下的缓存与数据库下的数据一致性
                .transactionAware()
                .withInitialCacheConfigurations(cacheConfigurations)
                .cacheDefaults(config)
                .build();
        //设置过期时间
        return cacheManager;
    }

    public RedisCacheConfiguration customRedisCacheConfiguration(RedisCacheConfiguration config, Duration ttl) {
        return config.entryTtl(ttl);
    }
}
```


## 🎯 4. 使用缓存注解

在 Face 包中使用缓存注解：

```java
@Component
@CacheConfig(cacheNames = StockConstant.STOCK)
public class StockCacheFaceImpl implements StockCacheFace {

    @Autowired
    private StockBusinessMapper stockBusinessMapper;
    
    @Cacheable(key = "#root.method.getName()")
    @Override
    public List<String> getAllStockCodeWithPredix() {
        //1.获取所有A股股票的编码
        List<String> allCodes = stockBusinessMapper.getAllStockCodes();
        //2.添加股票前缀 sh sz
        List<String> prefixCodes = allCodes.stream().map(code -> {
            code = code.startsWith("6") ? "sh" + code : "sz" + code;
            return code;
        }).collect(Collectors.toList());
        return prefixCodes;
    }
    
    @CacheEvict(key = "'getAllStockCodeWithPredix'")
    @Override
    public void updateStockInfoById(StockBusiness info) {
        stockBusinessMapper.updateByPrimaryKeySelective(info);
    }
}
```


## ✅ 5. 完成集成

现在你的项目已经成功集成了 Spring Cache，可以直接在业务方法上使用 `@Cacheable`、`@CachePut`、`@CacheEvict` 等注解来实现缓存功能。