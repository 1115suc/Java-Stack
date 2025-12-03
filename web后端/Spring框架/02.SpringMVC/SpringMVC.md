# 🌟 Spring MVC 基础

## 📚 Spring MVC 框架介绍

### 🔁 Spring MVC 是什么
- Spring MVC 是 Spring 框架的一部分，专门用于构建 Web 应用程序
- 它遵循 Model-View-Controller 设计模式，将应用程序的不同方面分离
- 主要负责处理 HTTP 请求和响应，实现 Web 层的功能

### ⚙️ Spring MVC 核心功能
- controller如何接收请求和数据
- 如何将请求和数据转发给业务层
- 如何将响应数据转换成json发回到前端

### 🏗️ Spring MVC 架构图
![1669630054810.png](img/1669630054810.png)

---

## 🎯 Spring MVC 核心组件

### 🧩 主要组件说明

1. **DispatcherServlet**: 前端控制器，负责接收所有请求并分发给相应的处理器
2. **HandlerMapping**: 处理器映射器，用于查找合适的处理器来处理请求
3. **Controller**: 控制器，实际处理请求的组件
4. **ModelAndView**: 模型和视图，封装了处理结果和视图信息
5. **ViewResolver**: 视图解析器，解析视图名称并返回具体视图
6. **View**: 视图，负责渲染结果并返回给客户端

---

## 🎯 Spring MVC 入门

### 🔧 Spring MVC 实例

1. 导入 Spring MVC 坐标
   ```xml
   <dependencies>
       <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>javax.servlet-api</artifactId>
          <version>3.1.0</version>
          <scope>provided</scope>
        </dependency>
   
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-webmvc</artifactId>
           <version>5.2.10.RELEASE</version>
       </dependency>
   </dependencies>
   ```


2. 定义 Controller 类
   ```java
    @Controller
    public class UserController {
        @RequestMapping("/save")
        @ResponseBody
        public String save(){
            System.out.println("user save ...");
            return "springmvc";
        }
    }
   ```


3. 创建 Spring MVC 配置类
   ```java
    @Configuration
    @ComponentScan("course.controller")
    public class SpringMvcConfig {
    }
   ```


4. 创建Tomcat的Servlet容器配置类
   ```java
       public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
        //加载springMVC配置
        protected WebApplicationContext createServletApplicationContext() {
            //初始化WebApplicationContext对象
            AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
            //加载指定配置类
            ctx.register(SpringMvcConfig.class);
            return ctx;
        }
        //设置Tomcat接收的请求哪些归SpringMVC处理
        protected String[] getServletMappings() {
            return new String[]{"/"};
        }
    
        //设置spring相关配置
        protected WebApplicationContext createRootApplicationContext() {
            return null;
        }
    }
   ```


---

## 🛠️ Spring MVC 核心注解

### 📍 请求映射注解

- `@RequestMapping`: 请求映射通用注解，可用于类或方法级别
- `@GetMapping`: 处理 GET 请求的映射注解
- `@PostMapping`: 处理 POST 请求的映射注解
- `@PutMapping`: 处理 PUT 请求的映射注解
- `@DeleteMapping`: 处理 DELETE 请求的映射注解
- `@PatchMapping`: 处理 PATCH 请求的映射注解

### 📥 数据绑定注解

- `@RequestParam`: 绑定请求参数到方法参数
- `@PathVariable`: 绑定 URL 中的占位符到方法参数
- `@RequestBody`: 将 HTTP 请求体绑定到方法参数
- `@RequestHeader`: 绑定请求头信息到方法参数
- `@CookieValue`: 绑定 Cookie 值到方法参数
- `@MatrixVariable`: 绑定矩阵变量到方法参数

### 📤 响应处理注解

- `@ResponseBody`: 将方法返回值直接写入 HTTP 响应体
- `@ModelAttribute`: 将方法参数或方法返回值绑定到模型属性

### 🏷️ 类级别注解

- `@Controller`: 标识一个类为控制器组件
- `@RestController`: 控制器类注解，组合了 `@Controller` 和 `@ResponseBody`
- `@SessionAttributes`: 声明需要存储在会话中的模型属性

---

## 💉 Spring MVC 参数接收

### 📊 基本数据类型参数

#### URL地址传参和表单传参
- 形参与参数名相同：可以直接获取请求参数
- 形参与参数名不同：需要使用 `@RequestParam` 绑定请求参数与方法形参

#### @RequestParam 注解说明
| 名称 | @RequestParam |
|------|---------------|
| 类型 | 形参注解 |
| 位置 | SpringMVC控制器方法形参定义前面 |
| 作用 | 绑定请求参数与处理器方法形参间的关系 |
| 相关参数 | required：是否为必传参数<br/>defaultValue：参数默认值 |

### 📦 POJO参数

简单数据类型一般处理的是参数个数比较少的请求，如果参数比较多，可以考虑使用POJO数据类型。

- POJO参数：请求参数名与形参对象属性名相同，定义POJO类型形参即可接收参数

```java
public class User {
    private String name;
    private Integer age;
    //setter...getter...略
}

//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
```


**注意:** 请求参数key的名称要和POJO中属性的名称一致，否则无法封装

### 📚 数组参数

请求参数名与形参对象属性名相同且请求参数为多个，定义数组类型即可接收参数

```java
//数组参数：同名请求参数可以直接映射到对应名称的形参数组对象中
@RequestMapping("/arrayParam")
@ResponseBody
public String arrayParam(String[] likes){
    System.out.println("数组参数传递 likes ==> "+ Arrays.toString(likes));
    return "{'module':'array param'}";
}
```


---

## 🔄 Spring MVC 请求处理流程

![1669464860746.png](img/1669464860746.png)

### 🔄 完整处理流程

1. 用户发送请求至前端控制器 `DispatcherServlet`
2. `DispatcherServlet` 收到请求后调用 `HandlerMapping` 处理器映射器
3. `HandlerMapping` 找到具体的处理器(可以根据 xml 配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给 `DispatcherServlet`
4. `DispatcherServlet` 调用 `HandlerAdapter` 处理器适配器
5. `HandlerAdapter` 经过适配调用具体的 `Controller` (也叫后端控制器)
6. `Controller` 执行完成返回 `ModelAndView`
7. `HandlerAdapter` 将 `Controller` 执行结果 `ModelAndView` 返回给 `DispatcherServlet`
8. `DispatcherServlet` 将 `ModelAndView` 传给 `ViewReslover` 视图解析器
9. `ViewReslover` 解析后返回具体 `View`
10. `DispatcherServlet` 根据 `View` 进行渲染(即将模型数据填充至视图中)
11. `DispatcherServlet` 响应用户

---

## 📦 Spring MVC 依赖总结

### 核心依赖

#### 1. Spring MVC 核心依赖
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
```


- **作用**：提供 Spring MVC 框架的核心功能，包括 `DispatcherServlet`、注解支持、请求处理等

#### 2. Servlet API 依赖
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```


- **作用**：提供 Servlet 规范的 API 支持，用于处理 HTTP 请求和响应
- **注意**：scope 设置为 provided，因为容器会提供此依赖

### 功能扩展依赖

#### 3. JSON 处理依赖
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```


- **作用**：提供 JSON 数据的序列化和反序列化功能
- **用途**：支持 `@RequestBody` 和 `@ResponseBody` 注解处理 JSON 数据
- **配合**：需要在配置类上添加 `@EnableWebMvc` 注解启用 JSON 转换功能
