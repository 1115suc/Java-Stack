# Spring框架

## Spring 框架介绍

- IOC
  - 使用对象时，由主动new产生对象转换为由外部提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转
  - Spring提供了一个容器，称为IOC容器，用来充当IOC思想中的“外部”
  - IOC容器负责对象的创建、初始化等一系列工作，被创建或被管理的对象在IoC容器中统称为Bean
- DI
  - 在容器中建立bean与bean之间的依赖关系的整个过程，称为依赖注入
- AOP
- 事务处理

### Spring Framework系统架构图
- Spring Framework是Spring生态圈中最基础的项目，是其他项目的根基
![image-20210729172153796.png](img/image-20210729172153796.png)
![image-20210729172352627.png](img/image-20210729172352627.png)

## IOC和DI入门案例

### IOC实例实现步骤

- 1.导入Spring坐标
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
</dependencies>
```

- 2.定义Spring管理的类（接口）
```java
public interface BookDao {
    public void save();
}

public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
}
```

- 3.创建Spring配置文件，配置对应类作为Spring管理的bean对象
    - 定义`applicationContext.xml`配置文件并配置`BookServiceImpl`
````java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookService" class="course.service.impl.BookServiceImpl"></bean>

</beans>
````

- 4.初始化IOC容器（Spring核心容器/Spring容器），通过容器获取bean对象
```java
public class App {
    public static void main(String[] args) {
        //1.创建IoC容器对象，加载spring核心配置文件
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2 从IOC容器中获取Bean对象(BookService对象)
        BookService bookService= (BookService)ctx.getBean("bookService");
        //3 调用Bean对象(BookService对象)的方法
        bookService.save();
    }
}
```

### DI实例实现步骤

- 0.环境代码

```java
public interface BookService {
    public void save();
}

public class BookServiceImpl implements BookService {
    private BookDao bookDao = new BookDaoImpl();
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```


- 1.删除使用new的形式创建对象的代码

```java
public class BookServiceImpl implements BookService {
    private BookDao bookDao;  //【第一步】删除使用new的形式创建对象的代码
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

- 2.提供依赖对象对应的setter方法
```java
public class BookServiceImpl implements BookService {
    private BookDao bookDao;
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
    //【第二步】提供依赖对象对应的setter方法
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
}
```

- 3.配置service与dao之间的关系
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
		bean标签：表示配置bean
    	id属性：表示给bean起名字
    	class属性：表示给bean定义类型
	-->
    <bean id="bookDao" class="course.dao.impl.BookDaoImpl"/>

    <bean id="bookService" class="course.service.impl.BookServiceImpl">
        <!--配置server与dao的关系
			property标签：表示配置当前bean的属性
        	name属性：表示配置哪一个具体的属性
        	ref属性：表示参照哪一个bean
		-->
        <property name="bookDao" ref="bookDao"/>
    </bean>
</beans>
```

### Bean的实例化四种方式

#### 构造方法实例化

- 1.`BookDaoImpl`实现类
```java
public class BookDaoImpl implements BookDao {
    public BookDaoImpl() {
        System.out.println("book dao constructor is running ....");
    }
    public void save() {
        System.out.println("book dao save ...");
    }
}
```

- 2.`applicationContext.xml`配置
```xml
<!--方式一：构造方法实例化bean-->
<bean id="bookDao" class="course.dao.impl.BookDaoImpl"/>
```

- 3.`AppForInstanceBook`测试类
```java
public class AppForInstanceBook {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();
    }
}
```

- 运行结果

![image-20210729194137151.png](img/image-20210729194137151.png)

#### 静态工厂实例化

- 1.`OrderDao`接口和`OrderDaoImpl`实现类
```java
public interface OrderDao {
    public void save();
}
public class OrderDaoImpl implements OrderDao {
    public void save() {
        System.out.println("order dao save ...");
    }
}
```

- 2.`OrderDaoFatory`工厂类
```java
public class OrderDaoFactory {
    public static OrderDao getOrderDao(){
        System.out.println("factory setup....");
        return new OrderDaoImpl();
    }
}
```

- 3.`applicationContext.xml`配置  
```xml
<!--方式二：使用静态工厂实例化bean-->
<bean id="orderDao" class="course.factory.OrderDaoFactory" factory-method="getOrderDao"/>
```

![image-20210729195248948.png](img/image-20210729195248948.png)

- 4.`AppForInstanceOrder`测试类
```java
public class AppForInstanceOrder {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        OrderDao orderDao = (OrderDao) ctx.getBean("orderDao");
        orderDao.save();
    }
}
```

- 运行结果
![image-20210729195009198.png](img/image-20210729195009198.png)


#### 实例工厂实例化

- 1.`UserDao`接口和`UserDaoImpl`实现类
```java
public interface UserDao {
    public void save();
}
public class UserDaoImpl implements UserDao {
    public void save() {
        System.out.println("user dao save ...");
    }
}
```

- 2.`UserDaoFactory`工厂类
```java
//实例工厂创建对象
public class UserDaoFactory {
    public UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```
- 3.`applicationContext.xml`配置
```xml
<!--方式三：使用实例工厂实例化bean-->
<bean id="userFactory" class="course.factory.UserDaoFactory"/>
<bean id="userDao" factory-method="getUserDao" factory-bean="userFactory"/>
```

![image-20210729200203249.png](img/image-20210729200203249.png)

- 4.`AppForInstanceUser`测试类
```java
public class AppForInstanceUser {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = (UserDao) ctx.getBean("userDao");
        userDao.save();
    }
}
```

- 运行结果
![image-20210729200240820.png](img/image-20210729200240820.png)


#### 实现`FactoryBean<T>`方式实例化

- 1.定义`UserDaoFactoryBean`实现`FactoryBean<UserDao>`接口
```java
//FactoryBean创建对象
public class UserDaoFactoryBean implements FactoryBean<UserDao> {
    //代替原始实例工厂中创建对象的方法
    public UserDao getObject() throws Exception {
        return new UserDaoImpl();
    }

    public Class<?> getObjectType() {
        return UserDao.class;
    }
}
```

- 2.`applicationContext.xml`配置
```xml
<!--方式四：使用FactoryBean实例化bean-->
<bean id="userDao" class="course.factory.UserDaoFactoryBean"/>
```

### 依赖注入

#### setter方法注入

- 引用类型
![1671182583315.png](img/1671182583315.png)

- 简单类型
![image-20210729203728173.png](img/image-20210729203728173.png)

#### 构造方法注入

- 引用类型
![image-20210729203859855.png](img/image-20210729203859855.png)

- 简单类型
![image-20210729204006542.png](img/image-20210729204006542.png)

- 参数适配
![image-20210729204117697.png](img/image-20210729204117697.png)

#### 依赖注入方式选择

1. 强制依赖使用构造器进行，使用setter注入有概率不进行注入导致null对象出现
2. 可选依赖使用setter注入进行，灵活性强
3. Spring框架倡导使用构造器，第三方框架内部大多数采用构造器注入的形式进行数据初始化，相对严谨
4. 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
5. 实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
6. 自己开发的模块推荐使用setter注入
