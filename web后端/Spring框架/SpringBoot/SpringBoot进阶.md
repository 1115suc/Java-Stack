# 🌟 Spring Boot 高级原理学习笔记

## 📚 Spring Boot 自动化配置原理

### 🔍 Spring Boot Starter 依赖管理机制

Spring Boot 通过 `spring-boot-dependencies` 管理了大量的官方 `starter`，这些 `starter` 已经帮助我们管理好了版本号。项目中使用时直接引入对应的 `starter` 即可，这个场景下需要的依赖就会自动导入到项目中，简化了繁琐的依赖管理。

所有的场景启动器的底层都依赖 `spring-boot-starter`：
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.10.RELEASE</version>
  <scope>compile</scope>
</dependency>
```


如果需要修改版本可以有两种方式：
- 重写 maven 属性
- 使用 Maven 依赖管理的就近原则

引入 `starter` 不仅仅是帮助我们管理了依赖，还帮我们做了很多的默认配置信息，简化了大量的配置。

### ⚙️ 自动化配置初体验

以 web MVC 自动化配置原理为例，Spring Boot 帮助我们完成了以下自动化配置：

1. 依赖版本和依赖什么 jar 都不需要开发者关注
2. 自动化配置
    - 自动配好 SpringMVC
        - 引入 SpringMVC 全套组件
        - 自动配好 SpringMVC 常用组件（三大组件，文件上传等）
    - 自动配好 Web 常见功能，如：字符编码问题，静态资源管理
3. 自动配好 Tomcat

有了 Spring Boot 以后，让开发人员重点关注业务本身，而不是环境上，提升了开发效率。

### 🛠️ @Configuration 配置注解

`@Configuration` 注解的作用是替代原始 spring 配置文件功能。

```java
@Configuration
public class MyConfig {
    @Bean   // 默认方法名称作为容器中的name
    public User getUser() {
        return new User();
    }
}
```

`proxyBeanMethods` 属性（since spring 5.2 以后）：
- `proxyBeanMethods = true`：Full 模式，保证每个 `@Bean` 方法被调用多少次返回的组件都是单实例的
- `proxyBeanMethods = false`：Lite 模式，每个 `@Bean` 方法被调用多少次返回的组件都是新创建的

组件依赖必须使用 Full 模式默认，Full 模式每次都会检查 bean，效率较 Lite 模式慢。

### 📥 @Import 注解使用

`@Import` 注解的作用是导入的类会被 Spring 加载到 IOC 容器中，提供 4 种用法：

1. 导入 Bean
2. 导入配置类
3. 导入 `ImportSelector` 实现类。一般用于加载配置文件中的类
4. 导入 `ImportBeanDefinitionRegistrar` 实现类

```java
@SpringBootApplication
@Import(User.class)  
//会自动执行当前类的构造方法创建对象，存到IOC容器, bean名称为：类的全路径
public class DataApplication {
  public static void main(String[] args) {
    ConfigurableApplicationContext applicationContext = SpringApplication.run(DataApplication.class, args);
  }
}
```

### 🔄 @Conditional 衍生条件装配

`@Conditional` 注解的作用是条件装配，满足 `Conditional` 指定的条件，则进行组件注入，初始化 Bean 对象到 IOC 容器。

常用的衍生注解包括：
- `@ConditionalOnClass`：当类路径存在指定类时生效
- `@ConditionalOnMissingClass`：当类路径不存在指定类时生效
- `@ConditionalOnBean`：当容器中存在指定 Bean 时生效
- `@ConditionalOnMissingBean`：当容器中不存在指定 Bean 时生效
- `@ConditionalOnProperty`：当配置文件中有指定属性时生效

`@ConditionalOnXXX` 注解存在的意义是：满足条件当前类或者 Bean 才有效，按需导入。

### 📦 @ConfigurationProperties 配置绑定

`@ConfigurationProperties` 配置绑定存在的目的是：获取配置属性或者是配置文件指定前缀的属性信息，并且初始化 Bean 对象到 IOC 容器。由此我们可以想：将来的配置我们可以放在配置文件中，通过这个注解来读取并封装成对象。

### 🚀 @SpringBootApplication 入口分析

`@SpringBootApplication` 是一个组合注解：

- `@SpringBootConfiguration`：是对 `@Configuration` 注解的包装，标识是一个配置类，所以引导类也是配置类
- `@ComponentScan`：组件扫描，默认扫描引导类所在的包及其子包所有带注解的类
- `@EnableAutoConfiguration`：启用自动配置的核心注解

### 🔧 @EnableAutoConfiguration 自动配置注解

`@EnableAutoConfiguration` 也是一个组合注解：

- `@AutoConfigurationPackage`：利用 `Registrar` 给容器中导入一系列组件，默认情况下将引导类的所有包及其子包的组件导入进来
- `@Import(AutoConfigurationImportSelector.class)`：利用 `selectImports` 方法中的 `getAutoConfigurationEntry` 方法给容器中批量导入相关组件

通过 `META-INF/spring.factories` 位置来加载配置文件，默认扫描系统里面所有 `META-INF/spring.factories` 位置的文件。

### 🎯 按条件开启自动配置类和配置项

所有的自动化配置虽然会全部加载，但底层有大量的 `@ConditionalOnXXX`，有很多自动配置类并不能完全开启。

以 `JdbcTemplateAutoConfiguration` 为例：
```java
//配置类，Lite模式
@Configuration(proxyBeanMethods = false)
//存在 DataSource、JdbcTemplate 类时再加载当前类
@ConditionalOnClass({DataSource.class, JdbcTemplate.class })
//容器中只有一个指定的Bean，或者这个Bean是首选Bean 加载当前类
@ConditionalOnSingleCandidate(DataSource.class)
//在配置类 DataSourceAutoConfiguration 之后执行
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
//如果条件满足：开始加载自动化配置的属性值 JdbcProperties
@EnableConfigurationProperties(JdbcProperties.class)
@Import({JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {
}
```


### 📋 Spring Boot 自动化配置流程总结

1. 程序启动找到自动化配置包下 `META-INF/spring.factories` 的 `EnableAutoConfiguration`
2. SpringBoot 先加载所有的自动配置类 `xxxxxAutoConfiguration`
3. 每个自动配置类按照条件进行生效
4. 生效的配置类就会给容器中装配很多组件
5. 只要容器中有这些组件，相当于这些功能就有了
6. 定制化配置
    - 用户直接自己 `@Bean` 替换底层的组件
    - 用户去看这个组件是获取的配置文件什么值就去修改

开发人员使用步骤总结：
- 引入场景依赖
- 查看自动配置了哪些（选做）
    - 自己分析，引入场景对应的自动配置一般都生效了
    - 配置文件中 `debug=true` 开启自动配置报告。Negative（不生效）\Positive（生效）
- 自己分析是否需要修改
- 参照文档修改配置项，`xxxxProperties` 绑定了配置文件的哪些
    - 自定义加入或者替换组件，`@Bean`、`@Component` 等

## 🩺 Spring Boot 健康监控

### 📊 Actuator 健康监控服务

每一个微服务在云上部署以后，我们都需要对其进行监控、追踪、审计、控制等。SpringBoot 就抽取了 `Actuator` 场景，使得我们每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

引入依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```


暴露所有监控信息为 HTTP：
```yaml
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露

  endpoint:
    health:
      enabled: true   # 开启健康检查详细信息
      show-details: always
```


### 👁️ Spring Boot Admin 可视化监控

SpringBoot Admin 有两个角色，客户端(Client)和服务端(Server)。

Spring Boot Admin 为注册的应用程序提供以下功能：
- 显示健康状况
- 显示详细信息，例如 JVM 和内存指标、micrometer.io 指标、数据源指标、缓存指标
- 显示内部信息
- 关注并下载日志文件
- 查看 JVM 系统和环境属性
- 查看 Spring Boot 配置属性
- 支持 Spring Cloud 的可发布/env-和// refresh-endpoint
- 轻松的日志级别管理
- 与 JMX-beans 交互
- 查看线程转储
- 查看 http-traces
- 查看审核事件
- 查看 http 端点
- 查看预定的任务
- 查看和删除活动会话（使用 spring-session）
- 查看 Flyway / Liquibase 数据库迁移
- 下载 heapdump
- 状态更改通知（通过电子邮件，Slack，Hipchat 等）
- 状态更改的事件日志（非持久性）

搭建 Server 端：
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.3.1</version>
</dependency>
```
```java
@SpringBootApplication
@EnableAdminServer
public class AdminApplication {
    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }
}
```


搭建 Client 端：
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.3.1</version>
</dependency>
```
```yaml
spring:   
  boot:
    admin:
      client:
        url: http://localhost:9999  # admin 服务地址
        instance:
          prefer-ip: true   # 显示IP
  application:
    name: boot_data  # 项目名称
    
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露

  endpoint:
    health:
      enabled: true   # 开启健康检查详细信息
      show-details: always
```
