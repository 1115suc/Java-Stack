# 📚 Swagger2 接口文档工具

## 🔧 项目集成 Swagger

### 🚀 启动项目并访问

- 启动 Spring Boot 项目后，访问：
    - 📄 文档页面：http://localhost:8080/swagger-ui.html
    - 📊 接口数据：http://localhost:8080/v2/api-docs

- ⚠️ 注意事项
    - 从 `Spring Boot 2.6` 开始默认使用 `PathPatternParser` 替代了 `AntPathMatcher`，而 `Springfox` 不兼容，需加上这个配置:

```yaml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```


### 📦 引入依赖

```xml
<!--swagger2接口文档的api工具-->
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


### ⚙️ 定义 Swagger 配置类

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
   @Bean
   public Docket buildDocket() {
      //构建在线API概要对象
       return new Docket(DocumentationType.SWAGGER_2)
               .select()
               .apis(RequestHandlerSelectors.basePackage("com.example.controller")) // 扫描包路径
               .paths(PathSelectors.any()) // 路径过滤规则
               .build()
               .apiInfo(apiInfo()) // 文档基本信息
               .enable(true) // 是否启用 Swagger（默认 true）
               .groupName("默认分组") // 分组名称
               .securitySchemes(securitySchemes()) // 全局认证方案（如 Token）
               .securityContexts(securityContexts()); // 安全上下文
   }
   
   private ApiInfo buildApiInfo() {
      //网站联系方式
      Contact contact = new Contact("1115suc","https://www.1115suc.com/","1115suc@gmail.com");
      return new ApiInfoBuilder()
              .title("Swagger2接口文档")//文档标题
              .description("这是一个方便前后端开发人员快速了解开发接口需求的在线接口API文档")//文档描述信息
              .contact(contact)//站点联系人相关信息
              .version("1.0.0")//文档版本
              .build();
   }
}
```


## 🏷️ 注解使用指南

### 📌 基本注解使用示例

- 推荐使用Idea插件 `Swagger Tools` 来辅助生成Swagger注解

![img.png](img/img.png)


```java
@RestController
@RequestMapping("/api")
@Api(value = "用户认证相关接口定义", tags = "用户功能-用户登录功能")
public class UserController {
    @Autowired
    private StockService stockService;
    
    @ApiImplicitParams({
            @ApiImplicitParam(paramType = "query", dataType = "int", name = "page", value = ""),
            @ApiImplicitParam(paramType = "query", dataType = "int", name = "pageSize", value = "")
    })
    @ApiOperation(value = "分页查询股票最新数据，并按照涨幅排序查询",
                  notes = "分页查询股票最新数据，并按照涨幅排序查询", 
                  httpMethod = "GET",
                  response = R.class
    )
    @GetMapping("/stock/all")
    public R<PageResult<StockUpdownDomain>> getStockPageInfo(
            @ApiParam(value = "当前页", required = false, defaultValue = "1") 
            @RequestParam(name = "page", required = false, defaultValue = "1") Integer page,
            @ApiParam(value = "每页大小", required = false, defaultValue = "20") 
            @RequestParam(name = "pageSize", required = false, defaultValue = "20") Integer pageSize) {
        return stockService.getStockPageInfo(page, pageSize);
    }

    @ApiImplicitParams({
            @ApiImplicitParam(paramType = "query", dataType = "int", name = "page", value = "当前页"),
            @ApiImplicitParam(paramType = "query", dataType = "int", name = "pageSize", value = "每页大小")
    })
    @ApiOperation(value = "将指定页的股票数据导出到excel表下",
                  notes = "将指定页的股票数据导出到excel表下",
                  httpMethod = "GET",
                  response = R.class
    )
    @GetMapping("/stock/export")
    public void stockExport(HttpServletResponse response,
                             @ApiParam(value = "当前页", required = false, defaultValue = "1") 
                             @RequestParam(name = "page", required = false, defaultValue = "1") Integer page,
                             @ApiParam(value = "每页大小", required = false, defaultValue = "20") 
                             @RequestParam(name = "pageSize", required = false, defaultValue = "20") Integer pageSize) {
        stockService.stockExport(response, page, pageSize);
    }
}
```
```java
@ConfigurationProperties(prefix = "stock")
@Data
@Api(tags = "股票信息配置")
public class StockInfoConfig {
    @ApiModelProperty("A股大盘ID集合")
    private List<String> inner;

    @ApiModelProperty("外盘ID集合")
    private List<String> outer;
    
    // ...
}
```


### 🎯 其他注解使用示例

```java
@RestController
@RequestMapping("/api/products")
@Api(tags = "商品管理接口")
public class ProductController {
    @ApiOperation(value = "获取商品详情")
    @ApiResponses({
            @ApiResponse(code = 200, message = "成功返回商品信息"),
            @ApiResponse(code = 404, message = "商品未找到"),
            @ApiResponse(code = 500, message = "服务器内部错误")
    })
    @GetMapping("/{id}")
    public Product getProduct(
            @ApiParam(value = "商品ID", required = true)
            @PathVariable Long id) {
        return productService.findById(id);
    }

    @ApiOperation(value = "搜索商品列表")
    @ApiResponse(code = 200, message = "成功返回商品列表")
    @GetMapping("/search")
    public List<Product> searchProducts(
            @ApiParam(value = "搜索关键字")
            @RequestParam(required = false) String keyword) {
        return productService.search(keyword);
    }

    @ApiOperation(value = "删除商品")
    @ApiIgnore  // 此接口不会在Swagger文档中显示
    @DeleteMapping("/{id}")
    public void deleteProduct(@PathVariable Long id) {
        productService.deleteById(id);
    }
}
```


## 📖 Swagger 注解详解

| 注解 | 位置 | 说明 |
|-----|-----|-----|
| `@Api` | 类 | 加载 Controller 类上，表示对类的说明 |
| `@ApiModel` | 类(通常是实体类) | 描述实体类的作用，通常表示接口接收参数的实体对象 |
| `@ApiModelProperty` | 属性 | 描述实体类的属性（用对象接收参数时，描述对象的一个字段） |
| `@ApiOperation` | 方法 | 说明方法的用途、作用 |
| `@ApiImplicitParams` | 方法 | 表示一组参数说明 |
| `@ApiImplicitParam` | 方法 | 用在 `@ApiImplicitParams` 注解中，指定一个请求参数的各个方面的属性 |
| `@ApiParam` | 方法入参或者方法之上 | 单个参数的描述信息，描述 form 表单、url 参数 |

### 📋 `@ApiImplicitParam` 注解详解

- **paramType**（查询参数类型）
    - `path`：以地址的形式（rest风格）提交数据
    - `query`：直接跟参数完成自动映射赋值(/add/user?name=Bob)
    - `body`：以流的形式提交 仅支持POST
    - `header`：参数在request headers 里边提交
    - `form`：以form表单的形式提交 仅支持POST

- **dataType**（参数的数据类型）
    - 参数的数据类型只作为标志说明，并没有实际验证
    - `Long`
    - `String`

- **name**（接收参数名）
    - 接收参数名(方法入参的名称)

- **value**（接收参数的意义描述）
    - 接收参数的意义描述（描述信息）

- **required**（参数是否必填）
    - `true`：必填
    - `false`：非必填

- **defaultValue**（默认值）
    - 默认值

> 💡 其它注解:
> - `@ApiResponse`：HTTP响应其中1个描述
> - `@ApiResponses`：HTTP响应整体描述
> - `@ApiIgnore`：使用该注解忽略这个API
> - `@ApiError`：发生错误返回的信息