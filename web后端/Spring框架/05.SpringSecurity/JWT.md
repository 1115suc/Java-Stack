# 🔐 JWT (JSON Web Token)

## 📋 常见认证方式对比

### 🍪 Cookie-Session 认证

#### 认证流程 🔄
- 用户输入用户名、密码或者用短信验证码方式登录系统
- 服务端验证后，创建一个 Session 记录用户登录信息，并且将 SessionID 存到 cookie，响应回浏览器
- 下次客户端再发起请求，自动带上 cookie 信息，服务端通过 cookie 获取 Session 信息进行校验

![image-20210116130304029.png](img/image-20210116130304029.png)

#### ⚠️ 弊端
- 只能在 web 场景下使用，如果是 APP 中，不能使用 cookie 的情况下就不能用了 📱❌
- 即使能在 web 场景下使用，也要考虑跨域问题，因为 cookie 不能跨域 🌐❌
- cookie 存在 CSRF（跨站请求伪造）的风险 🕷️
- 如果是分布式服务，需要考虑 Session 同步问题 🔄
- session-cookie机制是有状态的方式（后端保存用户信息-浪费后端服务器内存）💾

### 🔑 JWT令牌无状态认证

JSON Web Token（JWT-字符串）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

#### 认证流程 🚀
- 依然是用户登录系统
- 服务端验证，并通过指定的算法生成令牌返回给客户端
- 客户端拿到返回的 Token，存储到 local storage/session Storate/Cookie中 📦
- 下次客户端再次发起请求，将 Token 附加到 header 中
- 服务端获取 header 中的 Token，通过相同的算法对 Token 进行验证，如果验证结果相同，则说明这个请求是正常的，没有被篡改。这个过程可以完全不涉及到查询 Redis 或其他存储

![image-20210116135838264.png](img/image-20210116135838264.png)

#### ✅ 优点
- 使用 json 作为数据传输，有广泛的通用型，并且体积小，便于传输 📄
- 不需要在服务器端保存相关信息，节省内存资源的开销 💰
- jwt 载荷部分可以存储业务相关的信息（非敏感的），例如用户信息、角色等 👤

## 🧩 JWT组成结构

### 📄 头部（Header）
- 描述JWT的基本信息，如数据类型和签名算法
- 示例：`{"typ":"JWT","alg":"HS256"}`
- BASE64编码后：`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9`
- ⚠️ BASE64不是加密算法，可正反向编解码

### 📦 载荷（Payload）
- 存放有效信息，可自定义业务数据
- 示例格式：`{"sub":"1234567890","name":"John Doe","admin":true}`
- BASE64编码形成JWT第二部分

### 🔏 签证（Signature）
- 由三部分组成：`签名算法(header.payload.secret)`
- 使用secret密钥对header和payload进行加密
- 构成JWT的第三部分，确保令牌安全性

## 💻 实现JWT认证

### 📦 引入依赖
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

### 🔨 生成JWT令牌
```java
@Test
public void testGenerate(){
    String compact = Jwts.builder()
        .setId(UUID.randomUUID().toString()) // 设置唯一标识
        .setSubject("title") // 设置主题
        .claim("name", "张三") // 自定义信息
        .claim("age", 88) // 自定义信息
        .setExpiration(new DateTime().plusDays(1).toDate()) // 设置过期时间
        .setIssuedAt(new Date()) // 令牌签发时间
        .signWith(SignatureAlgorithm.HS256, "1115suc") // 签名算法, 秘钥
        .compact();
    System.out.println(compact);
}
```

### 🔍 解析JWT令牌
```java
@Test
public void testVerify(){
    String jwt = "";
    Claims claims = Jwts.parser().setSigningKey("1115suc").parseClaimsJws(jwt).getBody();
    Object name = claims.get("name"); // 获取自定义信息
    System.out.println(claims);
}
```

主要修改内容：
1. **代码注释中的表情已删除**：如`// 🆔 设置唯一标识`改为`// 设置唯一标识`
2. **保持了代码的专业性**：注释只保留纯文字说明
3. **保留了标题和正文的表情**：让整体内容保持生动性
4. **保持了技术内容的完整性**：所有代码逻辑和功能说明不变

这样既保持了文档的趣味性，又确保了代码部分的专业性和可读性。
