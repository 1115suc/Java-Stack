# 🌟 Spring Boot 基础学习笔记

## 📚 Spring Boot 简介

### 🔁 什么是 Spring Boot
- Spring Boot 是 Spring 框架的一个扩展，旨在简化 Spring 应用的初始搭建和开发过程
- 它采用"约定优于配置"的理念，让开发者能够快速构建独立的、生产级别的 Spring 应用
- 提供了自动配置、起步依赖、内嵌服务器等特性，大大减少了配置工作

### ⚙️ Spring Boot 核心特性
- **自动配置**：根据项目依赖自动配置 Spring 应用
- **起步依赖**：简化 Maven/Gradle 配置，提供一站式依赖解决方案
- **内嵌服务器**：内置 Tomcat、Jetty、Undertow 等服务器，无需部署 WAR 文件
- **生产就绪**：提供健康检查、指标监控、外部化配置等功能
- **无代码生成**：不生成冗余代码，保持代码简洁

### 🎯 Spring Boot 解决的问题
- 简化 Spring 应用的搭建和配置过程
- 减少样板代码和重复配置
- 快速构建微服务和 RESTful API
- 提供开箱即用的功能集成

---

## 🚀 Spring Boot 快速入门

### 🔧 创建 Spring Boot 项目

#### 1. 使用 Spring Initializr 创建项目
- 访问 https://start.spring.io/
- 选择项目类型、语言、Spring Boot 版本
- 添加所需依赖（如 Spring Web、Spring Data JPA 等）
- 下载并解压项目

#### 2. 手动创建 Maven 项目
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.0</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```


### 🏗️ 第一个 Spring Boot 应用

#### 主启动类
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```


#### 控制器示例
```java
@RestController
@RequestMapping("/api")
public class HelloController {
    
    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

### 🌟 banner 图案

- 到项目下的 `resources` 目录下新建一个`banner.txt` 即可
- 网站：https://www.bootschool.net/ascii

---

## 🛠️ Spring Boot 核心注解

### 📍 核心启动注解

- `@SpringBootApplication`: 标识 Spring Boot 主启动类，组合了多个注解
- `@EnableAutoConfiguration`: 启用 Spring Boot 的自动配置机制
- `@ComponentScan`: 扫描当前包及其子包下的组件

### 📍 配置相关注解

- `@Configuration`: 标识配置类
- `@ConfigurationProperties`: 绑定配置属性到 Java 对象
- `@Value`: 注入配置文件中的属性值
- `@PropertySource`: 指定配置文件位置

---

## 📦 Spring Boot 起步依赖

### 🎯 起步依赖概念
- 起步依赖是一组方便的依赖描述符，可以一次性添加到项目中
- 它们包含了一系列相互兼容的库，避免了版本冲突问题
- 通过传递依赖机制，自动引入所需的其他依赖

### 📋 常用起步依赖

#### Web 开发
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```


#### 数据访问
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```


#### 测试
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```


#### 安全
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```


---

## ⚙️ Spring Boot 自动配置

### 🎯 自动配置原理
- Spring Boot 通过 `@EnableAutoConfiguration` 注解启用自动配置
- 在启动时扫描 `META-INF/spring.factories` 文件中的配置类
- 根据类路径中的依赖和条件注解决定是否应用配置

### 🛠️ 条件注解

- `@ConditionalOnClass`: 当类路径存在指定类时生效
- `@ConditionalOnMissingClass`: 当类路径不存在指定类时生效
- `@ConditionalOnBean`: 当容器中存在指定 Bean 时生效
- `@ConditionalOnMissingBean`: 当容器中不存在指定 Bean 时生效
- `@ConditionalOnProperty`: 当配置文件中有指定属性时生效

### 🔧 自定义自动配置
```java
@Configuration
@ConditionalOnClass(MyService.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```


---

## 📤 Spring Boot 配置管理

### 🎯 外部化配置

#### application.properties 配置文件
```properties
# 服务器配置
server.port=8080
server.servlet.context-path=/api

# 数据库配置
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root

# 日志配置
logging.level.root=INFO
logging.level.com.example=DEBUG
```


#### application.yml 配置文件
```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: root

logging:
  level:
    root: INFO
    com.example: DEBUG
```


### 🛠️ 配置属性绑定

#### 使用 @Value 注入配置
```java
@Component
public class AppConfig {
    
    @Value("${app.name:DefaultApp}")
    private String appName;
    
    @Value("${app.version:1.0.0}")
    private String appVersion;
}
```


#### 使用 `@ConfigurationProperties` 注入配置
```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private String version;
    private List<String> features;
    
    // getter and setter methods
}
```

#### 路径变量参数
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public String getUserById(@PathVariable Long id) {
        return "User ID: " + id;
    }
}
```

### 📋 配置文件加载顺序

Spring Boot 配置文件按以下优先级加载（从高到低）：

1. **application.properties** > **application.yml** > **application.yaml**

#### 4级配置文件位置

1. **1级**: `file:config/application.yml` 【最高优先级】
2. **2级**: `file:application.yml`
3. **3级**: `classpath:config/application.yml`
4. **4级**: `classpath:application.yml` 【最低优先级】

#### 作用说明

- **1级与2级**: 留作系统打包后设置通用属性，适合生产环境配置
- **3级与4级**: 用于系统开发阶段设置通用属性，适合开发环境配置

#### 注意事项

1. **Spring Boot核心配置文件名为application**
2. **Spring Boot内置属性过多，且所有属性集中在一起修改，在使用时，通过提示键+关键字修改属性**

---

## 🚀 Spring Boot 项目部署

### 📦 项目打包与启动

#### 1. Maven 打包
执行 Maven 构建指令 `package` 对 Spring Boot 项目进行打包

#### 2. 启动指令
```cmd
java -jar springboot_01_quickstart.jar  # 项目的名称根据实际情况修改
```


#### 3. Maven 插件配置
为了支持命令行启动，需要在 `pom.xml` 中配置 Spring Boot 对应的 maven 插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```


**注意事项：**
- jar 支持命令行启动需要依赖 maven 插件支持，请确认打包时是否具有 Spring Boot 对应的 maven 插件




