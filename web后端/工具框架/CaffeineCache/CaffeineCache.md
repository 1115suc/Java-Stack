# 🚀 CaffeineCache 本地缓存工具

## 🔧 项目集成 Caffeine

### 📦 引入依赖

```xml
<!-- Caffeine 缓存核心依赖 -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.8</version>
</dependency>

<!-- Spring Boot Cache Starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```


### ⚙️ 基础配置

#### 启用缓存支持

```java
@SpringBootApplication
@EnableCaching  // 启用 Spring Cache
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```


#### 配置 Caffeine 缓存管理器

```java
@Configuration
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                .maximumSize(1000)                    // 最大缓存条目数
                .expireAfterWrite(10, TimeUnit.MINUTES) // 写入后10分钟过期
                .expireAfterAccess(5, TimeUnit.MINUTES) // 访问后5分钟过期
                .recordStats()                        // 启用统计信息
        );
        return cacheManager;
    }
}
```


---

## 🏷️ 注解使用指南

### 📌 常用缓存注解

| 注解 | 使用位置 | 功能 |
|------|----------|------|
| `@Cacheable` | 方法上 | 触发缓存填充 |
| `@CacheEvict` | 方法上 | 触发缓存清除 |
| `@CachePut` | 方法上 | 更新缓存而不干扰方法执行 |
| `@Caching` | 方法上 | 组合多个缓存操作 |
| `@CacheConfig` | 类上 | 共享缓存配置 |

### 📝 基本使用示例

```java
@Service
@CacheConfig(cacheNames = "users")  // 共享缓存名称
public class UserService {
    
    @Cacheable(key = "#id")
    public User getUserById(Long id) {
        // 模拟数据库查询
        return userRepository.findById(id);
    }
    
    @CachePut(key = "#user.id")
    public User updateUser(User user) {
        return userRepository.save(user);
    }
    
    @CacheEvict(key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
    
    @CacheEvict(allEntries = true)
    public void clearAllUsers() {
        // 清除所有用户缓存
    }
}
```


---

## 📖 Caffeine 配置详解

### ⚙️ 核心配置选项

#### 容量限制
```java
Caffeine.newBuilder()
    .maximumSize(1000)           // 基于条目数限制
    .maximumWeight(10000)        // 基于权重限制
    .weigher((key, value) -> value.toString().length())  // 权重计算策略
```


#### 过期策略
```java
Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)    // 写入后固定时间过期
    .expireAfterAccess(5, TimeUnit.MINUTES)    // 最后访问后过期
    .expireAfter(new Expiry<K, V>() {          // 自定义过期策略
        @Override
        public long expireAfterCreate(K key, V value, long currentTime) {
            return TimeUnit.MINUTES.toNanos(10);
        }
        
        @Override
        public long expireAfterUpdate(K key, V value, long currentTime, long currentDuration) {
            return currentDuration;
        }
        
        @Override
        public long expireAfterRead(K key, V value, long currentTime, long currentDuration) {
            return TimeUnit.MINUTES.toNanos(5);
        }
    })
```


#### 弱引用与软引用
```java
Caffeine.newBuilder()
    .weakKeys()      // 弱引用键
    .weakValues()    // 弱引用值
    .softValues()    // 软引用值（内存不足时回收）
```


---

## 🎯 高级特性

### 📊 缓存统计信息

```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager cacheManager = new CaffeineCacheManager();
    cacheManager.setCaffeine(Caffeine.newBuilder()
            .recordStats()  // 启用统计
            .maximumSize(1000)
    );
    return cacheManager;
}

// 获取统计信息
public void printCacheStats() {
    Cache<Object, Object> cache = cacheManager.getCache("users").getNativeCache();
    CacheStats stats = cache.stats();
    System.out.println("命中率: " + stats.hitRate());
    System.out.println("加载次数: " + stats.loadCount());
    System.out.println("总请求数: " + stats.requestCount());
}
```


### 🔧 手动缓存操作

```java
@Service
public class ManualCacheService {
    
    private final Cache<String, Object> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
    
    public Object getValue(String key) {
        return cache.getIfPresent(key);
    }
    
    public Object putValue(String key, Object value) {
        cache.put(key, value);
        return value;
    }
    
    public void removeValue(String key) {
        cache.invalidate(key);
    }
    
    public Object computeIfAbsent(String key, Function<String, Object> loader) {
        return cache.get(key, loader);
    }
}
```


### 🔄 异步缓存支持

```java
@Service
public class AsyncCacheService {
    
    private final AsyncCache<String, Object> asyncCache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .buildAsync();
    
    public CompletableFuture<Object> getValueAsync(String key) {
        return asyncCache.get(key, this::loadFromDatabase);
    }
    
    private Object loadFromDatabase(String key) {
        // 模拟异步数据加载
        return database.findById(key);
    }
}
```


---

## 🎯 最佳实践建议

### 📋 缓存策略选择

- **读多写少场景**: 使用 `@Cacheable` 配合 `expireAfterWrite`
- **频繁更新场景**: 使用 `@CachePut` 保证数据一致性
- **批量操作场景**: 使用 `@Caching` 组合多个缓存操作

### ⚠️ 注意事项

- 缓存键的生成策略要合理，避免冲突
- 大对象缓存要考虑内存占用
- 缓存穿透：可缓存空值或使用布隆过滤器
- 缓存雪崩：设置不同的过期时间
- 缓存击穿：热点数据永不过期或加互斥锁

> 💡 提示
> - 使用 `CacheLoader` 可以实现自动加载缺失的缓存项
> - 结合 Spring Actuator 可以监控缓存性能指标
> - 生产环境中建议开启缓存统计以便调优