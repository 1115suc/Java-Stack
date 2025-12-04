# 📦 RabbitMQ服务异步通信-高级篇

## 🔍 消息队列面临的实际问题
消息队列在使用过程中，面临着很多实际问题需要思考：

![image-20210718155003157](assets/image-20210718155003157.png)

---

## 🛡️ 消息可靠性保障
消息从发送，到消费者接收，会经历多个过程：

![image-20210718155059371](assets/image-20210718155059371.png)

### 消息丢失的常见原因
- **发送时丢失**：
  - 生产者发送的消息未送达exchange
  - 消息到达exchange后未到达queue
- **MQ宕机**，queue将消息丢失
- **consumer接收到消息后未消费就宕机**

### RabbitMQ解决方案
- ✅ 生产者确认机制
- ✅ mq持久化
- ✅ 消费者确认机制
- ✅ 失败重试机制

---

## 1️⃣ 生产者消息确认机制

### 📋 机制说明
RabbitMQ提供了publisher confirm机制来避免消息发送到MQ过程中丢失。这种机制必须给每个消息指定一个唯一ID。消息发送到MQ以后，会返回一个结果给发送者，表示消息是否处理成功。

**返回结果有两种方式：**
- **publisher-confirm**：发送者确认
  - 消息成功投递到交换机，返回ack
  - 消息未投递到交换机，返回nack
- **publisher-return**：发送者回执
  - 消息投递到交换机了，但是没有路由到队列。返回ACK，及路由失败原因。

![image-20210718160907166](assets/image-20210718160907166.png)

> ⚠️ 注意：
> ![image-20210718161707992](assets/image-20210718161707992.png)

### 🔧 1.1 修改配置
首先，修改publisher服务中的application.yml文件，添加下面的内容：

```yaml
spring:
  rabbitmq:
    publisher-confirm-type: correlated
    publisher-returns: true
    template:
      mandatory: true
```

**配置说明：**
- `publish-confirm-type`：开启publisher-confirm
  - `simple`：同步等待confirm结果，直到超时
  - `correlated`：异步回调，定义ConfirmCallback
- `publish-returns`：开启publish-return功能
- `template.mandatory`：定义消息路由失败时的策略

### 🔧 1.2 定义ReturnConfirm回调
每个RabbitTemplate只能配置一个ReturnCallback，因此需要在项目加载时配置：

```java
@Slf4j
@Configuration
public class CommonConfig implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        // 获取RabbitTemplate
        RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
        
        // 设置ReturnCallback
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            // 投递失败，记录日志
            log.info("消息发送失败，应答码{}，原因{}，交换机{}，路由键{},消息{}",
                    replyCode, replyText, exchange, routingKey, message.toString());
            // 如果有业务需要，可以重发消息
        });

        // 设置ConfirmCallback
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                if(ack){
                    // 3.1.ack，消息成功
                    log.debug("消息发送成功, ID:{}", correlationData.getId());
                }else{
                    // 3.2.nack，消息失败
                    log.error("消息发送失败, ID:{}, 原因{}",correlationData.getId(), cause);
                }
            }
        });
    }
    
    @Bean
    public DirectExchange simpleExchange(){
        // 三个参数：交换机名称、是否持久化、当没有queue与其绑定时是否自动删除
        return new DirectExchange("simple.direct", false, false);
    }
    
    @Bean
    public Queue simpleQueue(){
        return new Queue("simple.queue",false);
    }
    
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(simpleQueue()).to(simpleExchange()).with("simple");
    }
}
```

### 🧪 1.3 发送消息测试
在publisher服务的`course.mq.spring.SpringAmqpTest`类中，定义一个单元测试方法：

```java
public void testSendMessage2SimpleQueue() throws InterruptedException {
    // 1.消息体
    String message = "hello, spring amqp!";
    // 2.全局唯一的消息ID，需要封装到CorrelationData中
    CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
    // 4.发送消息
    rabbitTemplate.convertAndSend("task.direct", "task", message, correlationData);
    // 休眠一会儿，等待ack回执
    Thread.sleep(2000);
}
```

#### 📊 测试结果对比

| 测试场景 | 交换机 | 路由键 | 确认回调 | Return回调 |
|---------|--------|--------|----------|------------|
| **不存在的交换机** | task.direct | task | ❌ false | ❌ 未触发 |
| **存在交换机，不存在路由** | simple.direct | task | ✅ true | ✅ 触发 |
| **正确配置** | simple.direct | simple | ✅ true | ❌ 未触发 |

> 💡 **结论：** 通过发送确认和消息返还机制可以确保消息一定能够投递到指定的队列中，如果消息没有投递成功或返还了，也可以通过定时重新投递的方式进行补偿。

---

## 2️⃣ 消息持久化机制

生产者确认可以确保消息投递到RabbitMQ的队列中，但是消息发送到RabbitMQ以后，如果突然宕机，也可能导致消息丢失。

要想确保消息在RabbitMQ中安全保存，必须开启消息持久化机制：

### 📦 2.1 交换机持久化
RabbitMQ中交换机默认是非持久化的，mq重启后就丢失。

```java
@Bean
public DirectExchange simpleExchange(){
    // 三个参数：交换机名称、是否持久化、当没有queue与其绑定时是否自动删除
    return new DirectExchange("simple.direct", true, false);
}
```

> ✅ 默认情况下，由SpringAMQP声明的交换机都是持久化的。

### 📦 2.2 队列持久化
RabbitMQ中队列如果设置成非持久化的，mq重启后就丢失。

```java
@Bean
public Queue simpleQueue(){
    return new Queue("simple.queue",true);
}
```

> ✅ 默认情况下，由SpringAMQP声明的队列都是持久化的。

### 📦 2.3 消息持久化
利用SpringAMQP发送消息时，可以设置消息的属性（MessageProperties），指定delivery-mode：

- **1**：非持久化
- **2**：持久化

```java
@Test
public void testSendMessage2SimpleQueue() throws InterruptedException {
    String routingKey = "simple";
    String message = "hello, spring amqp!";
    // 自定义数据
    CorrelationData data = new CorrelationData(UUID.randomUUID().toString());
    // 发送消息
    rabbitTemplate.convertAndSend("simple.direct", routingKey, message, new MessagePostProcessor() {
        // 后置处理消息
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            // 设置消息的持久化方式
            message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.NON_PERSISTENT);
            return message;
        }
    },data);
}
```

> ✅ 默认情况下，SpringAMQP发出的任何消息都是持久化的，不用特意指定。

---

## 3️⃣ 消费者消息确认机制

RabbitMQ是**阅后即焚**机制，RabbitMQ确认消息被消费者消费后会立刻删除。

### 🎯 三种确认模式对比

| 模式 | 特点 | 可靠性 |
|-----|------|--------|
| **none** | 关闭ack，MQ假定消费者获取消息后会成功处理 | ❌ 不可靠 |
| **auto** | 自动ack，由spring监测listener代码是否出现异常 | ✅ 推荐 |
| **manual** | 手动ack，需要在业务代码结束后调用api发送ack | ⚠️ 需自行控制 |

### 🧪 3.1 演示none模式
```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: none # 关闭ack
```

```java
@RabbitListener(queues = "simple.queue")
public void listenSimpleQueue(String msg) {
    log.info("消费者接收到simple.queue的消息：【{}】", msg);
    // 模拟异常
    System.out.println(1 / 0);
    log.debug("消息处理完成！");
}
```

> ⚠️ 测试可以发现，当消息处理抛异常时，消息依然被RabbitMQ删除了。

### 🧪 3.2 演示auto模式
```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto # 自动ack
```

> ✅ 抛出异常后，Spring会自动返回nack，消息恢复至Ready状态，并且没有被RabbitMQ删除。

---

## 4️⃣ 消费失败重试机制

当消费者出现异常后，消息会不断requeue（重入队）到队列，再重新发送给消费者，然后再次异常，再次requeue，无限循环。

### 🔄 4.1 本地重试配置
```yaml
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true # 开启消费者失败重试
          initial-interval: 1000ms # 初识的失败等待时长为1秒
          multiplier: 1 # 失败的等待时长倍数
          max-attempts: 3 # 最大重试次数
          stateless: true # true无状态；false有状态
```

> ✅ 开启本地重试时，消息处理过程中抛出异常，不会requeue到队列，而是在消费者本地重试。
> 重试达到最大次数后，Spring会返回ack，消息会被丢弃。

### 🎯 4.2 失败策略选择

| 策略 | 处理方式 |
|-----|----------|
| **RejectAndDontRequeueRecoverer** | 重试耗尽后，直接reject，丢弃消息（默认） |
| **ImmediateRequeueMessageRecoverer** | 重试耗尽后，返回nack，消息重新入队 |
| **RepublishMessageRecoverer** | 重试耗尽后，将失败消息投递到指定的交换机 |

### 🚀 推荐方案：RepublishMessageRecoverer

```java
@Configuration
public class ErrorMessageConfig {
    @Bean
    public DirectExchange errorMessageExchange(){
        return new DirectExchange("error.direct");
    }
    
    @Bean
    public Queue errorQueue(){
        return new Queue("error.queue", true);
    }
    
    @Bean
    public Binding errorBinding(Queue errorQueue, DirectExchange errorMessageExchange){
        return BindingBuilder.bind(errorQueue).to(errorMessageExchange).with("error");
    }

    @Bean
    public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate){
        return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
    }
}
```

---

## 🎯 消息可靠性总结

如何确保RabbitMQ消息的可靠性？

1. ✅ **开启生产者确认机制**，确保生产者的消息能到达队列
2. ✅ **开启持久化功能**，确保消息未消费前在队列中不会丢失
3. ✅ **开启消费者确认机制为auto**，由spring确认消息处理成功后完成ack
4. ✅ **开启消费者失败重试机制**，并设置MessageRecoverer，多次重试失败后将消息投递到异常交换机，交由人工处理

---

# 💀 死信交换机

## 💡 什么是死信交换机

### 死信的三种情况：
- 消费者使用basic.reject或 basic.nack声明消费失败，并且消息的requeue参数设置为false
- 消息是一个过期消息，超时无人消费
- 要投递的队列消息满了，无法投递

### 死信交换机的作用：
如果队列绑定了死信交换机，死信会投递到死信交换机，可以收集所有消费者处理失败的消息，交由人工处理。

![image-20210718174328383](assets/image-20210718174328383.png)

---

## 🛠️ 死信交换机配置示例

```java
@Bean
public Queue simpleQueue(){
    return QueueBuilder.durable("simple.queue")
            .deadLetterExchange("dl.direct") // 指定死信交换机
            .build();
}

// 声明死信交换机 dl.direct
@Bean
public DirectExchange dlExchange(){
    return new DirectExchange("dl.direct", true, false);
}

// 声明存储死信的队列 dl.queue
@Bean
public Queue dlQueue(){
    return new Queue("dl.queue", true);
}

// 将死信队列 与 死信交换机绑定
@Bean
public Binding dlBinding(){
    return BindingBuilder.bind(dlQueue()).to(dlExchange()).with("dl");
}
```

> ⚠️ **注意：** 如果之前已经声明过simple.queue队列，需要删除后重新创建。

---

# ⏰ TTL（消息超时）

一个队列中的消息如果超时未消费，则会变为死信，超时分为两种情况：

1. **消息所在的队列设置了超时时间**
2. **消息本身设置了超时时间**

![image-20210718182643311](assets/image-20210718182643311.png)

---

# 📦 惰性队列

## 🚀 惰性队列的特点

### 优点：
- ✅ 基于磁盘存储，消息上限高
- ✅ 没有间歇性的page-out，性能比较稳定

### 缺点：
- ⚠️ 基于磁盘存储，消息时效性会降低
- ⚠️ 性能受限于磁盘的IO

## 🛠️ 惰性队列配置

```java
// 惰性队列
@Bean
public Queue lazyQueue(){
    return QueueBuilder.durable("lazy.queue")
            .lazy()
            .build();
}

// 普通队列
@Bean
public Queue normalQueue(){
    return QueueBuilder.durable("normal.queue")
            .build();
}
```

---

# 🏗️ MQ集群

## 🎯 集群分类

RabbitMQ的集群有两种模式：

| 集群类型 | 特点 | 适用场景 |
|---------|------|----------|
| **普通集群** | 分布式集群，队列分散到各个节点 | 提高并发能力 |
| **镜像集群** | 主从集群，有主从备份功能 | 提高数据可用性 |
| **仲裁队列** | 3.8版本新功能，基于Raft协议 | 强一致性 |

## 🚀 仲裁队列配置

```java
@Bean
public Queue quorumQueue() {
    return QueueBuilder
        .durable("quorum.queue") // 持久化
        .quorum() // 仲裁队列
        .build();
}
```

## 🔗 SpringAMQP连接MQ集群

```yaml
spring:
  rabbitmq:
    addresses: 192.168.150.105:8071, 192.168.150.105:8072, 192.168.150.105:8073
    username: itcast
    password: 123321
    virtual-host: /
```

---
        