# 🗡️ knife4j 接口文档增强工具

## 🔧 项目集成 knife4j

### 📦 引入依赖

```xml
<!--knife4j的依赖-->
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-spring-boot-starter</artifactId>
  <version>2.0.2</version>  
</dependency>
<!--支持接口参数校验处理-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### ⚙️ 配置 knife4j

```java
@Configuration
@EnableSwagger2
@EnableKnife4j
@Import(BeanValidatorPluginsConfiguration.class)
public class SwaggerConfiguration {
    // 配置内容与 Swagger2 基本一致
}
```

### 🚀 访问在线文档

- **knife4j 文档页面**：http://localhost:8080/doc.html
- **原始 Swagger 页面**：http://localhost:8080/swagger-ui.html

## 🏷️ 注解使用说明

- knife4j 完全兼容 Swagger2 的注解体系，使用方式与 `Swagger2` 完全一致：

## 📊 knife4j 增强特性

### 🔧 主要优势

- **更美观的UI界面**：相比原生 Swagger 提供更友好的文档展示
- **离线文档支持**：支持导出 HTML、Markdown、Word 等格式文档
- **接口调试增强**：提供更强大的在线调试功能
- **权限控制**：支持文档访问权限控制
- **国际化支持**：多语言界面支持


> 💡 **提示**：knife4j 是基于 Swagger 的增强 UI 实现，完全兼容 Swagger2 的注解和配置方式，无需修改原有代码即可获得更好的文档体验。