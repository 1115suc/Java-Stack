# 🚀 单体项目快速集成 Caffeine Cache（纯手动方式）

## 🔧 项目集成步骤

### 📦 引入依赖

在 `pom.xml` 中添加以下依赖：

```xml
<!-- Caffeine 缓存核心依赖 -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.8</version>
</dependency>
```


### ⚙️ 创建缓存配置类

```java
@Configuration
public class CaffeineConfig {
    
    @Bean
    public Cache<String, Object> manualCache() {
        return Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats()
                .build();
    }
    
    @Bean
    public LoadingCache<String, Object> loadingCache() {
        return Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats()
                .build(key -> loadFromDatabase(key));
    }
    
    private Object loadFromDatabase(String key) {
        // 实现数据加载逻辑
        return null;
    }
}
```


## 📌 手动操作缓存

### 基本缓存操作

```java
@Service
public class ManualCacheService {
    
    @Autowired
    private Cache<String, Object> manualCache;
    
    // 获取缓存
    public Object getValue(String key) {
        return manualCache.getIfPresent(key);
    }
    
    // 设置缓存
    public void putValue(String key, Object value) {
        manualCache.put(key, value);
    }
    
    // 删除缓存
    public void removeValue(String key) {
        manualCache.invalidate(key);
    }
    
    // 批量删除缓存
    public void removeValues(List<String> keys) {
        manualCache.invalidateAll(keys);
    }
    
    // 清空所有缓存
    public void clearAll() {
        manualCache.invalidateAll();
    }
}
```


### 使用 LoadingCache 自动加载

```java
@Service
public class LoadingCacheService {
    
    @Autowired
    private LoadingCache<String, Object> loadingCache;
    
    // 获取缓存（不存在时自动加载）
    public Object getValue(String key) {
        return loadingCache.get(key);
    }
    
    // 批量获取
    public Map<String, Object> getAllValues(Iterable<String> keys) {
        return loadingCache.getAll(keys);
    }
    
    // 刷新缓存
    public void refreshValue(String key) {
        loadingCache.refresh(key);
    }
}
```


## 💡 简单使用实例

### 用户信息服务实例

```java
@Service
public class UserService {
    
    @Autowired
    private Cache<String, User> userCache;
    
    @Autowired
    private UserRepository userRepository;
    
    /**
     * 根据用户ID获取用户信息（带缓存）
     */
    public User getUserById(String userId) {
        // 先从缓存中获取
        User cachedUser = userCache.getIfPresent(userId);
        if (cachedUser != null) {
            System.out.println("从缓存中获取用户: " + userId);
            return cachedUser;
        }
        
        // 缓存未命中，从数据库查询
        User dbUser = userRepository.findById(userId);
        if (dbUser != null) {
            // 放入缓存
            userCache.put(userId, dbUser);
            System.out.println("从数据库获取用户并缓存: " + userId);
        }
        
        return dbUser;
    }
    
    /**
     * 更新用户信息
     */
    public void updateUser(User user) {
        // 更新数据库
        userRepository.update(user);
        // 同步更新缓存
        userCache.put(user.getId(), user);
        System.out.println("用户信息已更新并同步到缓存: " + user.getId());
    }
    
    /**
     * 删除用户
     */
    public void deleteUser(String userId) {
        // 删除数据库记录
        userRepository.deleteById(userId);
        // 删除缓存
        userCache.invalidate(userId);
        System.out.println("用户已删除，缓存已清理: " + userId);
    }
}
```


### 商品价格缓存实例

```java
@Service
public class ProductService {

    @Autowired
    private LoadingCache<String, BigDecimal> priceCache;

    @Autowired
    private ProductRepository productRepository;

    /**
     * 获取商品价格（自动加载+降级处理）
     */
    public BigDecimal getProductPrice(String productId) {
        try {
            return priceCache.get(productId);
        } catch (Exception e) {
            // 缓存加载失败时的降级处理
            System.err.println("缓存获取失败，尝试从数据库直接获取: " + productId);
            BigDecimal price = loadPriceFromDatabase(productId);
            if (price != null) {
                // 更新缓存
                priceCache.put(productId, price);
            }
            return price;
        }
    }

    /**
     * 批量获取商品价格
     */
    public Map<String, BigDecimal> getBatchProductPrices(List<String> productIds) {
        try {
            return priceCache.getAll(productIds);
        } catch (Exception e) {
            System.err.println("批量获取商品价格失败，逐个尝试获取");
            Map<String, BigDecimal> result = new HashMap<>();
            for (String productId : productIds) {
                BigDecimal price = getProductPrice(productId);
                if (price != null) {
                    result.put(productId, price);
                }
            }
            return result;
        }
    }

    /**
     * 更新商品价格（保证数据一致性）
     */
    @Transactional
    public void updateProductPrice(String productId, BigDecimal newPrice) {
        // 1. 先更新数据库
        productRepository.updatePriceById(productId, newPrice);

        // 2. 同步更新缓存
        priceCache.put(productId, newPrice);

        System.out.println("商品价格已更新，数据库和缓存同步: " + productId);
    }

    /**
     * 删除商品价格缓存
     */
    public void evictProductPrice(String productId) {
        priceCache.invalidate(productId);
        System.out.println("商品价格缓存已清除: " + productId);
    }

    /**
     * 实际的价格加载逻辑
     */
    private BigDecimal loadPriceFromDatabase(String productId) {
        System.out.println("从数据库加载商品价格: " + productId);
        return productRepository.findPriceById(productId);
    }
}
```


## ⚙️ Caffeine 核心配置选项

### 容量控制
```java
.maximumSize(1000)              // 最大缓存条目数量
.maximumWeight(10000)           // 最大权重限制
.weigher((key, value) -> value.toString().length())  // 权重计算策略
```


### 过期策略
```java
.expireAfterWrite(10, TimeUnit.MINUTES)   // 写入后多久过期
.expireAfterAccess(5, TimeUnit.MINUTES)   // 最后访问后多久过期
.expireAfter(new Expiry<K, V>() {         // 自定义过期策略
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


### 引用类型
```java
.weakKeys()     // 键使用弱引用
.weakValues()   // 值使用弱引用
.softValues()   // 值使用软引用（内存不足时回收）
```

## 🔐 数据一致性保障机制

### 1. Cache-Aside 模式（推荐用于手动缓存）

```java
@Service
public class ProductManualCacheService {
    
    @Autowired
    private Cache<String, BigDecimal> manualCache;
    
    @Autowired
    private ProductRepository productRepository;
    
    /**
     * 读取商品价格 - Cache-Aside模式
     */
    public BigDecimal getProductPrice(String productId) {
        // 1. 先查缓存
        BigDecimal cachedPrice = manualCache.getIfPresent(productId);
        if (cachedPrice != null) {
            return cachedPrice;
        }
        
        // 2. 缓存未命中，查数据库
        BigDecimal dbPrice = productRepository.findPriceById(productId);
        if (dbPrice != null) {
            // 3. 回填缓存
            manualCache.put(productId, dbPrice);
        }
        
        return dbPrice;
    }
    
    /**
     * 更新商品价格 - 保证一致性
     */
    @Transactional
    public void updateProductPrice(String productId, BigDecimal newPrice) {
        // 1. 先更新数据库
        productRepository.updatePriceById(productId, newPrice);
        
        // 2. 再更新缓存（或删除缓存让下次重新加载）
        manualCache.put(productId, newPrice);
        // 或者使用删除策略：manualCache.invalidate(productId);
    }
    
    /**
     * 删除商品 - 保证一致性
     */
    @Transactional
    public void deleteProduct(String productId) {
        // 1. 先删除数据库记录
        productRepository.deleteById(productId);
        
        // 2. 再删除缓存
        manualCache.invalidate(productId);
    }
}
```


### 2. Write-Through 模式（通过CacheLoader实现）

```java
@Service
public class ProductWriteThroughService {
    
    // 使用特殊标记表示空值，防止缓存穿透
    private static final BigDecimal EMPTY_PRICE = BigDecimal.valueOf(-1);
    
    private final LoadingCache<String, BigDecimal> writeThroughCache;
    
    public ProductWriteThroughService(ProductRepository productRepository) {
        this.writeThroughCache = Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats()
                .build(key -> {
                    BigDecimal price = productRepository.findPriceById(key);
                    // 返回特殊值而不是null，防止缓存穿透
                    return price != null ? price : EMPTY_PRICE;
                });
    }
    
    /**
     * 获取商品价格
     */
    public BigDecimal getProductPrice(String productId) {
        BigDecimal price = writeThroughCache.get(productId);
        // 如果是空值标记，返回null给调用方
        return price == EMPTY_PRICE ? null : price;
    }
    
    /**
     * 更新商品价格（写透模式）
     */
    @Transactional
    public void updateProductPrice(String productId, BigDecimal newPrice) {
        // 数据库更新由CacheLoader完成（延迟加载时）
        // 直接更新缓存，下次加载时会触发数据库更新
        writeThroughCache.put(productId, newPrice);
    }
}
```

## 🛡️ 缓存一致性最佳实践

### 1. 双写一致性策略
- **先更新数据库，再更新缓存**
- 确保在同一个事务中操作，避免中间状态

### 2. 缓存失效策略
- **先更新数据库，再删除缓存**
- 下次读取时重新加载最新数据

### 3. 异常处理机制
- 缓存获取失败时降级到数据库查询
- 查询到的数据回填到缓存中
- 对空值进行缓存，防止缓存穿透

### 4. 并发控制
```java
// 使用分布式锁防止并发更新问题
public BigDecimal getProductPriceWithLock(String productId) {
    String lockKey = "price_lock:" + productId;
    // 获取分布式锁
    if (distributedLock.tryLock(lockKey, 3000)) {
        try {
            return getProductPrice(productId);
        } finally {
            distributedLock.releaseLock(lockKey);
        }
    }
    // 获取锁失败，直接查询数据库
    return productRepository.findPriceById(productId);
}
```


通过以上机制，可以有效保证缓存与数据库之间的数据一致性，同时具备良好的容错能力和性能表现。

## 🎯 最佳实践建议

- **读多写少场景**：使用 `Cache` 手动管理或 `LoadingCache` 自动加载
- **复杂数据加载**：实现 `CacheLoader` 来处理缓存未命中情况
- **监控缓存效果**：启用 `recordStats()` 并定期查看统计信息
- **防止缓存穿透**：对空结果也进行缓存，设置较短过期时间
- **防止缓存雪崩**：给不同缓存设置随机过期时间
- **热点数据保护**：关键数据可以考虑永不过期或者加锁更新
