# SpringCache多级缓存

## 1.引入依赖

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
        
    <!-- Caffeine -->
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
        <version>3.1.6</version>
    </dependency>
    
    !-- Spring Cache -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    
    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
        <optional>true</optional>
    </dependency>
```

## 2.配置文件

```yml
multi-level-cache:
  caffeine:
    initial-capacity: 100   # 初始容量
    maximum-size: 1000      # 缓存最大容量
    expire-after-write: 5m  # 本地缓存5分钟过期
  redis:
    default-ttl: 30m        # Redis缓存30分钟过期
```

## 3.MultiLevelCache

```java
@Slf4j
public class MultiLevelCache implements Cache {

    private final String name;
    private final Cache caffeineCache;
    private final Cache redisCache;

    public MultiLevelCache(String name, Cache caffeineCache, Cache redisCache) {
        this.name = name;
        this.caffeineCache = caffeineCache;
        this.redisCache = redisCache;
    }

    @Override
    public String getName() {
        return this.name;
    }

    @Override
    public Object getNativeCache() {
        return this;
    }

    /**
     * 查询缓存：L1 -> L2
     */
    @Override
    public ValueWrapper get(Object key) {
        // 1. 先查询L1本地缓存
        ValueWrapper value = caffeineCache.get(key);
        if (value == null) {
            // 2. L1未命中，查询L2 Redis
            value = redisCache.get(key);
            if (value != null) {
                log.debug("Cache [{}] - L2 Redis HIT, key: {}, backfill to L1", name, key);
                // 回填到L1
                caffeineCache.put(key, value.get());
                return value;
            }

            log.debug("Cache [{}] - ALL MISS, key: {}", name, key);
            return null;
        } else {
            log.debug("Cache [{}] - L1 Caffeine HIT, key: {}", name, key);
            return value;
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T get(Object key, Class<T> type) {
        ValueWrapper wrapper = get(key);
        if (wrapper == null) {
            return null;
        }
        Object value = wrapper.get();
        if (value != null && type != null && !type.isInstance(value)) {
            throw new IllegalStateException(
                    "Cached value is not of required type [" + type.getName() + "]: " + value);
        }
        return (T) value;
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T get(Object key, Callable<T> valueLoader) {
        ValueWrapper wrapper = get(key);
        if (wrapper != null) {
            return (T) wrapper.get();
        }

        // 缓存未命中，调用valueLoader加载数据（通常是从DB）
        try {
            T value = valueLoader.call();
            if (value != null) {
                put(key, value);
            }
            return value;
        } catch (Exception e) {
            throw new ValueRetrievalException(key, valueLoader, e);
        }
    }

    /**
     * 写入缓存：同时写入L1和L2
     */
    @Override
    public void put(Object key, Object value) {
        log.debug("Cache [{}] - PUT key: {}", name, key);
        // 同时写入两级缓存
        caffeineCache.put(key, value);
        redisCache.put(key, value);
    }

    /**
     * 如果不存在则写入
     */
    @Override
    public ValueWrapper putIfAbsent(Object key, Object value) {
        ValueWrapper existingValue = get(key);
        if (existingValue == null) {
            put(key, value);
            return null;
        }
        return existingValue;
    }

    /**
     * 删除缓存：同时删除L1和L2
     */
    @Override
    public void evict(Object key) {
        log.debug("Cache [{}] - EVICT key: {}", name, key);
        // 先删除L2，再删除L1（防止其他节点回填）
        redisCache.evict(key);
        caffeineCache.evict(key);
    }

    @Override
    public boolean evictIfPresent(Object key) {
        evict(key);
        return true;
    }

    /**
     * 清空缓存
     */
    @Override
    public void clear() {
        log.debug("Cache [{}] - CLEAR all", name);
        redisCache.clear();
        caffeineCache.clear();
    }

    @Override
    public boolean invalidate() {
        clear();
        return true;
    }
}
```

## 4.MultiLevelCacheManager

```java
@Slf4j
public class MultiLevelCacheManager implements CacheManager {

    private final ConcurrentMap<String, Cache> cacheMap = new ConcurrentHashMap<>();
    private final RedisConnectionFactory redisConnectionFactory;
    private final MultiLevelCacheProperties properties;

    // Caffeine配置
    private final long caffeineMaximumSize;
    private final Duration caffeineExpireAfterWrite;

    // Redis配置
    private final Duration redisDefaultTtl;

    public MultiLevelCacheManager(RedisConnectionFactory redisConnectionFactory,
                                   MultiLevelCacheProperties properties) {
        this.redisConnectionFactory = redisConnectionFactory;
        this.properties = properties;
        this.caffeineMaximumSize = properties.getCaffeine().getMaximumSize();
        this.caffeineExpireAfterWrite = properties.getCaffeine().getExpireAfterWrite();
        this.redisDefaultTtl = properties.getRedis().getDefaultTtl();
    }

    @Override
    public Cache getCache(String name) {
        return cacheMap.computeIfAbsent(name, this::createMultiLevelCache);
    }

    @Override
    public Collection<String> getCacheNames() {
        return Collections.unmodifiableSet(cacheMap.keySet());
    }

    private Cache createMultiLevelCache(String name) {
        log.info("Creating multi-level cache: {}", name);

        // 创建L1 Caffeine缓存
        Cache caffeineCache = createCaffeineCache(name);

        // 创建L2 Redis缓存
        Cache redisCache = createRedisCache(name);

        return new MultiLevelCache(name, caffeineCache, redisCache);
    }

    private Cache createCaffeineCache(String name) {
        com.github.benmanes.caffeine.cache.Cache<Object, Object> cache = Caffeine.newBuilder()
                .initialCapacity(properties.getCaffeine().getInitialCapacity())
                .maximumSize(caffeineMaximumSize)
                .expireAfterWrite(caffeineExpireAfterWrite.toMillis(), TimeUnit.MILLISECONDS)
                .recordStats()  // 记录统计信息
                .build();
        return new CaffeineCache(name, cache);
    }

    private Cache createRedisCache(String name) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(redisDefaultTtl)
                .serializeValuesWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .prefixCacheNameWith("multi-level-cache:");

        RedisCacheWriter cacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);
        RedisCacheManager redisCacheManager = new RedisCacheManager(cacheWriter, config);

        return redisCacheManager.getCache(name);
    }
}
```

## 5.MultiLevelCacheProperties

```java
@Data
@Component
@ConfigurationProperties(prefix = "multi-level-cache")
public class MultiLevelCacheProperties {

    private CaffeineProperties caffeine = new CaffeineProperties();
    private RedisProperties redis = new RedisProperties();

    @Data
    public static class CaffeineProperties {
        private int initialCapacity = 100;
        private long maximumSize = 1000;
        private Duration expireAfterWrite = Duration.ofMinutes(5);
    }

    @Data
    public static class RedisProperties {
        private Duration defaultTtl = Duration.ofMinutes(30);
    }
}
```

## 6.CacheConfig

```java
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory,
                                     MultiLevelCacheProperties properties) {
        return new MultiLevelCacheManager(redisConnectionFactory, properties);
    }
}
```