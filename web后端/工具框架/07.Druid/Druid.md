# 🛡️ Druid 数据库连接池

## 🔍 Druid 简介
Druid 可以很好地监控 DB 池连接和 SQL 的执行情况，天生就是针对监控而生的 DB 连接池。

---

## 📊 Druid 数据源配置参数详解

| 参数名 | 默认值 | 说明 |
|--------|--------|------|
| `name` | 无 | 数据源名称，用于区分多个数据源，格式："DataSource-" + System.identityHashCode(this) |
| `url` | 无 | 连接数据库的URL，不同数据库格式不同（如MySQL: jdbc:mysql://host:port/database） |
| `username` | 无 | 连接数据库的用户名 |
| `password` | 无 | 连接数据库的密码，建议使用ConfigFilter避免直接写在配置文件中 |
| `driverClassName` | 自动识别 | 根据`url`自动识别数据库类型并选择相应的驱动类 |
| `initialSize` | 0 | 初始化时建立物理连接的个数，在调用`init`方法或第一次`getConnection`时发生 |
| `maxActive` | 8 | 最大连接池数量 |
| `maxIdle` | 8 | 已废弃，配置无效 |
| `minIdle` | 无 | 最小连接池数量 |
| `maxWait` | 无 | 获取连接时最大等待时间（毫秒），配置后默认启用公平锁 |
| `poolPreparedStatements` | false | 是否缓存PreparedStatement（PSCache），MySQL建议关闭，Oracle建议开启 |
| `maxOpenPreparedStatements` | -1 | 启用PSCache时的最大缓存数量，配置>0时自动将`poolPreparedStatements`设为true |
| `validationQuery` | 无 | 检测连接有效性的SQL查询语句，要求是查询语句 |
| `validationQueryTimeout` | 无 | 检测连接有效性的超时时间（秒） |
| `testOnBorrow` | true | 申请连接时执行`validationQuery`检测连接有效性，会影响性能 |
| `testOnReturn` | false | 归还连接时执行`validationQuery`检测连接有效性，会影响性能 |
| `testWhileIdle` | false | 空闲时检测连接有效性，建议配置为true保证安全性且不影响性能 |
| `timeBetweenEvictionRunsMillis` | 1分钟(1.0.14) | Destroy线程检测连接间隔时间和`testWhileIdle`判断依据 |
| `numTestsPerEvictionRun` | 无 | 已废弃，一个`DruidDataSource`只支持一个EvictionRun |
| `minEvictableIdleTimeMillis` | 30分钟(1.0.14) | 连接保持空闲而不被驱逐的最长时间 |
| `connectionInitSqls` | 无 | 物理连接初始化时执行的SQL语句 |
| `exceptionSorter` | 自动识别 | 根据`dbType`自动识别，处理数据库不可恢复异常并抛弃连接 |
| `filters` | 无 | 通过别名配置扩展插件，常用：stat（监控统计）、log4j（日志）、wall（防SQL注入） |
| `proxyFilters` | 无 | `Filter`列表，与`filters`组合使用而非替换关系 |

---

## 🚀 SpringBoot 集成 Druid

### 📦 1. 添加 Maven 依赖
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.21</version>
</dependency>
```


### ⚙️ 2. 切换数据源配置
Spring Boot 2.0 以上默认使用 `com.zaxxer.hikari.HikariDataSource` 数据源，但可以通过 `spring.datasource.type` 指定数据源：

```yaml
spring:
  datasource:
    username: root
    password: 24364726
    url: jdbc:mysql://192.168.88.128:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource # 自定义数据源
```


### 🛠️ 3. Druid 详细配置
```yaml
spring:
  datasource:
    username: root
    password: 24364726
    url: jdbc:mysql://192.168.88.128:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    # Druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    # 配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```


### 📚 4. 添加 Log4j 依赖
```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```


### 🎯 5. 配置 DruidDataSource
```java
@Configuration
public class DruidConfig {
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource() {
        return new DruidDataSource();
    }
}
```

---

## 📈 Druid 监控配置

### 🖥️ 1. 配置监控页面
```java
@Configuration
public class DruidConfig {
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();
        initParams.put("loginUsername", "admin"); // 后台管理界面的登录账号
        initParams.put("loginPassword", "123456"); // 后台管理界面的登录密码
        initParams.put("allow", ""); // 允许所有访问
        initParams.put("deny", "192.168.1.73"); // 拒绝访问的IP
        bean.setInitParameters(initParams);
        return bean;
    }
}
```

配置完成后访问：http://localhost:8080/druid/login.html

### 🌐 2. 配置 Web 监控过滤器
```java
@Bean
public FilterRegistrationBean webStatFilter() {
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new WebStatFilter());
    Map<String, String> initParams = new HashMap<>();
    initParams.put("exclusions", "*.js,*.css,/druid/*,/jdbc/*");
    bean.setInitParameters(initParams);
    bean.setUrlPatterns(Arrays.asList("/*"));
    return bean;
}
```

---

## 🔧 其他数据库连接池配置参考

### 💧 HikariCP 配置示例
```yaml
spring:
  datasource:
    url: jdbc:mysql://192.168.88.128:3306/Notification?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
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
```
