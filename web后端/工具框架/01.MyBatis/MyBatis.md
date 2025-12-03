# 📚 MyBatis 笔记

## 🔍 MyBatis 核心概念详解

### 🎯 什么是 MyBatis
MyBatis 是一个优秀的持久层框架，它简化了数据库操作：
- **消除冗余代码**：避免了几乎所有的 `JDBC` 代码和手动设置参数以及获取结果集的过程
- **灵活配置**：可以使用简单的 `XML` 或注解来配置和映射原生信息
- **对象关系映射**：将接口和 Java 的实体类映射成数据库中的记录
- **开源演进**：本是 apache 的一个开源项目 `ibatis`，2010 年迁移到 google code 并改名为 MyBatis

### 💾 持久化概念深入理解
**持久化**是将程序数据在持久状态和瞬时状态间转换的机制：

**核心特点**：
- 把数据（如内存中的对象）保存到可永久保存的存储设备中（如磁盘）
- `JDBC` 和文件 `IO` 都是持久化机制
- **必要性**：
  - 内存断电后数据会丢失，关键业务数据必须持久化
  - 内存价格昂贵，容量有限，需要将不常用数据持久化到外存

![1566614801843.png](img/1566614801843.png)

### 🏗️ 持久层架构设计
**持久层**是系统架构中的重要组成部分：
- 完成持久化工作的代码块 → `dao` 层（Data Access Object 数据访问对象）
- 企业级应用中，数据持久化大多通过关系数据库完成
- 独立的逻辑层面，专注于数据持久化逻辑实现，与业务逻辑分离

---

## 🚀 SpringBoot 整合 MyBatis 详细配置

### 📦 1. Maven 依赖配置详解
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```


**依赖说明**：
- `mybatis-spring-boot-starter`：Spring Boot 集成 MyBatis 的官方 starter
- 自动配置 `SqlSessionFactory`、`SqlSessionTemplate` 等核心组件
- 提供基于注解和 XML 配置的两种使用方式

### ⚙️ 2. 数据库连接详细配置
```yaml
spring:
  datasource:
    url: jdbc:mysql://192.168.88.128:3306/Notification?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true&useSSL=false
    username: root
    password: 24364726
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      pool-name: HikariCPDatasource
      minimum-idle: 5
      idle-timeout: 180000
      maximum-pool-size: 10
      auto-commit: true
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1

# MyBatis 配置
mybatis:
  type-aliases-package: course.common.pojo
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true # 开启驼峰映射
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```


**配置项详解**：
- `type-aliases-package`：指定别名包扫描路径，简化 XML 中的类型声明
- `mapper-locations`：指定 Mapper XML 文件的位置
- `map-underscore-to-camel-case`：开启数据库下划线命名到 Java 驼峰命名的自动转换
- `log-impl`：指定 MyBatis 日志实现类

### 🧱 3. 实体类创建与 Lombok 使用
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
</dependency>
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {
    private Integer id;
    private String departmentName;
}
```


**Lombok 注解说明**：
- `@Data`：自动生成 getter、setter、toString、equals、hashCode 方法
- `@NoArgsConstructor`：生成无参构造函数
- `@AllArgsConstructor`：生成全参构造函数

### 🎯 4. Mapper 接口定义规范
```java
@Mapper 
@Repository
public interface DepartmentMapper {
    // 获取所有部门信息
    List<Department> getDepartments();
    // 通过id获得部门
    Department getDepartment(Integer id);
}
```


**注解说明**：
- `@Mapper`：标记该接口为 MyBatis Mapper 接口，由 MyBatis 自动扫描注册
- `@Repository`：Spring 注解，标记为数据访问层组件，便于异常转换

### 📄 5. Mapper XML 映射文件详解
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.mapper.DepartmentMapper">
    <select id="getDepartments" resultType="Department">
       select * from department;
    </select>
    
    <select id="getDepartment" resultType="Department" parameterType="int">
       select * from department where id = #{id};
    </select>
</mapper>
```


**XML 元素说明**：
- `namespace`：指定对应的 Mapper 接口全限定名
- `id`：对应接口中的方法名
- `resultType`：指定返回结果的类型
- `parameterType`：指定传入参数的类型
- `#{id}`：参数占位符，防止 SQL 注入

### 📁 6. Maven 资源过滤配置详解
```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
    </resource>
</resources>
```


**配置目的**：
- Maven 默认只处理 `src/main/resources` 下的资源文件
- 此配置使得 `src/main/java` 下的 XML 文件也能被打包到最终的 JAR 中
- 解决 Mapper XML 文件放在 Java 包中无法被识别的问题

---

## 🏷️ MyBatis 核心标签深度解析

### 🔍 Select 标签详细说明
`<select>` 标签用于执行查询操作，是最常用的 MyBatis 标签之一。

#### 基本语法结构
```xml
<select id="方法名" parameterType="参数类型" resultType="返回类型">
    SQL 查询语句
</select>
```


#### 主要属性详解
- `id`：命名空间中的唯一标识符，必须与 `Mapper` 接口中对应方法名完全一致
- `parameterType`：传入参数的类型，可以是基本类型、POJO 或 Map
- `resultType`：返回结果的类型，MyBatis 会自动将查询结果映射到对应的 Java 对象
- `resultMap`：当数据库字段名与 Java 属性名不一致时，引用外部定义的 `resultMap`

#### 实际示例
```xml
<!-- 查询单个用户 -->
<select id="getUserById" parameterType="int" resultType="User">
    SELECT id, name, age FROM user WHERE id = #{id}
</select>

<!-- 查询多个用户 -->
<select id="getAllUsers" resultType="User">
    SELECT id, name, age FROM user
</select>

<!-- 带参数查询 -->
<select id="findUsersByName" parameterType="string" resultType="User">
    SELECT id, name, age FROM user WHERE name LIKE CONCAT('%', #{name}, '%')
</select>
```


#### 参数占位符说明
- `#{id}`：预编译参数占位符，安全防 SQL 注入
- `${id}`：字符串替换，直接拼接 SQL（不推荐，有安全风险）

### ➕ Insert 标签关键属性详解
`<insert>` 标签用于执行插入操作，特别适用于需要获取自动生成主键的场景。

#### 基本语法
```xml
<insert id="方法名" parameterType="参数类型" useGeneratedKeys="true" keyProperty="主键属性">
    INSERT 语句
</insert>
```


#### 关键属性说明
- `useGeneratedKeys`：是否使用数据库自动生成的主键
  - `true`：使用自动生成主键（如 MySQL 的 AUTO_INCREMENT）
  - `false`：不使用自动生成主键
- `keyProperty`：指定主键字段对应的 Java 对象属性名
- `keyColumn`：指定主键在数据库中的列名（可选）

#### 实际示例
```xml
<!-- 插入用户并获取自动生成的主键 -->
<insert id="insertUser" parameterType="User" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user(name, age) VALUES(#{name}, #{age})
</insert>

<!-- 插入时不获取主键 -->
<insert id="insertUserSimple" parameterType="User">
    INSERT INTO user(name, age) VALUES(#{name}, #{age})
</insert>
```


### ✏️ Update 标签使用示例
`<update>` 标签用于执行更新操作。

#### 基本语法
```xml
<update id="方法名" parameterType="参数类型">
    UPDATE 语句
</update>
```


#### 实际示例
```xml
<!-- 更新用户信息 -->
<update id="updateUser" parameterType="User">
    UPDATE user SET name=#{name}, age=#{age} WHERE id=#{id}
</update>

<!-- 部分更新 -->
<update id="updateUserSelective" parameterType="User">
    UPDATE user 
    <set>
        <if test="name != null">name=#{name},</if>
        <if test="age != null">age=#{age},</if>
    </set>
    WHERE id=#{id}
</update>
```


### ❌ Delete 标签基本用法
`<delete>` 标签用于执行删除操作。

#### 基本语法
```xml
<delete id="方法名" parameterType="参数类型">
    DELETE 语句
</delete>
```


#### 实际示例
```xml
<!-- 根据ID删除用户 -->
<delete id="deleteUser" parameterType="int">
    DELETE FROM user WHERE id = #{id}
</delete>

<!-- 批量删除 -->
<delete id="deleteUsers" parameterType="list">
    DELETE FROM user WHERE id IN
    <foreach item="id" collection="list" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>
```


### 📥 ParameterType 参数类型详解
`parameterType` 指定传入参数的数据类型，MyBatis 会根据类型自动处理参数映射。

#### 支持的参数类型
1. **基本类型**：`int`, `string`, `double`, `boolean` 等
   ```xml
   <select id="getUserById" parameterType="int" resultType="User">
       SELECT * FROM user WHERE id = #{id}
   </select>
   ```


2. **POJO 类型**：完整的 Java 对象
   ```xml
   <insert id="insertUser" parameterType="User">
       INSERT INTO user(name, age) VALUES(#{name}, #{age})
   </insert>
   ```


3. **Map 类型**：用于传递多个参数
   ```xml
   <select id="findUsers" parameterType="map" resultType="User">
       SELECT * FROM user WHERE name = #{userName} AND age = #{userAge}
   </select>
   ```


4. **List/Array 类型**：用于批量操作
   ```xml
   <select id="findUsersByIds" parameterType="list" resultType="User">
       SELECT * FROM user WHERE id IN
       <foreach item="id" collection="list" open="(" separator="," close=")">
           #{id}
       </foreach>
   </select>
   ```


### 🔑 KeyProperty 主键属性说明
配合 `useGeneratedKeys` 使用，用于获取数据库自动生成的主键值。

#### 使用场景
当你插入一条记录后需要立即获取数据库生成的主键 ID 时非常有用。

#### 示例
```xml
<insert id="insertUser" parameterType="User" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user(name, age) VALUES(#{name}, #{age})
</insert>
```


这样插入后，`User` 对象的 `id` 属性会被自动填充为数据库生成的主键值。

### 🔄 UseGeneratedKeys 自动生成主键详解
控制是否使用数据库自动生成的主键。

#### 不同数据库的支持情况
- **MySQL**：支持 `AUTO_INCREMENT` 字段
- **PostgreSQL**：支持 `SERIAL` 字段
- **Oracle**：需要使用序列（SEQUENCE）和触发器（TRIGGER）组合
- **SQL Server**：支持 `IDENTITY` 字段

#### Oracle 示例（使用序列）
```xml
<insert id="insertUser" parameterType="User">
    <selectKey keyProperty="id" resultType="int" order="BEFORE">
        SELECT user_seq.nextval FROM dual
    </selectKey>
    INSERT INTO user(id, name, age) VALUES(#{id}, #{name}, #{age})
</insert>
```


---

## 🗺️ ResultMap 结果映射高级用法

### 什么时候需要 ResultMap？
当数据库字段名与 Java 对象属性名不一致时，就需要使用 `resultMap` 进行手动映射。

### 基本语法结构
```xml
<resultMap id="映射ID" type="Java类型">
    <id property="Java属性名" column="数据库列名"/>
    <result property="Java属性名" column="数据库列名"/>
</resultMap>
```


### 元素说明
- `<id>`：标识主键字段，用于优化 MyBatis 内部处理（可选，但推荐使用）
- `<result>`：普通字段映射
- `property`：Java 对象的属性名
- `column`：数据库表的列名

### 实际示例
假设数据库表字段为 `user_id`, `user_name`, `user_age`，而 Java 对象属性为 `id`, `name`, `age`：

```xml
<resultMap id="UserResultMap" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <result property="age" column="user_age"/>
</resultMap>

<select id="getUserById" parameterType="int" resultMap="UserResultMap">
    SELECT user_id, user_name, user_age FROM user WHERE user_id = #{id}
</select>
```


### 复杂映射场景
当查询结果包含计算字段或需要特殊处理时：

```xml
<resultMap id="UserDetailMap" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <result property="age" column="user_age"/>
    <result property="fullName" column="full_name"/> <!-- 计算字段 -->
    <result property="statusDesc" column="status_desc"/> <!-- 状态描述 -->
</resultMap>
```


---

## 🎯 动态 SQL 高级特性（初学者必学）

### 🧠 If 标签条件判断
根据条件动态生成 SQL 语句的一部分。

#### 基本语法
```xml
<if test="条件表达式">
    SQL 片段
</if>
```


#### 实际示例
```xml
<select id="findUsers" resultType="User">
    SELECT * FROM user
    WHERE 1=1
    <if test="name != null and name != ''">
        AND name LIKE CONCAT('%', #{name}, '%')
    </if>
    <if test="age != null">
        AND age = #{age}
    </if>
</select>
```


#### 条件表达式语法
- `!= null`：检查参数是否为空
- `!= ''`：检查字符串是否为空串
- `and`、`or`：逻辑运算符
- `==`、`>`、`<`：比较运算符

### 🔄 Choose, When, Otherwise 标签
类似于 Java 中的 `switch-case` 语句，实现单一分支逻辑。

#### 基本语法
```xml
<choose>
    <when test="条件1">
        SQL 片段1
    </when>
    <when test="条件2">
        SQL 片段2
    </when>
    <otherwise>
        默认 SQL 片段
    </otherwise>
</choose>
```


#### 实际示例
```xml
<select id="findUsersByCondition" resultType="User">
    SELECT * FROM user
    <where>
        <choose>
            <when test="name != null and name != ''">
                name LIKE CONCAT('%', #{name}, '%')
            </when>
            <when test="age != null">
                age = #{age}
            </when>
            <otherwise>
                1=1  <!-- 返回所有记录 -->
            </otherwise>
        </choose>
    </where>
</select>
```


### 📍 Where 标签智能处理
自动处理 WHERE 子句，智能去除多余的 `AND` 或 `OR`。

#### 传统写法的问题
```xml
<!-- 不推荐：可能会产生 WHERE AND 的语法错误 -->
<select id="findUsersBad" resultType="User">
    SELECT * FROM user WHERE 
    <if test="name != null">
        AND name = #{name}
    </if>
    <if test="age != null">
        AND age = #{age}
    </if>
</select>
```


#### 使用 Where 标签的正确写法
```xml
<select id="findUsersGood" resultType="User">
    SELECT * FROM user
    <where>
        <if test="name != null">
            AND name = #{name}
        </if>
        <if test="age != null">
            AND age = #{age}
        </if>
    </where>
</select>
```


### ⚙️ Set 标签更新优化
用于更新操作，自动处理 SET 子句，去除多余的逗号。

#### 传统写法的问题
```xml
<!-- 不推荐：可能会产生多余的逗号 -->
<update id="updateUserBad" parameterType="User">
    UPDATE user SET 
    <if test="name != null">name=#{name},</if>
    <if test="age != null">age=#{age},</if>
    WHERE id=#{id}
</update>
```


#### 使用 Set 标签的正确写法
```xml
<update id="updateUserGood" parameterType="User">
    UPDATE user
    <set>
        <if test="name != null">name=#{name},</if>
        <if test="age != null">age=#{age},</if>
    </set>
    WHERE id=#{id}
</update>
```


### 🔧 Trim 标签通用处理
最灵活的动态 SQL 标签，可以自定义前缀、后缀等。

#### 基本属性
- `prefix`：添加前缀
- `suffix`：添加后缀
- `prefixOverrides`：去除前缀中的内容
- `suffixOverrides`：去除后缀中的内容

#### 实际示例
```xml
<!-- 等价于 <where> 标签 -->
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    <if test="name != null">AND name = #{name}</if>
    <if test="age != null">AND age = #{age}</if>
</trim>

<!-- 等价于 <set> 标签 -->
<trim prefix="SET" suffixOverrides=",">
    <if test="name != null">name=#{name},</if>
    <if test="age != null">age=#{age},</if>
</trim>
```


### 🔁 Foreach 标签集合遍历
用于遍历集合，常用于 `IN` 语句和批量操作。

#### 基本属性
- `collection`：要遍历的集合（list/array/map）
- `item`：每次迭代的元素变量名
- `index`：索引变量名（可选）
- `open`：开始字符
- `close`：结束字符
- `separator`：分隔符

#### 实际示例
```xml
<!-- IN 查询 -->
<select id="findUsersByIds" resultType="User">
    SELECT * FROM user WHERE id IN
    <foreach item="id" collection="ids" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>

<!-- 批量插入 -->
<insert id="insertUsers">
    INSERT INTO user(name, age) VALUES
    <foreach item="user" collection="users" separator=",">
        (#{user.name}, #{user.age})
    </foreach>
</insert>
```


#### Collection 属性说明
- 传入 `List` 时：`collection="list"`
- 传入 `Array` 时：`collection="array"`
- 传入 `Map` 时：使用 Map 中的 key
- 使用 `@Param` 注解时：使用注解指定的名称

---

## 🔗 高级查询深入理解（关联查询）

### 一对一查询实现
使用 `<association>` 标签实现对象间的关联关系。

#### 场景示例
用户（User）和账户（Account）是一对一关系：

```xml
<resultMap id="UserAccountMap" type="User">
    <!-- 用户基本信息 -->
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <result property="age" column="user_age"/>
    
    <!-- 一对一关联：用户的账户信息 -->
    <association property="account" javaType="Account">
        <id property="id" column="account_id"/>
        <result property="balance" column="balance"/>
        <result property="accountNo" column="account_no"/>
    </association>
</resultMap>

<select id="getUserWithAccount" resultMap="UserAccountMap">
    SELECT 
        u.id as user_id,
        u.name as user_name,
        u.age as user_age,
        a.id as account_id,
        a.balance,
        a.account_no
    FROM user u
    LEFT JOIN account a ON u.id = a.user_id
    WHERE u.id = #{userId}
</select>
```


### 一对多查询实现
使用 `<collection>` 标签实现集合类型的关联关系。

#### 场景示例
用户（User）和订单（Order）是一对多关系：

```xml
<resultMap id="UserOrdersMap" type="User">
    <!-- 用户基本信息 -->
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <result property="age" column="user_age"/>
    
    <!-- 一对多关联：用户的所有订单 -->
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderId" column="order_no"/>
        <result property="amount" column="amount"/>
        <result property="createTime" column="create_time"/>
    </collection>
</resultMap>

<select id="getUserWithOrders" resultMap="UserOrdersMap">
    SELECT 
        u.id as user_id,
        u.name as user_name,
        u.age as user_age,
        o.id as order_id,
        o.order_no,
        o.amount,
        o.create_time
    FROM user u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.id = #{userId}
</select>
```


### 多对多查询实现
同样使用 `<collection>` 标签，通过中间表实现复杂关联。

#### 场景示例
用户（User）和角色（Role）是多对多关系（通过 user_role 中间表）：

```xml
<resultMap id="UserRoleMap" type="User">
    <!-- 用户基本信息 -->
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <result property="age" column="user_age"/>
    
    <!-- 多对多关联：用户的角色列表 -->
    <collection property="roles" ofType="Role">
        <id property="id" column="role_id"/>
        <result property="roleName" column="role_name"/>
        <result property="description" column="description"/>
    </collection>
</resultMap>

<select id="getUserWithRoles" resultMap="UserRoleMap">
    SELECT 
        u.id as user_id,
        u.name as user_name,
        u.age as user_age,
        r.id as role_id,
        r.role_name,
        r.description
    FROM user u
    LEFT JOIN user_role ur ON u.id = ur.user_id
    LEFT JOIN role r ON ur.role_id = r.id
    WHERE u.id = #{userId}
</select>
```


---

## ⏳ 延迟加载（懒加载）机制

### 什么是延迟加载？
延迟加载是指在需要时才加载关联数据，而不是一次性加载所有数据，可以提高性能。

### 配置方式
在 `application.yml` 中配置：

```yaml
mybatis:
  configuration:
    lazy-loading-enabled: true          # 启用延迟加载
    aggressive-lazy-loading: false      # 禁用激进延迟加载
```


### 使用方法
在 `resultMap` 中使用 `fetchType="lazy"`：

```xml
<resultMap id="UserLazyLoadMap" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    
    <!-- 延迟加载账户信息 -->
    <association property="account" javaType="Account" fetchType="lazy">
        <id property="id" column="account_id"/>
        <result property="balance" column="balance"/>
    </association>
</resultMap>
```


### 配置说明
- `lazy-loading-enabled`：设置为 `true` 启用延迟加载
- `aggressive-lazy-loading`：
  - `true`：调用任何方法都会触发加载（MyBatis 3.4.1 之前默认值）
  - `false`：只有真正访问属性时才会触发加载（推荐）

---

## 🗃️ MyBatis 缓存机制详解

### 一级缓存（SqlSession 级别）
#### 基本概念
- **默认开启**：无需任何配置自动生效
- **生命周期**：与 `SqlSession` 相同
- **作用域**：仅在当前 `SqlSession` 内有效
- **存储位置**：内存中

#### 工作原理
```java
// 第一次查询，会执行 SQL 并缓存结果
User user1 = sqlSession.selectOne("getUserById", 1);

// 第二次查询相同语句和参数，直接从缓存获取
User user2 = sqlSession.selectOne("getUserById", 1);

// user1 和 user2 是同一个对象
System.out.println(user1 == user2); // true
```


#### 缓存失效情况
1. 执行增删改操作
2. 调用 `sqlSession.clearCache()` 清除缓存
3. `SqlSession` 关闭

### 二级缓存（Mapper 级别）
#### 基本概念
- **需要手动开启**：在 `mapper.xml` 中添加缓存声明
- **作用域**：同一个 `Mapper` 的所有 `SqlSession` 共享
- **存储位置**：内存中（可配置其他存储方式）

#### 开启方式
在 `mapper.xml` 文件中添加：

```xml
<!-- 在 mapper 标签下添加 -->
<cache eviction="LRU" flushInterval="60000" size="512" readOnly="true"/>

<!-- 或者使用默认配置 -->
<cache/>
```


#### 缓存属性详细说明
- `eviction`：缓存回收策略
  - `LRU`（默认）：最近最少使用，移除最长时间不被使用的对象
  - `FIFO`：先进先出，按对象进入缓存的顺序来移除
  - `SOFT`：软引用，基于垃圾回收器状态和软引用规则移除对象
  - `WEAK`：弱引用，更积极地移除基于垃圾收集器状态和弱引用规则的对象
- `flushInterval`：刷新间隔（毫秒）
  - 定时清空缓存，不设置则只在语句调用时刷新
- `size`：缓存大小
  - 最多缓存的对象数量，默认值是 1024
- `readOnly`：是否只读
  - `true`：只读缓存，返回缓存对象本身（性能好但不安全）
  - `false`：读写缓存，返回缓存对象的拷贝（安全但性能稍差）

#### 使用注意事项
1. 实体类必须实现 `Serializable` 接口
2. 多表查询的缓存可能存在问题
3. 分布式环境下需要考虑缓存一致性

---

## 📝 MyBatis 注解开发详解（简化配置）

### 基础 CRUD 注解实现
使用注解可以避免编写 XML 文件，让代码更加简洁。

#### 常用注解
```java
@Mapper
public interface UserMapper {
    // 查询
    @Select("SELECT * FROM user WHERE id = #{id}")
    User getUserById(Integer id);
    
    // 插入
    @Insert("INSERT INTO user(name, age) VALUES(#{name}, #{age})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insertUser(User user);
    
    // 更新
    @Update("UPDATE user SET name=#{name}, age=#{age} WHERE id=#{id}")
    int updateUser(User user);
    
    // 删除
    @Delete("DELETE FROM user WHERE id = #{id}")
    int deleteUser(Integer id);
    
    // 多参数查询
    @Select("SELECT * FROM user WHERE name LIKE CONCAT('%', #{name}, '%') AND age = #{age}")
    List<User> findUsers(@Param("name") String name, @Param("age") Integer age);
}
```


### 结果映射注解高级用法
当字段名不一致时，使用 `@Results` 注解：

```java
@Results({
    @Result(property = "id", column = "user_id"),
    @Result(property = "name", column = "user_name"),
    @Result(property = "age", column = "user_age")
})
@Select("SELECT user_id, user_name, user_age FROM user WHERE user_id = #{id}")
User getUserWithResults(Integer id);
```


### 动态 SQL 注解编程
对于复杂的动态 SQL，可以使用 `@SelectProvider` 等注解：

```java
// Mapper 接口
@SelectProvider(type = UserSqlProvider.class, method = "findUsers")
List<User> findUsers(@Param("name") String name, @Param("age") Integer age);

// SQL 提供类
public class UserSqlProvider {
    public String findUsers(Map<String, Object> params) {
        StringBuilder sql = new StringBuilder("SELECT * FROM user WHERE 1=1");
        
        if (params.get("name") != null && !"".equals(params.get("name"))) {
            sql.append(" AND name LIKE CONCAT('%', #{name}, '%')");
        }
        
        if (params.get("age") != null) {
            sql.append(" AND age = #{age}");
        }
        
        return sql.toString();
    }
}
```


### 注解开发优缺点对比
#### 优点
- 代码简洁，减少 XML 文件
- IDE 支持好，便于调试
- 类型安全，编译时检查

#### 缺点
- 复杂 SQL 难以维护
- 动态 SQL 支持有限
- 不如 XML 灵活

#### 建议使用场景
- 简单的 CRUD 操作：使用注解
- 复杂的动态 SQL：使用 XML
- 混合使用：根据具体情况选择