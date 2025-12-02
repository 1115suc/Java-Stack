# 🚀 Spring Cache 缓存框架

## 🔧 Spring Cache 使用

### 📦 引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

---

## ⚙️ 核心配置详解

### 🛠️ 基础配置

```yaml
spring:
  redis:
    host: 192.168.88.128     # Redis服务器地址
    database: 1              # Redis数据库索引（默认为0）
    port: 6379               # Redis服务器连接端口
    password:                # Redis服务器连接密码（默认为空）
```


### 🔧 缓存管理器配置

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {
    // 配置 cacheManager 代替默认的cacheManager （缓存管理器）
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        //仅仅序列化对象的属性，且属性不可为final修饰
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        serializer.setObjectMapper(objectMapper);
        // 配置key value序列化
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer))
                //关闭控制存储
                .disableCachingNullValues()
                //修改前缀与key的间隔符号，默认是::
                .computePrefixWith(cacheName->cacheName+":");

        //设置特有的Redis配置
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        //定制化的Cache 设置过期时间 eg:以role：开头的缓存存活时间为10s
        cacheConfigurations.put("role",customRedisCacheConfiguration(config,Duration.ofSeconds(10)));
        cacheConfigurations.put("stock",customRedisCacheConfiguration(config,Duration.ofSeconds(3000)));
        cacheConfigurations.put("market",customRedisCacheConfiguration(config,Duration.ofSeconds(300)));
        //构建redis缓存管理器
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                //Cache事务支持
                .transactionAware()
                .withInitialCacheConfigurations(cacheConfigurations)
                .cacheDefaults(config)
                .build();
        //设置过期时间
        return cacheManager;
    }
    
    public RedisCacheConfiguration customRedisCacheConfiguration(RedisCacheConfiguration config, Duration ttl) {
        //设置缓存缺省超时时间
        return config.entryTtl(ttl);
    }
}
```


---

## 📖 缓存注解详解

### 🎯 @Cacheable 注解

当方法被调用时，会先检查缓存中是否存在对应的结果，如果存在则直接返回缓存数据，否则执行方法并将结果存入缓存

| 属性 | 类型 | 作用 | 实例 |
|------|------|------|------|
| `value`/`cacheNames` | `String[]` | 指定缓存名称，即缓存所在的命名空间 | `@Cacheable("users")` 或 `@Cacheable(value = "products")` |
| `key` | `String` | 缓存数据的键生成策略，默认使用方法参数组合生成 | `@Cacheable(value = "users", key = "#id")` 根据参数id生成key |
| `condition` | `String` | 条件表达式，满足条件时才缓存 | `@Cacheable(value = "users", condition = "#id > 0")` 只缓存id大于0的结果 |
| `unless` | `String` | 条件表达式，满足条件时不缓存 | `@Cacheable(value = "users", unless = "#result == null")` 结果为空时不缓存 |

```java
@Service
@CacheConfig(cacheNames = "role")//提取缓存的前缀配置
public class RoleServiceImpl implements IRoleService {
    @Autowired
    private RoleMapper roleMapper;

    @Override
    //@Cacheable(cacheNames = "role", key = "#id",condition = "#id>0",unless = "#result==null")
    @Cacheable(key = "#id",condition = "#id>0",unless = "#result==null")
    public Role findById(Integer id) {
        return roleMapper.selectByPrimaryKey(id);
    }
  
    @Cacheable(key ="#root.method.getName()")//直接引用mehtodname异常
    @Override
    public R findAllRole() {
        List<Role> roleList = roleMapper.findAll();
        return R.ok(roleList);
    }
}
```


### 🗑️ @CacheEvict 注解

用于从缓存中移除数据，通常在更新或删除操作后使用，以确保缓存数据与数据库保持一致

| 属性 | 类型 | 作用 | 实例 |
|------|------|------|------|
| `value`/`cacheNames` | `String[]` | 指定缓存名称，即要清除的缓存所在的命名空间 | `@CacheEvict("users")` 或 `@CacheEvict(value = "products")` |
| `key` | `String` | 指定要清除的缓存项的键，根据方法参数生成 | `@CacheEvict(value = "users", key = "#id")` 清除指定id的缓存 |
| `condition` | `String` | 条件表达式，满足条件时才执行清除操作 | `@CacheEvict(value = "users", condition = "#id > 0")` 只有id大于0时才清除缓存 |
| `allEntries` | `boolean` | 是否清除整个缓存分区的所有条目，默认false | `@CacheEvict(value = "users", allEntries = true)` 清除users缓存中的所有数据 |
| `beforeInvocation` | `boolean` | 是否在方法执行前清除缓存，默认false（方法执行后清除） | `@CacheEvict(value = "users", key = "#id", beforeInvocation = true)` 方法执行前清除缓存 |

```java
@Override
@CacheEvict(key = "#id")
public Integer delete(Integer id) {
    return roleMapper.deleteByPrimaryKey(id);
}
```


### ✏️ @CachePut 注解

用于将方法返回值缓存，通常在插入或更新操作后使用，以确保缓存数据与数据库保持一致

| 属性 | 类型 | 作用 | 示例 |
|------|------|------|------|
| `value`/`cacheNames` | `String[]` | 指定缓存名称，即要更新的缓存所在的命名空间 | `@CachePut("users")` 或 `@CachePut(value = "products")` |
| `key` | `String` | 指定要更新的缓存项的键，根据方法参数生成 | `@CachePut(value = "users", key = "#id")` 更新指定id的缓存 |
| `condition` | `String` | 条件表达式，满足条件时才执行更新操作 | `@CachePut(value = "users", condition = "#id > 0")` 只有id大于0时才更新缓存 | 
| `unless` | `String` | 条件表达式，满足条件时不更新缓存 | `@CachePut(value = "users", unless = "#result == null")` 结果为空时不更新缓存 |

```java
@Override
@CachePut(key = "#result.id")//更新或者添加缓存---》有则更新，无则添加
public Role update(Role role) {
  roleMapper.updateByPrimaryKey(role);
  return role;
}
```


### 🔄 @Caching 注解

用于组合多个缓存注解，可以同时指定多个缓存操作，如同时清除多个缓存项或同时更新多个缓存项

| 属性 | 类型 | 作用 | 示例 |
|------|------|------|------|
| `cacheable` | `Cacheable[]` | 组合多个 `@Cacheable` 注解，用于缓存方法返回值 | `@Caching(cacheable = {@Cacheable("users"), @Cacheable("products")})` |
| `evict` | `CacheEvict[]` | 组合多个 `@CacheEvict` 注解，用于清除缓存项 | `@Caching(evict = {@CacheEvict("users"), @CacheEvict("products")})` |
| `put` | `CachePut[]` | 组合多个 `@CachePut` 注解，用于更新缓存项 | `@Caching(put = {@CachePut("users"), @CachePut("products")})` |

```java
@Override
@Caching(
        cacheable = {
                @Cacheable(key = "#role.rolename")
           
        },
        put = {
                @CachePut(key = "#role.id"),
                @CachePut(key = "#role.rolecode")
        },
        evict = {
                @CacheEvict(key = "8")
        }
)
public R add(Role role) {
    try {
        roleMapper.insert(role);
    } catch (Exception e) {
        return R.error();
    }
    return R.ok(role.getId());
}
```


---

## 🎯 SpEL 上下文数据

Spring Cache 提供了丰富的 SpEL 上下文数据，可以在缓存注解的表达式中使用：

| SpEL上下文数据 | 描述 | 示例 |
|---------------|------|------|
| `#root.method` | 当前方法的反射对象 | `#root.method.name` 获取方法名 |
| `#root.target` | 当前被调用的对象实例 | `#root.target.getClass().getSimpleName()` 获取目标类名 |
| `#root.caches` | 当前执行上下文关联的缓存列表 | `#root.caches[0].name` 获取第一个缓存名称 |
| `#root.methodName` | 当前被执行的方法名称 | `#root.methodName` |
| `#root.args` | 当前方法的参数数组 | `#root.args[0]` 获取第一个参数 |
| `#参数名` | 直接访问方法参数名称 | `#id` 访问名为id的参数 |
| `#result` | 方法执行后的返回值（仅在`unless`和`@CachePut`中可用） | `#result != null` 判断返回值是否为空 |
| `#argument` | 当只有一个参数时，可直接使用此名称引用 | `#argument` |

---

## 🎯 最佳实践建议

### 📋 缓存策略选择

- **读多写少场景**: 使用 `@Cacheable` 配合适当过期时间
- **频繁更新场景**: 使用 `@CachePut` 和 `@CacheEvict` 保证数据一致性
- **复杂业务场景**: 使用 `@Caching` 组合多种缓存操作

### ⚠️ 注意事项

- 缓存键的设计要合理，避免冲突和不必要的缓存穿透
- 对于敏感数据更新，要及时清理相关缓存
- 缓存异常不应影响主业务流程，需做好异常处理
- 合理设置缓存过期时间，平衡性能和数据一致性

### 💡 性能优化建议

- 使用 `@CacheConfig` 提取公共缓存配置，减少重复代码
- 合理利用 `condition` 和 `unless` 控制缓存行为
- 对于大对象缓存要考虑序列化性能
- 定期监控缓存命中率和性能指标

---

## 🔍 核心原理解析

### 🏗️ 工作机制

Spring Cache 通过 AOP 代理实现缓存功能：

1. **代理创建**: `@EnableCaching` 启用缓存后，Spring 为带有缓存注解的 bean 创建代理
2. **拦截器链**: 缓存操作通过 `CacheInterceptor` 拦截器处理
3. **缓存管理**: `CacheManager` 负责管理多个 `Cache` 实例
4. **缓存操作**: 通过具体的缓存实现（如 Redis、Caffeine）执行缓存读写

---

## 🏭 生产环境使用场景实例

### 📊 用户信息服务缓存

```java
@Service
@CacheConfig(cacheNames = "user")
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    // 查询用户信息 - 高频读取
    @Cacheable(key = "#id", condition = "#id != null")
    @Override
    public User findById(Long id) {
        return userMapper.selectById(id);
    }
    
    // 更新用户信息 - 需要同步更新缓存
    @CachePut(key = "#user.id")
    @Override
    public User updateUser(User user) {
        userMapper.updateById(user);
        return user;
    }
    
    // 删除用户 - 需要清除缓存
    @CacheEvict(key = "#id")
    @Override
    public void deleteUser(Long id) {
        userMapper.deleteById(id);
    }
    
    // 批量操作 - 清除整个缓存区域
    @CacheEvict(allEntries = true)
    @Override
    public void batchUpdateUsers(List<User> users) {
        userMapper.batchUpdate(users);
    }
}
```


### 🛒 商品库存缓存管理

```java
@Service
@CacheConfig(cacheNames = "inventory")
public class InventoryServiceImpl implements InventoryService {
    
    @Autowired
    private InventoryMapper inventoryMapper;
    
    // 查询库存 - 需要设置较短过期时间
    @Cacheable(key = "#productId", unless = "#result == null")
    @Override
    public Integer getStock(Long productId) {
        return inventoryMapper.getStockByProductId(productId);
    }
    
    // 扣减库存 - 需要同步更新缓存并做条件判断
    @Caching(
        put = {@CachePut(key = "#productId")},
        evict = {@CacheEvict(key = "'summary_' + #productId")}
    )
    @Override
    public boolean decreaseStock(Long productId, Integer quantity) {
        int result = inventoryMapper.decreaseStock(productId, quantity);
        return result > 0;
    }
    
    // 库存预警 - 不缓存低库存商品
    @Cacheable(key = "'warning_' + #threshold", condition = "#threshold > 10")
    @Override
    public List<InventoryWarning> getLowStockWarnings(Integer threshold) {
        return inventoryMapper.getLowStockWarnings(threshold);
    }
}
```
