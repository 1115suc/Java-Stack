# 🌟 Spring AOP 学习笔记

## 📚 AOP 基础概念

### 🔍 AOP 简介
- **AOP**(Aspect Oriented Programming)面向切面编程，一种编程范式，指导开发者如何组织程序结构
    - **OOP**(Object Oriented Programming)面向对象编程
- **作用**：在不惊动原始设计的基础上为其进行功能增强。简单的说就是在不改变方法源代码的基础上对方法进行功能增强。
- **Spring理念**：无入侵式/无侵入式

### 🎯 AOP 核心概念
- **连接点**（JoinPoint）：正在执行的方法，例如：`update()`、`delete()`、`select()`等都是连接点。
- **切入点**（Pointcut）：匹配连接点的式子
    - 在SpringAOP中，一个切入点可以只描述一个具体方法，也可以匹配多个方法
        - 一个具体方法：`course.dao`包下的`BookDao`接口中的无形参无返回值的save方法
        - 匹配多个方法：所有的save方法，所有的get开头的方法，所有以Dao结尾的接口中的任意方法，所有带有一个参数的方法
- **通知**（Advice）：在切入点前后执行的操作，也就是增强的共性功能
    - 在SpringAOP中，功能最终以方法的形式呈现
- **通知类**：通知方法所在的类叫做通知类
- **切面**（Aspect）：描述通知与切入点的对应关系，也就是哪些通知方法对应哪些切入点方法

### ⚡ AOP 核心注解
- `@Aspect`：定义切面类
- `@Pointcut`：定义切入点表达式
- `@Before`：前置通知
- `@After`：后置通知
- `@AfterReturning`：返回通知
- `@AfterThrowing`：异常通知
- `@Around`：环绕通知
- `@Order`：设置优先级
- `@EnableAspectJAutoProxy`：开启AOP功能

---

## 🚀 AOP 使用指南

### 1️⃣ 引入依赖
```xml
<dependencies>
    <!--spring核心依赖，会将spring-aop传递进来-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    
    <!--切入点表达式依赖，目的是找到切入点方法，也就是找到要增强的方法-->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.4</version>
    </dependency>
</dependencies>
```


### 2️⃣ 定义dao接口与实现类
```java
public interface BookDao {
    public void save();
    public void update();
}

@Repository
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println(System.currentTimeMillis());
        System.out.println("book dao save ...");
    }
    public void update(){
        System.out.println("book dao update ...");
    }
}
```


### 3️⃣ 定义通知类，制作通知方法
```java
//通知类必须配置成Spring管理的bean
@Component
public class MyAdvice {
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```


### 4️⃣ 定义切入点表达式、配置切面(绑定切入点与通知关系)
```java
@Component
//设置当前类为切面类类
@Aspect
public class MyAdvice {
    //设置切入点，@Pointcut注解要求配置在方法上方
    @Pointcut("execution(void course.dao.BookDao.update())")
    private void pt(){}

    //设置在切入点pt()的前面运行当前操作(前置通知)
    @Before("pt()")
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```


### 5️⃣ 在配置类中进行Spring注解包扫描和开启AOP功能
```java
@Configuration
@ComponentScan("course")
//开启注解开发AOP功能
@EnableAspectJAutoProxy
public class SpringConfig {
}
```


---

## ⚙️ AOP 工作机制

### 🔄 AOP工作流程

1. Spring容器启动
2. 读取所有切面配置中的切入点
3. 初始化bean，判定bean对应的类中的方法是否匹配到任意切入点
    - 匹配失败，创建原始对象
    - 匹配成功，创建原始对象（目标对象）的代理对象
4. 获取bean执行方法
    - 获取的bean是原始对象时，调用方法并执行，完成操作
    - 获取的bean是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作

### 🎯 AOP核心概念
- **目标对象**（Target）：被代理的对象，也叫原始对象，该对象中的方法没有任何功能增强。
- **代理对象**（Proxy）：代理后生成的对象，由Spring帮我们创建代理对象。

---

## 🔧 切入点表达式详解

### 📝 语法格式

- **切入点**：要进行增强的方法
- **切入点表达式**：要进行增强的方法的描述方式

#### 示例：
- 描述方式一：执行`course.dao`包下的`BookDao`接口中的无参数`update`方法
  ```java
  execution(void course.dao.BookDao.update())
  ```


- 描述方式二：执行`course.dao.impl`包下的`BookDaoImpl`类中的无参数`update`方法
  ```java
  execution(void course.dao.impl.BookDaoImpl.update())
  ```


- **切入点表达式标准格式**：动作关键字(访问修饰符  返回值  包名.类/接口名.方法名(参数) 异常名）
  ```java
  execution(public User com.itheima.service.UserService.findById(int))
  ```


### 🎯 通配符

> 目的：可以使用通配符描述切入点，快速描述。

- **\*** ：单个独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现
  > 匹配`course`包下的任意包中的`UserService`类或接口中所有find开头的带有一个参数的方法
  ```java
  execution(public * course.*.UserService.find*(*))
  ```


- **..** ：多个连续的任意符号，可以独立出现，常用于简化包名与参数的书写
  > 匹配`com`包下的任意包中的`UserService`类或接口中所有名称为`findById`的方法
  ```java
  execution(public User com..UserService.findById(..))
  ```


- **+**：专用于匹配子类类型
  ```java
  execution(* *..*Service+.*(..))
  ```


---

## 📢 AOP 通知类型

- **前置通知**`@Before`：在目标方法执行之前执行
- **后置通知**`@After`：在目标方法执行之后执行
- **返回通知**`@AfterReturning`：在目标方法正常返回之后执行
- **异常通知**`@AfterThrowing`：在目标方法抛出异常之后执行
- **环绕通知**`@Around`：在目标方法执行前后执行，可以决定目标方法执行还是不执行
    - 环绕通知方法形参必须是`ProceedingJoinPoint`，表示正在执行的连接点，使用该对象的`proceed()`方法表示对原始对象方法进行调用，返回值为原始对象方法的返回值。
    - 环绕通知方法的返回值建议写成`Object`类型，用于将原始对象方法的返回值进行返回，哪里使用代理对象就返回到哪里。
  ```java
  @Around("pt()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
      System.out.println("around before advice ...");
      Object ret = pjp.proceed();
      System.out.println("around after advice ...");
      return ret;
  }
  ```


---

## 📊 切入点数据获取

### 获取参数

> 说明：在前置通知和环绕通知中都可以获取到连接点方法的参数们

- `JoinPoint`对象描述了连接点方法的运行状态，可以获取到原始方法的调用参数
  ```java
  @Before("pt()")
  public void before(JoinPoint jp) {
      Object[] args = jp.getArgs(); //获取连接点方法的参数们
      System.out.println(Arrays.toString(args));
  }
  ```


- `ProccedJointPoint`是`JoinPoint`的子类
  ```java
  @Around("pt()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
      Object[] args = pjp.getArgs(); //获取连接点方法的参数们
      System.out.println(Arrays.toString(args));
      Object ret = pjp.proceed();
      return ret;
  }
  ```


### 获取返回值

> 说明：在返回后通知和环绕通知中都可以获取到连接点方法的返回值

- 返回后通知可以获取切入点方法返回值信息，使用形参可以接收对应的返回值
  ```java
  @AfterReturning(value = "pt()",returning = "ret")
  public void afterReturning(String ret) { //变量名要和returning="ret"的属性值一致
      System.out.println("afterReturning advice ..."+ret);
  }
  ```


- 环绕通知中可以手工书写对原始方法的调用，得到的结果即为原始方法的返回值
  ```java
  @Around("pt()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
      // 手动调用连接点方法，返回值就是连接点方法的返回值
      Object ret = pjp.proceed();
      return ret;
  }
  ```


### 获取异常

> 说明：在抛出异常后通知和环绕通知中都可以获取到连接点方法中出现的异常

- 抛出异常后通知可以获取切入点方法中出现的异常信息，使用形参可以接收对应的异常对象
  ```java
  @AfterThrowing(value = "pt()",throwing = "t")
  public void afterThrowing(Throwable t) {//变量名要和throwing = "t"的属性值一致
      System.out.println("afterThrowing advice ..."+ t);
  }
  ```


- 抛出异常后通知可以获取切入点方法运行的异常信息，使用形参可以接收运行时抛出的异常对象
  ```java
  @Around("pt()")
  public Object around(ProceedingJoinPoint pjp)  {
      Object ret = null;
      //此处需要try...catch处理，catch中捕获到的异常就是连接点方法中抛出的异常
      try {
          ret = pjp.proceed();
      } catch (Throwable t) {
          t.printStackTrace();
      }
      return ret;
  }
  ```
