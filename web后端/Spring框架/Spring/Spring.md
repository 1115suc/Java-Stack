# 🌟 Spring 学习笔记

## 📦 Spring容器

### 🏗️ 创建容器

- **方式一**：类路径加载配置文件
  ```java
  ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
  ```


- **方式二**：文件路径加载配置文件
  ```java
  ApplicationContext ctx = new FileSystemXmlApplicationContext("D:\\applicationContext.xml");
  ```


- **加载多个配置文件**
  ```java
  ApplicationContext ctx = new ClassPathXmlApplicationContext("bean1.xml", "bean2.xml");
  ```


### 🎯 获取bean对象

- **方式一**：使用bean名称获取
  > 弊端：需要自己强制类型转换
  ```java
  BookDao bookDao = (BookDao) ctx.getBean("bookDao");
  ```


- **方式二**：使用bean名称获取并指定类型 ✅ *推荐使用*
  ```java
  BookDao bookDao = ctx.getBean("bookDao", BookDao.class);
  ```


- **方式三**：使用bean类型获取
  > 弊端：如果IOC容器中同类型的Bean对象有多个，此处获取会报错
  ```java
  BookDao bookDao = ctx.getBean(BookDao.class);
  ```


### 🧱 容器类层次结构

![image-20210730102842030.png](img/image-20210730102842030.png)

### ⚙️ `BeanFactory`

- 类路径加载配置文件
  ```java
  Resource resources = new ClassPathResource("applicationContext.xml");
  BeanFactory bf = new XmlBeanFactory(resources);
  BookDao bookDao = bf.getBean("bookDao", BookDao.class);
  bookDao.save();
  ```


- `BeanFactory`创建完毕后，所有的Bean均为**延迟加载**，也就是说我们调用`getBean()`方法获取Bean对象时才创建Bean对象并返回给我们

### 📋 Spring核心容器总结

#### 🧰 容器相关

- `BeanFactory`是IoC容器的顶层接口，初始化`BeanFactory`对象时，加载的bean延迟加载
- `ApplicationContext`接口是Spring容器的核心接口，初始化时bean立即加载
- `ApplicationContext`接口提供基础的bean操作相关方法，通过其他接口扩展其功能
- `ApplicationContext`接口常用初始化类
    - `ClassPathXmlApplicationContext`(常用) 🌟
    - `FileSystemXmlApplicationContext`

#### 🧩 Bean对象相关
![image-20210730103438742.png](img/image-20210730103438742.png)

#### 💉 依赖注入相关
![image-20210730103701525.png](img/image-20210730103701525.png)

---

## 🎯 Spring注解开发

### 🔑 核心注解

- `@Component`：表示当前类是一个组件类，Spring会自动扫描并创建对象
    - `@Controller`：表示当前类是一个控制器类
    - `@Service`：表示当前类是一个业务逻辑类
    - `@Repository`：表示当前类是一个数据访问类
- `@Configuration`：表示当前类是一个配置类
    - `@ConfigurationProperties`: 表示将当前类中的属性值和配置文件中的属性值进行绑定
    - `@EnableConfigurationProperties`: 表示将当前类中的属性值和配置文件中的属性值进行绑定
- `@Bean`：表示当前方法创建一个bean对象，并交给Spring管理
- `@Import`：表示导入其他配置类
- `@Autowired`：表示自动注入
- `@Qualifier`：表示指定注入的bean名称
- `@Value`：表示注入普通属性
- `@PropertySource`: 表示加载指定的配置文件
- `@Scope`：表示指定bean的作用范围
- `@PostConstruct`：表示当前方法在bean对象初始化之后执行
- `@PreDestroy`：表示当前方法在bean对象销毁之前执行
- `@ComponentScan`: 表示指定要扫描的包

> ⚠️ 注意：`@PostConstruct`和`@PreDestroy`注解是jdk中提供的注解，从jdk9开始，jdk中的`javax.annotation`包被移除了，也就是说这两个注解就用不了了，可以额外导入一下依赖解决这个问题。

```xml
<dependency>
  <groupId>javax.annotation</groupId>
  <artifactId>javax.annotation-api</artifactId>
  <version>1.3.2</version>
</dependency>
```


### 🚀 基本使用

1. 在`applicationContext.xml`中开启Spring注解包扫描
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
   	 <!--扫描course包及其子包下的类中注解-->
       <context:component-scan base-package="course"/>
   </beans>
   ```


2. 在类上使用`@Component`注解定义Bean
   ```java
   @Component("bookDao")
   public class BookDaoImpl implements BookDao {
       public void save() {
           System.out.println("book dao save ...");
       }
   }
   
   @Component
   public class BookServiceImpl implements BookService {
       private BookDao bookDao;
   
       public void setBookDao(BookDao bookDao) {
           this.bookDao = bookDao;
       }
   
       public void save() {
           System.out.println("book service save ...");
           bookDao.save();
       }
   }
   ```


> 如果`@Component`注解没有使用参数指定Bean的名称，那么类名首字母小写就是Bean在IOC容器中的默认名称。例如：`BookServiceImpl`对象在IOC容器中的名称是`bookServiceImpl`。

3. 在测试类中获取Bean对象

   ```java
   public class AppForAnnotation {
       public static void main(String[] args) {
           ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
           BookDao bookDao = (BookDao) ctx.getBean("bookDao");
           System.out.println(bookDao);
           //按类型获取bean
           BookService bookService = ctx.getBean(BookService.class);
           System.out.println(bookService);
       }
   }
   ```


### 🌈 纯注解开发模式

#### `@Qualifier`和`@Autowired`

- `@Autowired`：注入引用类型，自动装配模式，默认按类型装配
- `@Qualifier`：自动装配bean时按bean名称装配

> `@Qualifier`注解无法单独使用，必须配合`@Autowired`注解使用

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    @Qualifier("bookDao")
    private BookDao bookDao;

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```


#### `@Value`和`@PropertySource`

- `@Value`：注入普通类型
- `@PropertySource`：加载指定的配置文件，注解只能加载指定的配置文件，不能加载多个配置文件

```java
@Configuration
@ComponentScan("course")
@PropertySource({"classpath:jdbc.properties"}) //{}可以省略不写
public class SpringConfig {
}

@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    @Value("${name}")
    private String name;
}
```


---

## 📚 Spring注解开发补充知识

### 🔍 更多实用注解

1. **条件注解**
    - `@Conditional`：根据条件决定是否创建Bean
    - `@Profile`：根据不同环境激活不同的Bean配置

2. **事件监听注解**
    - `@EventListener`：监听Spring应用事件
    - `@Async`：异步执行方法（需配合`@EnableAsync`）

3. **Web相关注解**
    - `@RequestMapping`系列：处理HTTP请求映射
    - `@ResponseBody`：将返回值直接写入HTTP响应体
    - `@RestController`：组合了`@Controller`和`@ResponseBody`

4. **事务注解**
    - `@Transactional`：声明式事务管理

### 🎯 最佳实践建议

- **命名规范**：使用有意义的Bean名称，避免歧义
- **依赖注入**：优先使用构造器注入保证不可变性和必需依赖
- **配置分离**：将不同模块的配置分离到不同的配置类中
- **组件扫描**：合理设置扫描路径，避免不必要的类被扫描
- **属性绑定**：使用`@ConfigurationProperties`替代多个`@Value`注解

这些额外的知识可以帮助你更深入地理解和使用Spring框架的各种功能！🌟