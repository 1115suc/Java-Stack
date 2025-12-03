# 🌟 Spring MVC 进阶

## 🌐 RESTful 风格

### 🎯 RESTful 简介
- **REST**(Representational State Transfer)：表现层状态转移，是一种软件架构风格
- **RESTful**：符合 REST 风格的 Web 服务设计
- 强调资源的概念，每个 URI 代表一种资源
- 通过 HTTP 动词对服务器资源进行操作

### 📋 RESTful 设计原则

#### 1. URI 设计规范
- 使用名词表示资源，不使用动词
- 使用复数形式命名资源集合
- 使用斜杠表示资源层级关系
- 使用连字符分隔多个单词

#### 2. HTTP 动词映射
| HTTP方法 | 行为 | 说明 |
|---------|------|------|
| GET | 查询 | 获取资源信息 |
| POST | 创建 | 新建资源 |
| PUT | 更新 | 更新资源(全量更新) |
| PATCH | 更新 | 更新资源(部分更新) |
| DELETE | 删除 | 删除资源 |

#### 3. 状态码规范
- 2xx：操作成功
- 3xx：重定向
- 4xx：客户端错误
- 5xx：服务器错误

### 🛠️ Spring MVC 中的 RESTful 实现

#### RESTful 控制器示例
```java
@RestController
@RequestMapping("/users")
public class UserController {
    
    // GET /users - 查询所有用户
    @GetMapping
    public List<User> getAllUsers() {
        // 实现查询逻辑
        return userService.findAll();
    }
    
    // GET /users/{id} - 根据ID查询用户
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        // 实现查询逻辑
        return userService.findById(id);
    }
    
    // POST /users - 创建用户
    @PostMapping
    public User createUser(@RequestBody User user) {
        // 实现创建逻辑
        return userService.save(user);
    }
    
    // PUT /users/{id} - 更新用户
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        // 实现更新逻辑
        user.setId(id);
        return userService.update(user);
    }
    
    // DELETE /users/{id} - 删除用户
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        // 实现删除逻辑
        userService.deleteById(id);
    }
}
```


#### RESTful 参数处理
- 使用 `@PathVariable` 获取 URI 中的资源标识符
- 使用 `@RequestBody` 接收请求体中的资源数据
- 使用 `@RequestParam` 处理查询参数

---

## ⚠️ Spring MVC 异常处理

### 🎯 异常处理概述

在程序开发过程中不可避免地会遇到各种异常现象，各层级均可能出现异常：

- **框架内部异常**：因使用不合规导致
- **数据层异常**：因外部服务器故障导致（如服务器访问超时）
- **业务层异常**：因业务逻辑书写错误导致（如索引异常等）
- **表现层异常**：因数据收集、校验等规则导致（如数据类型不匹配）
- **工具类异常**：因工具类书写不严谨导致（如连接未释放等）

### 🛠️ 异常处理器实现

#### 统一异常处理类
```java
@RestControllerAdvice  //用于标识当前类为REST风格对应的异常处理器
public class ProjectExceptionAdvice {

    //统一处理所有的Exception异常
    @ExceptionHandler(Exception.class)
    public Result doOtherException(Exception ex){
        return new Result(666,null);
    }
}
```


#### @RestControllerAdvice 注解说明
| 名称 | @RestControllerAdvice |
|------|----------------------|
| 类型 | 类注解 |
| 位置 | Rest风格开发的控制器增强类定义上方 |
| 作用 | 为Rest风格开发的控制器类做增强 |
| 说明 | 此注解自带@ResponseBody注解与@Component注解 |

#### @ExceptionHandler 注解说明
| 名称 | @ExceptionHandler |
|------|------------------|
| 类型 | 方法注解 |
| 位置 | 专用于异常处理的控制器方法上方 |
| 作用 | 设置指定异常的处理方案 |
| 说明 | 出现异常后终止原始控制器执行，并转入当前方法执行 |

### 📋 项目异常分类与处理方案

#### 异常分类
1. **业务异常**（BusinessException）
    - 规范的用户行为产生的异常
    - 不规范的用户行为操作产生的异常
2. **系统异常**（SystemException）
    - 项目运行过程中可预计且无法避免的异常
3. **其他异常**（Exception）
    - 编程人员未预期到的异常

#### 处理方案
- **业务异常**：发送对应消息传递给用户，提醒规范操作
- **系统异常**：发送固定消息安抚用户，发送特定消息给运维人员，记录日志
- **其他异常**：发送固定消息安抚用户，发送特定消息给编程人员，记录日志

### 🔧 异常处理代码实现

#### 自定义异常类

##### 系统级异常
```java
//自定义异常处理器，用于封装异常信息，对异常进行分类
public class SystemException extends RuntimeException{
    private Integer code;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public SystemException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public SystemException(Integer code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }
}
```


##### 业务级异常
```java
//自定义异常处理器，用于封装异常信息，对异常进行分类
public class BusinessException extends RuntimeException{
    private Integer code;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(Integer code,String message,Throwable cause) {
        super(message, cause);
        this.code = code;
    }
}
```


#### 异常编码定义
```java
public class Code {
    public static final Integer SYSTEM_ERR = 50001;
    public static final Integer SYSTEM_TIMEOUT_ERR = 50002;
    public static final Integer SYSTEM_UNKNOW_ERR = 59999;
    public static final Integer BUSINESS_ERR = 60002;
}
```


#### 触发自定义异常示例
```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    private BookDao bookDao;

    public Book getById(Integer id) {
        //模拟业务异常，包装成自定义异常
        if(id <0){
            throw new BusinessException(Code.BUSINESS_ERR,"请不要使用你的技术挑战我的耐性!");
        }
    }
}
```


#### 异常处理器完整实现
```java
@RestControllerAdvice //用于标识当前类为REST风格对应的异常处理器
public class ProjectExceptionAdvice {
    //@ExceptionHandler用于设置当前处理器类对应的异常类型
    @ExceptionHandler(SystemException.class)
    public Result doSystemException(SystemException ex){
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,ex对象发送给开发人员
        return new Result(ex.getCode(),null,ex.getMessage());
    }

    @ExceptionHandler(BusinessException.class)
    public Result doBusinessException(BusinessException ex){
        return new Result(ex.getCode(),null,ex.getMessage());
    }

    //除了自定义的异常处理器，保留对Exception类型的异常处理，用于处理非预期的异常
    @ExceptionHandler(Exception.class)
    public Result doOtherException(Exception ex){
        //记录日志
        //发送消息给运维
        //发送邮件给开发人员,ex对象发送给开发人员
        return new Result(Code.SYSTEM_UNKNOW_ERR,null,"系统繁忙，请稍后再试！");
    }
}
```


---

## 🚦 Spring MVC 拦截器

### 🎯 拦截器简介

#### 拦截器概念和作用
- **拦截器**（Interceptor）是一种动态拦截方法调用的机制，在SpringMVC中动态拦截控制器方法的执行

#### 拦截器和过滤器的区别
- **归属不同**：`Filter` 属于Servlet技术，`Interceptor` 属于SpringMVC技术
- **拦截内容不同**：`Filter` 对所有访问进行增强，`Interceptor` 仅针对SpringMVC的访问进行增强

![image-20210805180846313.png](img/image-20210805180846313.png)

### 🔧 拦截器入门案例

#### 拦截器代码实现

##### 定义拦截器
```java
@Component //注意当前类必须受Spring容器控制
//定义拦截器类，实现HandlerInterceptor接口
public class ProjectInterceptor implements HandlerInterceptor {
    @Override
    //原始方法调用前执行的内容
    //返回值类型可以拦截控制的执行，true放行，false终止
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle...");
        return true;
    }

    @Override
    //原始方法调用后执行的内容
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle...");
    }

    @Override
    //原始方法调用完成后执行的内容
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion...");
    }
}
```


##### 配置加载拦截器
```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    @Autowired
    private ProjectInterceptor projectInterceptor;

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        //配置拦截器
        registry.addInterceptor(projectInterceptor)
            .addPathPatterns("/books","/books/*");
    }
}
```


使用标准接口 `WebMvcConfigurer` 简化开发：
```java
@Configuration
@ComponentScan({"course.controller"})
@EnableWebMvc
//实现WebMvcConfigurer接口可以简化开发，但具有一定的侵入性
public class SpringMvcConfig implements WebMvcConfigurer {
    @Autowired
    private ProjectInterceptor projectInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //配置多拦截器
        registry.addInterceptor(projectInterceptor)
            .addPathPatterns("/books","/books/*");
    }
}
```


### 📥 拦截器参数详解

#### 前置处理方法
```java
//原始方法调用前执行的内容
//返回值类型可以拦截控制的执行，true放行，false终止
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    System.out.println("preHandle...");
    return true;
}
```


- **参数**：
    1. `request`: 请求对象
    2. `response`: 响应对象
    3. `handler`: 被调用的处理器对象，本质上是一个方法对象
- **返回值**：返回值为 `false`，被拦截的处理器将不执行

#### 后置处理方法
```java
//原始方法调用后执行的内容
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    System.out.println("postHandle...");
}
```


- **参数**：`modelAndView`：如果处理器执行完成具有返回结果，可以读取到对应数据与页面信息
- **注意**：如果处理器方法出现异常了，该方法不会执行

#### 完成后处理方法
```java
//原始方法调用完成后执行的内容
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    System.out.println("afterCompletion...");
}
```


- **参数**：`ex`: 如果处理器执行过程中出现异常对象，可以针对异常情况进行单独处理
- **注意**：无论处理器方法内部是否出现异常，该方法都会执行

### 🔗 拦截器链配置

#### 多个拦截器配置

##### 定义第二个拦截器
```java
@Component
public class ProjectInterceptor2 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle...222");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle...222");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion...222");
    }
}
```


##### 配置多个拦截器
```java
@Configuration
@ComponentScan({"com.itheima.controller"})
@EnableWebMvc
//实现WebMvcConfigurer接口可以简化开发，但具有一定的侵入性
public class SpringMvcConfig implements WebMvcConfigurer {
    @Autowired
    private ProjectInterceptor projectInterceptor;
    @Autowired
    private ProjectInterceptor2 projectInterceptor2;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //配置多拦截器
        registry.addInterceptor(projectInterceptor)
            .addPathPatterns("/books","/books/*");
        registry.addInterceptor(projectInterceptor2)
            .addPathPatterns("/books","/books/*");
    }
}
```


#### 拦截器链工作流程
- 当配置多个拦截器时，形成拦截器链
- 拦截器链的运行顺序参照拦截器添加顺序为准
- 当拦截器中出现对原始处理器的拦截，后面的拦截器均终止运行
- 当拦截器运行中断，仅运行配置在前面的拦截器的 `afterCompletion` 操作
-
![image-20210805181537718.png](img/image-20210805181537718.png)

---

## 💉 Spring MVC 高级参数接收

### 🏗️ 嵌套POJO参数

#### 新增POJO类
```java
public class User {
    private String name;
    private Integer age;
    private Address address;
    //setter...getter...略
}

public class Address {
    private String province;
    private String city;
    //setter...getter...略
}
```


#### 后台接收参数
```java
//POJO参数：请求参数与形参对象中的属性对应即可完成参数传递
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
```


### 🧺 集合参数（List、Map）

#### List参数
```java
//集合参数：同名请求参数可以使用@RequestParam注解映射到对应名称的集合对象中作为数据
@RequestMapping("/listParam")
@ResponseBody
public String listParam(@RequestParam List<String> likes){
    System.out.println("集合参数传递 likes ==> "+ likes);
    return "{'module':'list param'}";
}
```


#### Map参数
```java
@RequestMapping("/mapParam")
@ResponseBody
public String mapParam(@RequestParam Map<String,String> maps) {
    System.out.println(maps);
    return "{'module':'mapParam'}";
}
```


### 📅 Date日期类型参数

```java
@RequestMapping("/dateParam")
@ResponseBody
public String dateParam(Date date,@DateTimeFormat(pattern = "yyyy-MM-dd") Date date2,@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") Date date3){
    System.out.println("date:"+date);
    System.out.println("date2:"+date2);
    System.out.println("date3:"+date3);
    return "{'module':'dateParam'}";
}
```


#### @DateTimeFormat 注解说明
| 名称 | @DateTimeFormat |
|------|-----------------|
| 类型 | 形参注解 |
| 位置 | SpringMVC控制器方法形参前面 |
| 作用 | 设定日期时间型数据格式 |
| 相关属性 | pattern：指定日期时间格式字符串 |

**注意:** SpringMvc默认时间格式为 yyyy/MM/dd，其他时间格式需要使用`@DateTimeFormat`转换

### 📎 File文件类型参数

#### 前期准备
1. 添加fileupload依赖
```xml
<!--添加fileupload依赖-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```


2. 配置解析器
```java
@Bean("multipartResolver")
public CommonsMultipartResolver multipartResolver (){
    CommonsMultipartResolver resolver = new CommonsMultipartResolver();
    resolver.setDefaultEncoding("UTF-8");
    resolver.setMaxUploadSize(1024*1024);
    return resolver;
}
```


#### 后台接收参数
```java
@RequestMapping("/fileParam")
@ResponseBody
public String fileParam(MultipartFile file){
    if(!file.isEmpty()){
        try {
            file.transferTo(new File("D://test.txt"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    return "{'module':'file'}";
}
```


### 📦 JSON类型参数

#### 前期准备
- 添加jackson依赖
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```


- 开启SpringMVC注解支持
```java
@Configuration
@ComponentScan("course.controller")
//开启json数据类型自动转换
@EnableWebMvc
public class SpringMvcConfig {
}
```


#### JSON参数接收
```java
//JSON对象数据
@RequestMapping("/pojoParamForJson")
@ResponseBody
public String pojoParamForJson(@RequestBody User user) {
    System.out.println(user);
    return "{'module':'pojoParamForJson'}";
}

//JSON普通数组
@RequestMapping("/arrayParamForJson")
@ResponseBody
public String arrayParamForJson(@RequestBody String[] likes){
    System.out.println(Arrays.toString(likes));
    return "{'module':'arrayParamForJson'}";
}

//JSON对象数组
@RequestMapping("/arrayPojoParamForJson")
@ResponseBody
public String arrayPojoParamForJson(@RequestBody List<User> list){
    System.out.println(list);
    return "{'module':'arrayPojoParamForJson'}";
}
```