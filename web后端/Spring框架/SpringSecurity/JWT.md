# JWT

## 常见认证方式

### Cookie-Session
#### 认证过程大致如下：
- 用户输入用户名、密码或者用短信验证码方式登录系统
- 服务端验证后，创建一个 Session 记录用户登录信息 ，并且将 SessionID 存到 cookie，响应回浏览器
- 下次客户端再发起请求，自动带上 cookie 信息，服务端通过 cookie 获取 Session 信息进行校验

![image-20210116130304029.png](img/image-20210116130304029.png)

#### **弊端**
- 只能在 web 场景下使用，如果是 APP 中，不能使用 cookie 的情况下就不能用了；
- 即使能在 web 场景下使用，也要考虑跨域问题，因为 cookie 不能跨域；（域名或者ip一致，端口号一致，协议要一致）
- cookie 存在 CSRF（跨站请求伪造）的风险；
- 如果是分布式服务，需要考虑 Session 同步（同步）问题；
- session-cookie机制是有状态的方式（后端保存主题的用户信息-浪费后端服务器内存）

### JWT令牌无状态认证

JSON Web Token（JWT-字符串）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

#### 认证过程:
- 依然是用户登录系统；
- 服务端验证，并通过指定的算法生成令牌返回给客户端;
- 客户端拿到返回的 Token，存储到 local storage/session Storate/Cookie中；
- 下次客户端再次发起请求，将 Token 附加到 header 中；
- 服务端获取 header 中的 Token ，通过相同的算法对 Token 进行验证，如果验证结果相同，则说明这个请求是正常的，没有被篡改。这个过程可以完全不涉及到查询 Redis 或其他存储；

![image-20210116135838264.png](img/image-20210116135838264.png)

#### **优点**
- 使用 json 作为数据传输，有广泛的通用型，并且体积小，便于传输
- 不需要在服务器端保存相关信息，节省内存资源的开销
- jwt 载荷部分可以存储业务相关的信息（非敏感的），例如用户信息、角色等

## JWT组成

- **头部（Header）（非敏感）**
    - 头部用于描述关于该JWT的最基本的信息，例如数据类型以及签名所用的算法等，本质是一个JSON格式对象；
    - 举例说明
        - {"typ":"JWT","alg":"HS256"}  解释：在头部指明了签名算法是HS256算法，整个JSON对象被BASE64编码形成JWT头部字符串信息；
        - BASE64编码详见：https://tool.oschina.net/encrypt ,编码后的字符串：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
        - 值得注意的是BASE64不是加密算法，可进行正向编码和反向解码处理;
- **载荷（playload）（非敏感数据）**
    - 载荷就是存放有效信息的地方，该部分的信息是可以自定义的；
    - 载荷payload格式:{"sub":"1234567890","name":"John Doe","admin":true}
    - 载荷相关的JSON对象经过BASE64编码形成JWT第二部分：eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG7CoERvZSIsImFkbWluIjp0cnVlfQ==
- **签证（signature）**
    - jwt的第三部分是一个签证信息，这个签证信息由三部分组成：<u>签名算法( header (base64后的).payload (base64后的) . secret )</u>
    - 这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret秘钥组合加密，然后就构成了jwt的第三部分：TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

## 实现JWT认证

