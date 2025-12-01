# 🔄 Swagger2 升级到 Swagger3 指南

## 📋 主要变化对比

### 依赖配置变更

**Swagger2 依赖**
```xml
<!-- Swagger2 依赖 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.9.2</version>
</dependency>

<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.9.2</version>
</dependency>
```


**Swagger3 (OpenAPI3) 依赖**
```xml
<!-- Swagger3 依赖 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-boot-starter</artifactId>
  <version>3.0.0</version>
</dependency>
```


或者使用官方 OpenAPI 实现：
```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-ui</artifactId>
  <version>1.6.14</version>
</dependency>
```


### 配置类变更

**Swagger2 配置类**
```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
   @Bean
   public Docket buildDocket() {
       return new Docket(DocumentationType.SWAGGER_2)
               .select()
               .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
               .paths(PathSelectors.any())
               .build()
               .apiInfo(apiInfo())
               .enable(true)
               .groupName("默认分组");
   }
}
```


**Swagger3 配置类**
```java
@Configuration
@EnableOpenApi
public class SwaggerConfiguration {
   @Bean
   public Docket buildDocket() {
       return new Docket(DocumentationType.OAS_30) // 变更点
               .select()
               .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
               .paths(PathSelectors.any())
               .build()
               .apiInfo(apiInfo())
               .enable(true)
               .groupName("默认分组");
   }
}
```


使用 springdoc-openapi 的配置：
```java
@Configuration
public class OpenApiConfig {
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("API 文档")
                        .version("1.0")
                        .description("这是 API 描述"));
    }
}
```


### 访问路径变更

- **Swagger2**: http://localhost:8080/swagger-ui.html
- **Swagger3**: http://localhost:8080/swagger-ui/index.html 或 http://localhost:8080/swagger-ui/

## 🏷️ 注解变化对比

### 核心注解对照表

| Swagger2 注解 | Swagger3/OpenAPI3 对应注解 | 说明 |
|-------------|-------------------------|-----|
| `@Api` | `@Tag` | 类级别注解，描述控制器 |
| `@ApiOperation` | `@Operation` | 方法级别注解，描述接口操作 |
| `@ApiParam` | `@Parameter` | 参数描述注解 |
| `@ApiModel` | `@Schema` | 实体类描述注解 |
| `@ApiModelProperty` | `@Schema` | 实体属性描述注解 |
| `@ApiIgnore` | `@Hidden` | 隐藏接口注解 |

### 实体类注解变化示例

**Swagger2 实体类**
```java
@ApiModel(description = "用户信息")
public class UserDTO {
    @ApiModelProperty(value = "用户ID", example = "1")
    private Long id;
    
    @ApiModelProperty(value = "用户名", example = "张三")
    private String username;
}
```


**Swagger3 实体类**
```java
@Schema(description = "用户信息")
public class UserDTO {
    @Schema(description = "用户ID", example = "1")
    private Long id;
    
    @Schema(description = "用户名", example = "张三")
    private String username;
}
```


### 控制器注解变化示例

**Swagger2 控制器**
```java
@RestController
@RequestMapping("/api/users")
@Api(tags = "用户管理接口")
public class UserController {
    
    @ApiOperation(value = "获取用户详情", notes = "通过用户ID查询用户详细信息")
    @ApiResponses({
        @ApiResponse(code = 200, message = "成功返回用户信息"),
        @ApiResponse(code = 404, message = "用户未找到")
    })
    @GetMapping("/{id}")
    public User getUser(
            @ApiParam(value = "用户ID", required = true) 
            @PathVariable Long id) {
        return userService.findById(id);
    }
}
```


**Swagger3 控制器**
```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "用户管理接口", description = "用户相关操作")
public class UserController {
    
    @Operation(summary = "获取用户详情", description = "通过用户ID查询用户详细信息")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "成功返回用户信息"),
        @ApiResponse(responseCode = "404", description = "用户未找到")
    })
    @GetMapping("/{id}")
    public User getUser(
            @Parameter(description = "用户ID", required = true) 
            @PathVariable Long id) {
        return userService.findById(id);
    }
}
```


## ⚠️ 升级注意事项

### 1. 包名变更
- Swagger2 使用 `io.swagger.annotations` 包
- Swagger3 使用 `io.swagger.core.v3.annotations` 包

### 2. 配置兼容性
- Spring Boot 2.6+ 版本推荐使用 Swagger3 或 springdoc-openapi
- 无需再配置 `ant_path_matcher` 策略

### 3. 功能增强
- Swagger3 支持 OpenAPI 3.0 规范
- 更好的 Spring Boot 集成
- 支持更多媒体类型和安全方案

### 4. 迁移建议
1. 逐步替换注解，优先迁移核心接口
2. 测试文档生成和 UI 显示效果
3. 更新团队开发文档和规范
4. 验证所有接口的文档描述准确性


## 🔍 Swagger 新版本与老版本核心区别

### 📦 依赖项配置差异

**Swagger2 (老版本)**
```xml
<!-- 需要添加两个依赖项 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.9.2</version>
</dependency>

<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.9.2</version>
</dependency>
```


**Swagger3 (新版本)**
```xml
<!-- 只需要添加一个启动器依赖 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-boot-starter</artifactId>
  <version>3.0.0</version>
</dependency>
```


### ⚡ 启动注解变更

- **老版本**：使用 `@EnableSwagger2` 注解启用 Swagger
- **新版本**：使用 `@EnableOpenApi` 注解启用 Swagger

### 📄 文档类型配置差异

- **老版本**：`Docket` 配置使用 `DocumentationType.SWAGGER_2`
- **新版本**：`Docket` 配置使用 `DocumentationType.OAS_30`

### 🌐 访问地址变化

- **老版本访问地址**：http://localhost:8080/swagger-ui.html
- **新版本访问地址**：http://localhost:8080/swagger-ui/index.html 或 http://localhost:8080/swagger-ui/

> 💡 **小贴士**：新版本简化了配置流程，提供了更好的 OpenAPI 3.0 支持和 Spring Boot 集成体验