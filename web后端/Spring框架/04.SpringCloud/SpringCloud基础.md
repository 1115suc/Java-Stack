# 🚀 SpringCloud基础

## 📦 微服务架构演进

### 🏢 单体架构

所有功能都在一个应用中，打成一个包部署

- ✅ **优点：**
  - 简单、方便
- ❌ **缺点：**
  - 代码耦合度高
  - 维护和升级困难

![image-20210713202807818](assets/image-20210713202807818.png)

### 🌐 分布式架构

根据业务功能对系统做拆分，每个业务功能模块作为独立项目开发，称为一个服务。

- ✅ **优点：**
  - 代码解耦
  - 每个服务都可以独立部署、维护、升级，有利于服务升级和拓展
- ❌ **缺点：**
  - 服务调用关系错综复杂
  - 服务之间的通信成本高

![image-20210713203124797](assets/image-20210713203124797.png)

### 🔬 微服务架构

微服务的架构特征：

- 🎯 **单一职责**：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责
- 🛠️ **自治**：团队独立、技术独立、数据独立，独立部署和交付
- 🌍 **面向服务**：服务提供统一标准的接口，与语言和技术无关
- 🛡️ **隔离性强**：服务调用做好隔离、容错、降级，避免出现级联问题

![image-20210713203753373](assets/image-20210713203753373.png)

### 📚 SpringCloud

![image-20210713204155887](assets/image-20210713204155887.png)

### ✅ 总结

- **单体架构**：简单方便，高度耦合，扩展性差，适合小型项目。例如：学生管理系统

- **分布式架构**：松耦合，扩展性好，但架构复杂，难度大。适合大型互联网项目，例如：京东、淘宝

- **微服务**：一种良好的分布式架构方案

  ✅ **优点**：拆分粒度更小、服务更独立、耦合度更低

  ❌ **缺点**：架构非常复杂，运维、监控、部署难度提高

- **SpringCloud**是微服务架构的一站式解决方案，集成了各种优秀微服务功能组件

## 🔧 服务拆分和远程调用

### 📋 服务拆分原则

- 🚫 不同微服务，不要重复开发相同业务
- 💾 微服务数据独立，不要访问其它微服务的数据库
- 🔌 微服务可以将自己的业务暴露为接口，供其它微服务调用

![image-20210713210800950](assets/image-20210713210800950.png)

- 📊 实例

![image-20210713211009593](assets/image-20210713211009593.png)

## 🏪 Nacos 注册中心

### 📝 服务注册到 Nacos

Nacos 是SpringCloudAlibaba的组件，而SpringCloudAlibaba也遵循SpringCloud中定义的服务注册、服务发现规范。
因此使用Nacos和使用Eureka对于微服务来说，并没有太大区别。
主要差异在于：

- 依赖不同
- 服务地址不同

#### 1️⃣ 引入依赖

**父工程中引入依赖：**
```xml
<dependencyManagement>
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
</dependencyManagement>
```

**子工程中引入依赖：**
```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 2️⃣ 配置Nacos地址
```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
```

### 🏗️ 服务分级存储模型

一个**服务**可以有多个**实例**，例如我们的user-service，可以有:

- 127.0.0.1:8081
- 127.0.0.1:8082
- 127.0.0.1:8083

假如这些实例分布于全国各地的不同机房，例如：

- 127.0.0.1:8081，在上海机房
- 127.0.0.1:8082，在上海机房
- 127.0.0.1:8083，在杭州机房

Nacos就将同一机房内的实例 划分为一个**集群**。

也就是说，user-service是服务，一个服务可以包含多个集群，如杭州、上海，每个集群下可以有多个实例，形成分级模型，如图：

![image-20210713232522531](assets/image-20210713232522531.png)

微服务互相访问时，应该尽可能访问同集群实例，因为本地访问速度更快。当本集群内不可用时，才访问其它集群。例如：

![image-20210713232658928](assets/image-20210713232658928.png)

#### 1️⃣ 给服务配置集群

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

重启两个user-service实例后，我们可以在Nacos控制台看到下面结果：

![image-20210713232916215](assets/image-20210713232916215.png)

#### 2️⃣ 同集群优先的负载均衡
默认的`ZoneAvoidanceRule`并不能实现根据同集群优先来实现负载均衡。

因此Nacos中提供了一个`NacosRule`的实现，可以优先从同集群中挑选实例。

##### 🔧 配置集群信息

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

##### ⚖️ 修改负载均衡规则

```yaml
userservice:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则 
```

### ⚖️ 权重配置

实际部署中会出现这样的场景：

服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求。

但默认情况下`NacosRule`是同集群内随机挑选，不会考虑机器的性能问题。

因此，Nacos提供了权重配置来控制访问频率，权重越大则访问频率越高。

在Nacos控制台，找到user-service的实例列表，点击编辑，即可修改权重：

![image-20210713235133225](assets/image-20210713235133225.png)

在弹出的编辑窗口，修改权重：

![image-20210713235235219](assets/image-20210713235235219.png)

> ⚠️ **注意**：如果权重修改为0，则该实例永远不会被访问

### 🏢 环境隔离

Nacos提供了namespace来实现环境隔离功能。

- nacos中可以有多个namespace
- namespace下可以有group、service等
- 不同namespace之间相互隔离，例如不同namespace的服务互相不可见

#### 1️⃣ 创建namespace
默认情况下，所有service、data、group都在同一个namespace，名为public：

![image-20210714000414781](assets/image-20210714000414781.png)

我们可以点击页面新增按钮，添加一个namespace：

![image-20210714000440143](assets/image-20210714000440143.png)

然后，填写表单：

![image-20210714000505928](assets/image-20210714000505928.png)

就能在页面看到一个新的namespace：

![image-20210714000522913](assets/image-20210714000522913.png)

#### 2️⃣ 给微服务配置namespace

给微服务配置namespace只能通过修改配置来实现。

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ
        namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 # 命名空间，填ID
```

重启order-service后，访问控制台，可以看到下面的结果：

![image-20210714000830703](assets/image-20210714000830703.png)

![image-20210714000837140](assets/image-20210714000837140.png)

此时访问order-service，因为namespace不同，会导致找不到userservice，控制台会报错：

![image-20210714000941256](assets/image-20210714000941256.png)

### 🔄 Nacos与Eureka的区别

Nacos的服务实例分为两种l类型：

- 🔄 **临时实例**：如果实例宕机超过一定时间，会从服务列表剔除，默认的类型。
- 🏢 **非临时实例**：如果实例宕机，不会从服务列表剔除，也可以叫永久实例。

配置一个服务实例为永久实例：

```yaml
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false # 设置为非临时实例
```

Nacos和Eureka整体结构类似，服务注册、服务拉取、心跳等待，但是也存在一些差异：

![image-20210714001728017](assets/image-20210714001728017.png)

#### ✅ Nacos与eureka的共同点
- 🔄 都支持服务注册和服务拉取
- 💓 都支持服务提供者心跳方式做健康检测

#### 🔄 Nacos与Eureka的区别
- 🔍 Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式
- 🚫 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
- 📢 Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
- ⚖️ Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式

## 🎯 核心要点总结

- **微服务架构**是分布式架构的演进，具有更好的解耦性和扩展性
- **SpringCloud**提供了一站式微服务解决方案
- **Nacos**作为注册中心，支持服务分级、权重配置、环境隔离等高级功能
- **Nacos与Eureka**在功能特性上有所差异，Nacos提供了更多企业级特性

> 💡 掌握这些基础概念是深入学习SpringCloud微服务架构的重要前提！