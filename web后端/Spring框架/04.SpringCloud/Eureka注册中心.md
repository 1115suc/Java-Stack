# Eureka注册中心 🏢

假如我们的服务提供者user-service部署了多个实例，如图：

![image-20210713214925388](assets/image-20210713214925388.png)

大家思考几个问题：🤔

- order-service在发起远程调用的时候，该如何得知user-service实例的ip地址和端口？
- 有多个user-service实例地址，order-service调用时该如何选择？
- order-service如何得知某个user-service实例是否依然健康，是不是已经宕机？

## 🔧 Eureka的结构和作用

![image-20210713220104956](assets/image-20210713220104956.png)

### ❓ 问题1：order-service如何得知user-service实例地址？

获取地址信息的流程如下：📋

- user-service服务实例启动后，将自己的信息注册到eureka-server（Eureka服务端）。这个叫**服务注册**
- eureka-server保存服务名称到服务实例地址列表的映射关系
- order-service根据服务名称，拉取实例地址列表。这个叫**服务发现**或**服务拉取**

### ❓ 问题2：order-service如何从多个user-service实例中选择具体的实例？

- order-service从实例列表中利用负载均衡算法选中一个实例地址
- 向该实例地址发起远程调用

### ❓ 问题3：order-service如何得知某个user-service实例是否依然健康，是不是已经宕机？

- user-service会每隔一段时间（默认30秒）向eureka-server发起请求，报告自己状态，称为**心跳** 💓
- 当超过一定时间没有发送心跳时，eureka-server会认为微服务实例故障，将该实例从服务列表中剔除
- order-service拉取服务时，就能将故障实例排除了

> 💡 一个微服务，既可以是服务提供者，又可以是服务消费者，因此eureka将服务注册、服务发现等功能统一封装到了eureka-client端

## 🏗️ 搭建eureka-server

![image-20210713220509769](assets/image-20210713220509769.png)

### 1️⃣ 创建eureka-server服务

在cloud-demo父工程下，创建一个子模块：

![image-20210713220605881](assets/image-20210713220605881.png)

### 2️⃣ 引入eureka依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### 3️⃣ 编写启动类
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### 4️⃣ 编写配置文件
```yaml
server:
  port: 10086
spring:
  application:
    name: eureka-server # eureka-server的应用名称，会在eureka中显示
eureka:
  client:
    service-url: 
      defaultZone: http://127.0.0.1:10086/eureka
    register-with-eureka: false # false表示不向注册中心注册自己。
    fetch-registry: false # false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务  
```

### 5️⃣ 启动服务

启动EurekaServerApplication类，访问http://127.0.0.1:10086/，可以看到eureka-server的首页：

![image-20210713222157190](assets/image-20210713222157190.png)

## 📝 注册服务提供者

### 1️⃣ 引入依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2️⃣ 编写配置文件
```yaml
spring:
  application:
    name: userservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

### 3️⃣ 启动多个user-service实例

为了演示一个服务有多个实例的场景，我们添加一个SpringBoot的启动配置，再启动一个user-service。

首先，复制原来的user-service启动配置：

![1645512020157](assets/1645512020157.png)

![image-20210713222656562](assets/image-20210713222656562.png)

然后，在弹出的窗口中，填写信息：

![image-20210713222757702](assets/image-20210713222757702.png)

现在，SpringBoot窗口会出现两个user-service启动配置：

![image-20210713222841951](assets/image-20210713222841951.png)

不过，第一个是8081端口，第二个是8082端口。

启动两个user-service实例：

![image-20210713223041491](assets/image-20210713223041491.png)

查看eureka-server管理页面：

![image-20210713223150650](assets/image-20210713223150650.png)

## 🔍 服务发现

### 1️⃣ 引入依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2️⃣ 编写配置文件
服务发现也需要知道eureka地址，因此第二步与服务注册一致，都是配置eureka信息：

```yaml
spring:
  application:
    name: orderservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

### 3️⃣ 服务拉取和负载均衡
最后，我们要去eureka-server中拉取user-service服务的实例列表，并且实现负载均衡。

不过这些动作不用我们去做，只需要添加一些注解即可。

@LoadBalanced注解：

![image-20210713224049419](assets/image-20210713224049419.png)

### 4️⃣ 服务调用

```java
@RestController
@RequestMapping("order")
public class OrderController {
   @Autowired
   private OrderService orderService;
   
   @Autowired
   private RestTemplate restTemplate;
   
    @GetMapping("{orderId}")
    public Order queryOrderByUserId(@PathVariable("orderId") Long orderId) {
        // 根据id查询订单并返回
        Order order = orderService.queryOrderById(orderId);
        // 根据订单关联的用户id远程查询用户信息
        Long userId = order.getUserId();                  // 服务名称
        User user = restTemplate.getForObject("http://userservice/user/" + userId, User.class);
        order.setUser(user);
        return order;
    }
}
```

## 📚 核心概念总结

### 🎯 服务注册 (Service Registration)
- 服务提供者启动时向Eureka Server注册自己的信息
- 包括服务名称、IP地址、端口号等元数据

### 🔍 服务发现 (Service Discovery)
- 服务消费者从Eureka Server获取可用的服务实例列表
- 基于服务名称进行服务查找

### 💓 心跳机制 (Heartbeat)
- 服务提供者定期向Eureka Server发送心跳
- 默认30秒一次，用于证明服务实例健康状态

### ⚖️ 负载均衡 (Load Balancing)
- 通过@LoadBalanced注解实现客户端负载均衡
- 自动从多个服务实例中选择合适的实例进行调用

### 🛡️ 容错机制 (Fault Tolerance)
- Eureka Server检测到服务实例故障时自动剔除
- 确保服务消费者不会调用到不可用的实例

## 💡 最佳实践建议

1. **配置优化**：根据实际业务场景调整心跳间隔和超时时间
2. **多节点部署**：Eureka Server建议部署多个实例以实现高可用
3. **自我保护模式**：了解并合理配置Eureka的自我保护机制
4. **监控告警**：建立完善的监控体系，及时发现服务注册异常

