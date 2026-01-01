# 📚 MyBatis-Plus 笔记

## 🔍 MyBatis-Plus 简介

### 🎯 什么是 MyBatis-Plus

![image-20201008112252456.png](img/image-20201008112252456.png)

MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生：

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence）
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持代码生成器**：可快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码

### 💾 MyBatis-Plus 优势
- **减少重复代码**：大量减少 Wrapper ，各中繁琐操作
- **灵活的配置**：全局配置、属性注入等
- **内置分页插件**：基于物理分页，配置即用
- **内置性能分析插件**：可输出 sql 语句以及其执行时间，建议开发测试时启用
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断

---

## 🚀 SpringBoot 整合 MyBatis-Plus

### 📦 1. 添加 Maven 依赖
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>
```


### ⚙️ 2. 数据库连接配置
```yaml
spring:
  datasource:
    url: jdbc:mysql://192.168.88.128:3306/mp?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
    username: root
    password: 24364726
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      pool-name: HikariCPDatasource
      minimum-idle: 5
      idle-timeout: 180000
      maximum-pool-size: 10
      auto-commit: true
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1

# MyBatis-Plus 配置
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: auto  # 主键策略
```


### 🧱 3. 创建实体类
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("user") // 指定数据库表名
public class User {
    @TableId(type = IdType.AUTO) // 主键注解
    private Long id;
    
    private String name;
    private Integer age;
    private String email;
    
    @TableField(fill = FieldFill.INSERT) // 自动填充
    private Date createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
}
```


### 🎯 4. 创建 Mapper 接口 
```java
@Mapper
@Repository
public interface UserMapper extends BaseMapper<User> {
    // 继承 BaseMapper 后自动拥有常用的 CRUD 方法
}
```


### 📄 5. 创建 Service 层
```java
// Service 接口
public interface UserService extends IService<User> {
}

// Service 实现类
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```


---

## 🏷️ MyBatis-Plus 核心功能详解

### 🔍 通用 CRUD 操作

#### BaseMapper 接口常用方法
```java
public interface BaseMapper<T> {
    // 插入
    int insert(T entity);
    
    // 根据 ID 删除
    int deleteById(Serializable id);
    
    // 根据 entity 条件删除
    int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
    
    // 根据 ID 更新
    int updateById(@Param(Constants.ENTITY) T entity);
    
    // 根据 entity 和 wrapper 条件更新
    int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> wrapper);
    
    // 根据 ID 查询
    T selectById(Serializable id);
    
    // 查询所有
    List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
    
    // 根据 Wrapper 条件查询总数
    Long selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
}
```


#### IService 接口常用方法
```java
public interface IService<T> {
    // 批量插入
    boolean saveBatch(Collection<T> entityList);
    
    // 根据 ID 批量删除
    boolean removeBatchByIds(Collection<? extends Serializable> idList);
    
    // 根据 ID 批量更新
    boolean updateBatchById(Collection<T> entityList);
    
    // 查询所有
    List<T> list();
    
    // 分页查询
    IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);
}
```


### ➕ 条件构造器（Wrapper）

#### QueryWrapper 查询条件构造
```java
@Service
public class UserServiceImpl {
    @Autowired
    private UserMapper userMapper;
    
    public void queryExample() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        
        // 等于
        queryWrapper.eq("name", "张三");
        
        // 大于
        queryWrapper.gt("age", 18);
        
        // like
        queryWrapper.like("name", "张");
        
        // in
        queryWrapper.in("id", Arrays.asList(1, 2, 3));
        
        // 排序
        queryWrapper.orderByDesc("create_time");
        
        List<User> users = userMapper.selectList(queryWrapper);
    }
}
```


#### LambdaQueryWrapper 类型安全查询
```java
public void lambdaQueryExample() {
    LambdaQueryWrapper<User> lambdaQuery = new LambdaQueryWrapper<>();
    
    // 类型安全的查询条件
    lambdaQuery.eq(User::getName, "张三")
              .gt(User::getAge, 18)
              .like(User::getEmail, "@")
              .orderByDesc(User::getCreateTime);
              
    List<User> users = userMapper.selectList(lambdaQuery);
}
```


### ✏️ 更新构造器（UpdateWrapper）
```java
public void updateExample() {
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    
    // 设置更新字段
    updateWrapper.set("age", 25)
                 .set("email", "zhangsan@example.com")
                 .eq("name", "张三");
                 
    userMapper.update(null, updateWrapper);
}
```


---

## 🗺️ MyBatis-Plus 注解详解

### 实体类常用注解

#### @TableName 表名注解
```java
@TableName("sys_user") // 指定数据库表名
public class User {
    // ...
}
```


#### @TableId 主键注解
```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
```


**IdType 主键策略**：
- `AUTO`：数据库 ID 自增
- `NONE`：无状态，未设置主键类型
- `INPUT`：用户输入 ID
- `ASSIGN_ID`：分配 ID（使用雪花算法）
- `ASSIGN_UUID`：分配 UUID

#### @TableField 字段注解
```java
@TableField("user_name") // 指定数据库字段名
private String name;

@TableField(exist = false) // 该字段在数据库表中不存在
private String remark;

@TableField(fill = FieldFill.INSERT) // 自动填充策略
private Date createTime;
```


#### @TableLogic 逻辑删除
```java
@TableLogic
private Integer deleted; // 0-未删除，1-已删除
```


---

## ⚙️ MyBatis-Plus 插件配置

### 分页插件配置
```java
@Configuration
public class MybatisPlusConfig {
    @Configuration
    public class MybatisPlusConfig {
        @Bean
        public MybatisPlusInterceptor mybatisPlusInterceptor() {
            MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
            PaginationInnerInterceptor paginationInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
            // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
            // paginationInterceptor.setOverflow(false);
            // 设置最大单页限制数量，-1不受限制
            paginationInterceptor.setMaxLimit(-1L);
            interceptor.addInnerInterceptor(paginationInterceptor);
            return interceptor;
        }
    }
}
```


### 使用分页查询
```java
public void pageQuery() {
    Page<User> page = new Page<>(1, 10); // 当前页，每页条数
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.gt("age", 18);
    
    Page<User> result = userMapper.selectPage(page, wrapper);
    
    System.out.println("总记录数：" + result.getTotal());
    System.out.println("当前页数据：" + result.getRecords());
}
```


### 自动填充功能
```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", Date.class, new Date());
        this.strictInsertFill(metaObject, "updateTime", Date.class, new Date());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", Date.class, new Date());
    }
}
```


---

## 🎯 MyBatis-Plus 代码生成器

### 添加代码生成器依赖
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>

<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.30</version>
</dependency>
```


### 代码生成器配置
```java
public class CodeGenerator {
    public static void main(String[] args) {
        // 1. 配置数据源
        DataSourceConfig dataSourceConfig = new DataSourceConfig.Builder(
                "jdbc:mysql://localhost:3306/test",
                "root",
                "password"
        ).build();

        // 2. 全局配置
        GlobalConfig globalConfig = new GlobalConfig.Builder()
                .outputDir(System.getProperty("user.dir") + "/src/main/java")
                .author("your_name")
                .enableSwagger()
                .build();

        // 3. 包配置
        PackageConfig packageConfig = new PackageConfig.Builder()
                .parent("com.example")
                .entity("entity")
                .mapper("mapper")
                .service("service")
                .controller("controller")
                .build();

        // 4. 策略配置
        StrategyConfig strategyConfig = new StrategyConfig.Builder()
                .addInclude("user", "department") // 需要生成的表名
                .entityBuilder()
                .enableLombok()
                .logicDeleteColumnName("deleted")
                .build();

        // 5. 执行代码生成
        new AutoGenerator(dataSourceConfig)
                .global(globalConfig)
                .packageInfo(packageConfig)
                .strategy(strategyConfig)
                .execute();
    }
}
```


---

## 🔧 MyBatis-Plus 高级特性

### 乐观锁插件
```java
// 配置乐观锁插件
@Bean
public MybatisPlusInterceptor optimisticLockerInterceptor(){
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}

// 实体类中添加版本号字段
@Version
private Integer version;
```


### 多租户 SQL 解析器
```java
// 配置多租户插件
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    TenantLineInnerInterceptor tenantInterceptor = new TenantLineInnerInterceptor(
        new TenantLineHandler() {
            @Override
            public Expression getTenantId() {
                return new LongValue(1L); // 租户 ID
            }
            
            @Override
            public String getTenantIdColumn() {
                return "tenant_id";
            }
            
            @Override
            public boolean ignoreTable(String tableName) {
                return !"user".equalsIgnoreCase(tableName);
            }
        }
    );
    interceptor.addInnerInterceptor(tenantInterceptor);
    return interceptor;
}
```


### 动态表名 SQL 解析器
```java
@Bean
public DynamicTableNameInnerInterceptor dynamicTableNameInnerInterceptor() {
    DynamicTableNameInnerInterceptor interceptor = new DynamicTableNameInnerInterceptor();
    interceptor.setTableNameHandler((sql, tableName) -> {
        // 动态表名逻辑
        return tableName + "_2023";
    });
    return interceptor;
}
```


---

## 📝 MyBatis-Plus 最佳实践

### 实体类设计规范
```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("sys_user")
@ApiModel(value="User对象", description="用户表")
public class User extends Model<User> {
    
    @ApiModelProperty(value = "主键ID")
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    @ApiModelProperty(value = "姓名")
    private String name;

    @ApiModelProperty(value = "年龄")
    private Integer age;

    @ApiModelProperty(value = "邮箱")
    private String email;

    @ApiModelProperty(value = "创建时间")
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @ApiModelProperty(value = "更新时间")
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    @ApiModelProperty(value = "逻辑删除标识")
    @TableLogic
    private Integer deleted;

    @ApiModelProperty(value = "版本号")
    @Version // 乐观锁
    private Integer version;
}
```


### Service 层最佳实践
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    
    // 自定义业务方法
    public List<User> findUsersByCondition(String name, Integer minAge) {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        
        if (StringUtils.isNotBlank(name)) {
            wrapper.like(User::getName, name);
        }
        
        if (minAge != null) {
            wrapper.ge(User::getAge, minAge);
        }
        
        return this.list(wrapper);
    }
    
    // 分页查询
    public IPage<User> findUserPage(Page<User> page, String name) {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        
        if (StringUtils.isNotBlank(name)) {
            wrapper.like(User::getName, name);
        }
        
        return this.page(page, wrapper);
    }
}
```


### Controller 层示例
```java
@RestController
@RequestMapping("/user")
@Api(tags = "用户管理")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    @ApiOperation("根据ID查询用户")
    public User getUserById(@PathVariable Long id) {
        return userService.getById(id);
    }
    
    @PostMapping
    @ApiOperation("新增用户")
    public boolean saveUser(@RequestBody User user) {
        return userService.save(user);
    }
    
    @PutMapping
    @ApiOperation("更新用户")
    public boolean updateUser(@RequestBody User user) {
        return userService.updateById(user);
    }
    
    @DeleteMapping("/{id}")
    @ApiOperation("删除用户")
    public boolean deleteUser(@PathVariable Long id) {
        return userService.removeById(id);
    }
    
    @GetMapping("/page")
    @ApiOperation("分页查询用户")
    public IPage<User> getUserPage(
            @RequestParam(defaultValue = "1") Integer current,
            @RequestParam(defaultValue = "10") Integer size,
            @RequestParam(required = false) String name) {
        
        Page<User> page = new Page<>(current, size);
        return userService.findUserPage(page, name);
    }
}
```
