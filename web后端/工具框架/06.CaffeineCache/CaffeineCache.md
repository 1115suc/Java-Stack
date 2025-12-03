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

## 🎯 核心使用方式

### 🔧 手动缓存操作

```java
public class ManualCacheService {
    
    private final Cache<String, Object> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .recordStats()  // 启用统计
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
    
    public void clearCache() {
        cache.invalidateAll();
    }
}
```

### 📊 缓存统计信息

```java
public class CacheStatsService {
    private final Cache<String, Object> cache = Caffeine.newBuilder()
            .maximumSize(1000)
            .recordStats()  // 启用统计
            .build();

    public void printCacheStats() {
        CacheStats stats = cache.stats();
        System.out.println("命中率: " + stats.hitRate());
        System.out.println("加载次数: " + stats.loadCount());
        System.out.println("总请求数: " + stats.requestCount());
        System.out.println("平均加载时间: " + stats.averageLoadPenalty());
        System.out.println("缓存条目数: " + cache.estimatedSize());
    }
}
```

### 🔄 异步缓存支持

```java
public class AsyncCacheService {
    
    private final AsyncCache<String, Object> asyncCache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .buildAsync();
    
    public CompletableFuture<Object> getValueAsync(String key) {
        return asyncCache.get(key, this::loadFromDatabase);
    }
    
    private Object loadFromDatabase(String key) {
        // 模拟数据加载
        return database.findById(key);
    }
}
```

### 🚀 CacheLoader 自动加载

```java
public class CacheLoaderService {
    
    private final LoadingCache<String, Object> loadingCache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build(new CacheLoader<String, Object>() {
                @Override
                public Object load(String key) throws Exception {
                    return loadFromDatabase(key);
                }
            });
    
    public Object getValue(String key) {
        return loadingCache.get(key);  // 不存在时自动加载
    }
    
    public Map<String, Object> getValues(List<String> keys) {
        return loadingCache.getAll(keys);  // 批量加载
    }
    
    private Object loadFromDatabase(String key) {
        return database.findById(key);
    }
}
```

---

## 🎯 高级特性

### 🔄 刷新机制

```java
public class RefreshCacheService {
    
    private final LoadingCache<String, Object> refreshCache = Caffeine.newBuilder()
            .maximumSize(100)
            .refreshAfterWrite(5, TimeUnit.MINUTES)  // 5分钟后刷新
            .build(new CacheLoader<String, Object>() {
                @Override
                public Object load(String key) throws Exception {
                    return loadFromDatabase(key);
                }
                
                @Override
                public CompletableFuture<Object> reload(String key, Object oldValue) throws Exception {
                    return CompletableFuture.supplyAsync(() -> loadFromDatabase(key));
                }
            });
}
```

### 🎯 监听器

```java
public class CacheListenerService {
    
    private final Cache<String, Object> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .removalListener((String key, Object value, RemovalCause cause) -> {
                System.out.printf("Key %s was removed (%s)%n", key, cause);
            })
            .build();
}
```

---

## 🎯 最佳实践建议

### 📋 缓存策略选择

- **读多写少场景**: 使用 `expireAfterWrite` 配合适当过期时间
- **频繁更新场景**: 使用 `refreshAfterWrite` 实现异步刷新
- **批量操作场景**: 使用 `getAll` 方法批量加载数据

### ⚠️ 注意事项

- 缓存键的生成策略要合理，避免冲突
- 大对象缓存要考虑内存占用
- 缓存穿透：可缓存空值或使用布隆过滤器
- 缓存雪崩：设置不同的过期时间
- 缓存击穿：热点数据使用异步刷新机制

### 💡 性能优化建议

- 合理设置缓存大小，避免内存溢出
- 启用统计功能监控缓存性能
- 使用合适的过期策略平衡数据一致性和性能
- 考虑使用弱引用或软引用应对内存压力

---

## 🔍 核心原理解析

### 🏗️ 数据结构

Caffeine 基于以下核心数据结构实现：

- **ConcurrentHashMap**: 线程安全的哈希表存储缓存条目
- **W-TinyLFU**: 改进的缓存淘汰算法，提供更好的命中率
- **RingBuffer**: 高效的并发队列用于事件处理

### ⚡ 并发机制

- **StripedBuffer**: 减少并发冲突的缓冲机制
- **CAS操作**: 大量使用原子操作保证线程安全
- **分段锁**: 降低锁竞争，提高并发性能

### 🎯 淘汰算法

Caffeine 使用 Window TinyLFU (W-TinyLFU) 算法：
- **Window Cache**: 新进入的缓存项
- **Probation**: 试用期，观察访问频率
- **Protected**: 保护期，确保热点数据不被淘汰
- **TinyLFU**: 使用频率统计进行淘汰决策

---

## 🏭 生产环境使用场景实例

### 📊 商品信息缓存系统

```java
@Service
public class ProductCacheService {
    
    // 商品基础信息缓存 - 读多写少，设置较长过期时间
    private final LoadingCache<Long, Product> productCache = Caffeine.newBuilder()
            .maximumSize(10000)  // 最多缓存1万个商品
            .expireAfterWrite(30, TimeUnit.MINUTES)  // 30分钟过期
            .refreshAfterWrite(20, TimeUnit.MINUTES)  // 20分钟开始刷新
            .recordStats()  // 启用统计
            .build(new CacheLoader<Long, Product>() {
                @Override
                public Product load(Long productId) {
                    return productRepository.findById(productId)
                            .orElseThrow(() -> new ProductNotFoundException(productId));
                }
                
                @Override
                public CompletableFuture<Product> reload(Long key, Product oldValue) {
                    return CompletableFuture.supplyAsync(() -> load(key), executor);
                }
            });
    
    // 商品库存缓存 - 频繁变更，设置较短过期时间
    private final Cache<Long, Integer> stockCache = Caffeine.newBuilder()
            .maximumSize(5000)
            .expireAfterWrite(5, TimeUnit.MINUTES)  // 5分钟过期
            .recordStats()
            .build();
    
    public Product getProduct(Long productId) {
        return productCache.get(productId);
    }
    
    public Integer getStock(Long productId) {
        return stockCache.get(productId, id -> inventoryService.getAvailableStock(id));
    }
    
    public void updateStock(Long productId, Integer newStock) {
        stockCache.put(productId, newStock);
    }
    
    public void evictProduct(Long productId) {
        productCache.invalidate(productId);
        stockCache.invalidate(productId);
    }
}
```

### 🛡️ 用户权限缓存系统

```java
@Component
public class UserPermissionCache {
    
    // 用户权限缓存 - 高并发读取，变更频率中等
    private final LoadingCache<Long, Set<String>> userPermissions = Caffeine.newBuilder()
            .maximumSize(50000)  // 支持5万用户权限缓存
            .expireAfterWrite(1, TimeUnit.HOURS)  // 1小时过期
            .refreshAfterWrite(45, TimeUnit.MINUTES)  // 45分钟刷新
            .recordStats()
            .removalListener((Long userId, Set<String> permissions, RemovalCause cause) -> {
                log.info("用户 {} 权限缓存被移除，原因: {}", userId, cause);
            })
            .build(new CacheLoader<Long, Set<String>>() {
                @Override
                public Set<String> load(Long userId) {
                    return permissionService.getUserPermissions(userId);
                }
            });
    
    // 角色权限缓存 - 变更较少，设置较长过期时间
    private final LoadingCache<Long, Set<String>> rolePermissions = Caffeine.newBuilder()
            .maximumSize(1000)  // 缓存1000个角色
            .expireAfterWrite(6, TimeUnit.HOURS)  // 6小时过期
            .recordStats()
            .build(roleId -> permissionService.getRolePermissions(roleId));
    
    public boolean hasPermission(Long userId, String permission) {
        Set<String> permissions = userPermissions.get(userId);
        return permissions != null && permissions.contains(permission);
    }
    
    public Set<String> getUserPermissions(Long userId) {
        return userPermissions.get(userId);
    }
    
    public void refreshUserPermission(Long userId) {
        userPermissions.refresh(userId);
    }
    
    public void evictUserPermission(Long userId) {
        userPermissions.invalidate(userId);
    }
}
```

### 📈 数据统计缓存

```java
@Service
@Slf4j
public class StatisticsCacheService {
    
    // 实时统计数据缓存 - 高并发，允许短暂不一致
    private final Cache<String, BigDecimal> statisticsCache = Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)  // 10分钟过期
            .recordStats()
            .build();
    
    // 异步加载缓存 - 处理复杂统计计算
    private final AsyncCache<String, ReportData> reportCache = Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .buildAsync();
    
    public BigDecimal getDailySales(String date) {
        return statisticsCache.get("daily_sales_" + date, key -> {
            log.info("计算 {} 的日销售额", date);
            return orderService.calculateDailySales(date);
        });
    }
    
    public CompletableFuture<ReportData> getMonthlyReport(String month) {
        return reportCache.get("monthly_report_" + month, 
            key -> CompletableFuture.supplyAsync(() -> {
                log.info("生成 {} 的月度报告", month);
                return reportService.generateMonthlyReport(month);
            }, reportExecutor));
    }
    
    public void refreshHotStatistics() {
        // 刷新热点统计数据
        List<String> hotKeys = Arrays.asList("daily_sales", "weekly_sales", "monthly_sales");
        hotKeys.forEach(key -> {
            try {
                statisticsCache.refresh(key);
            } catch (Exception e) {
                log.error("刷新统计缓存失败: {}", key, e);
            }
        });
    }
}
```

### 🌐 API 响应缓存

```java
@RestController
@RequestMapping("/api/v1")
public class ApiCacheController {
    
    // API 响应缓存 - 减少重复计算和数据库查询
    private final Cache<String, ApiResponse> apiCache = Caffeine.newBuilder()
            .maximumSize(10000)
            .expireAfterWrite(15, TimeUnit.MINUTES)
            .recordStats()
            .build();
    
    @GetMapping("/hot-products")
    public ApiResponse<List<Product>> getHotProducts(
            @RequestParam(defaultValue = "20") int limit) {
        
        String cacheKey = "hot_products_" + limit;
        
        return apiCache.get(cacheKey, key -> {
            log.info("查询热门商品，限制: {}", limit);
            
            List<Product> products = productService.getHotProducts(limit);
            
            return ApiResponse.success(products);
        });
    }
    
    @GetMapping("/category-tree")
    public ApiResponse<CategoryTree> getCategoryTree() {
        return apiCache.get("category_tree", key -> {
            log.info("构建商品分类树");
            CategoryTree tree = categoryService.buildCategoryTree();
            return ApiResponse.success(tree);
        });
    }
    
    @PostMapping("/clear-cache")
    public ApiResponse<String> clearCache(@RequestParam String key) {
        if ("all".equals(key)) {
            apiCache.invalidateAll();
            return ApiResponse.success("所有缓存已清除");
        } else {
            apiCache.invalidate(key);
            return ApiResponse.success("缓存 " + key + " 已清除");
        }
    }
}
```

### ⚡ 分布式缓存本地备份

```java
@Component
public class DistributedCacheBackup {
    
    // 本地缓存作为 Redis 的备份，减少网络开销
    private final Cache<String, Object> localCache = Caffeine.newBuilder()
            .maximumSize(5000)
            .expireAfterWrite(5, TimeUnit.MINUTES)  // 比 Redis 过期时间短
            .recordStats()
            .build();
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public <T> T get(String key, Class<T> clazz) {
        // 先查本地缓存
        Object localValue = localCache.getIfPresent(key);
        if (localValue != null) {
            return clazz.cast(localValue);
        }
        
        // 本地缓存没有，查 Redis
        Object redisValue = redisTemplate.opsForValue().get(key);
        if (redisValue != null) {
            // 写入本地缓存
            localCache.put(key, redisValue);
            return clazz.cast(redisValue);
        }
        
        return null;
    }
    
    public void set(String key, Object value, long timeout, TimeUnit unit) {
        // 同时写入 Redis 和本地缓存
        redisTemplate.opsForValue().set(key, value, timeout, unit);
        localCache.put(key, value);
    }
    
    public void evict(String key) {
        // 同时清除 Redis 和本地缓存
        redisTemplate.delete(key);
        localCache.invalidate(key);
    }
}
```

### 📊 生产环境监控配置

```java
@Configuration
@Slf4j
public class CacheMonitorConfig {
    
    @Scheduled(fixedDelay = 300000) // 每5分钟执行一次
    public void monitorCacheStats() {
        // 监控所有缓存的统计信息
        Map<String, Cache> caches = getAllCaches();
        
        caches.forEach((name, cache) -> {
            if (cache instanceof com.github.benmanes.caffeine.cache.Cache) {
                com.github.benmanes.caffeine.cache.Cache caffeineCache = 
                    (com.github.benmanes.caffeine.cache.Cache) cache;
                
                CacheStats stats = caffeineCache.stats();
                
                log.info("缓存监控 - {}: 命中率={}%, 请求数={}, 加载数={}, 平均加载时间={}ms, 缓存大小={}",
                        name,
                        String.format("%.2f", stats.hitRate() * 100),
                        stats.requestCount(),
                        stats.loadCount(),
                        String.format("%.2f", stats.averageLoadPenalty() / 1_000_000),
                        caffeineCache.estimatedSize());
                
                // 命中率过低告警
                if (stats.hitRate() < 0.8 && stats.requestCount() > 1000) {
                    log.warn("缓存 {} 命中率过低: {}%", name, String.format("%.2f", stats.hitRate() * 100));
                    // 发送告警通知
                    alertService.sendCacheAlert(name, stats.hitRate());
                }
            }
        });
    }
}
```

### ⚠️ 生产环境注意事项

1. **内存监控**：设置合理的 `maximumSize`，监控 JVM 内存使用
2. **过期策略**：根据业务特点选择合适的过期时间
3. **缓存穿透**：对空值也进行缓存，避免频繁查询数据库
4. **缓存雪崩**：为不同缓存设置不同的过期时间，避免同时失效
5. **性能监控**：开启 `recordStats()`，定期分析缓存命中率
6. **异常处理**：缓存操作要有异常处理，避免影响主业务流程
7. **分布式环境**：考虑使用 Redis + 本地缓存的双层架构
8. **数据一致性**：缓存更新要保证最终一致性