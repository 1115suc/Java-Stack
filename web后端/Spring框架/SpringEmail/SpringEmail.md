# 📧 Spring Email 邮件发送

## 🚀 快速入门

邮件发送在日常开发中非常常见，Spring Boot 为邮件发送提供了很好的支持：

- 邮件发送需要引入 `spring-boot-starter-mail`
- Spring Boot 自动配置 `MailSenderAutoConfiguration`
- 定义 `MailProperties` 内容，配置在 `application.yml` 中
- 自动装配 `JavaMailSender`
- 测试邮件发送

### 📦 依赖引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```


查看引入的依赖，可以看到 `jakarta.mail`：

```xml
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>jakarta.mail</artifactId>
    <version>1.6.4</version>
    <scope>compile</scope>
</dependency>
```


## ⚙️ 核心配置

### 📝 配置文件设置

在 `application.yml` 中配置邮件相关参数：

```yaml
spring:
  mail:
    host: smtp.qq.com
    port: 465
    username: 
    password: 
    default-encoding: UTF-8
    properties:
      mail:
        smtp:
          socketFactory:
            class: javax.net.ssl.SSLSocketFactory
        debug: true
```


### 🧠 核心配置类

Spring Boot 自动配置的核心类：

- `MailSenderAutoConfiguration`：自动配置邮件发送器
- `MailProperties`：邮件配置属性类
- `JavaMailSender`：Java 邮件发送接口

## 📨 邮件发送实现

### 💼 邮件服务类

```java
@Service
public class MailService {
    
    @Autowired
    private JavaMailSender javaMailSender;
    
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        message.setFrom("your-email@qq.com");
        
        javaMailSender.send(message);
    }
}
```


### 📧 不同类型邮件发送

#### 1. 简单文本邮件

```java
@Service
public class MailService {
    
    @Autowired
    private JavaMailSender javaMailSender;
    
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom("your-email@qq.com");
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        
        javaMailSender.send(message);
    }
}
```


#### 2. HTML格式邮件

```java
@Service
public class MailService {
    
    @Autowired
    private JavaMailSender javaMailSender;

    private void sentMailCode(String toEmail , String code){
        try{
            MimeMessage message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(emailConfig.getSendUsername());
            helper.setTo(toEmail);

            helper.setSubject("青言速递 邮箱验证码");

            String htmlContent = new String(Files.readAllBytes(Paths.get("static/email/EmailHtml.html")), "UTF-8");
            htmlContent = htmlContent.replace("123456", code);
            helper.setText(htmlContent, true);

            javaMailSender.send(message);
        }catch(Exception e){
            ...
        }
    }
}
```


#### 3. 带附件的邮件

```java
@Service
public class MailService {
    
    @Autowired
    private JavaMailSender javaMailSender;
    
    public void sendAttachmentsMail(String to, String subject, String content, String filePath) throws MessagingException {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        
        helper.setFrom("your-email@qq.com");
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);
        
        FileSystemResource file = new FileSystemResource(new File(filePath));
        String fileName = file.getFilename();
        helper.addAttachment(fileName, file);
        
        javaMailSender.send(message);
    }
}
```


#### 4. 带静态资源的邮件

```java
@Service
public class MailService {
    
    @Autowired
    private JavaMailSender javaMailSender;
    
    public void sendInlineResourceMail(String to, String subject, String content, String rscPath, String rscId) throws MessagingException {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        
        helper.setFrom("your-email@qq.com");
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);
        
        FileSystemResource res = new FileSystemResource(new File(rscPath));
        helper.addInline(rscId, res);
        
        javaMailSender.send(message);
    }
}
```


## 🌐 邮件服务商配置

### QQ邮箱配置
```yaml
spring:
  mail:
    host: smtp.qq.com
    username: your-email@qq.com
    password: your-authorization-code
    port: 465
```


### 163邮箱配置
```yaml
spring:
  mail:
    host: smtp.163.com
    username: your-email@163.com
    password: your-authorization-code
    port: 25
```


### Gmail配置
```yaml
spring:
  mail:
    host: smtp.gmail.com
    username: your-email@gmail.com
    password: your-app-password
    port: 587
```


## ⚠️ 注意事项

1. **授权码问题**：大部分邮箱服务商需要使用授权码而非登录密码
2. **SSL/TLS配置**：根据邮件服务器要求配置相应的安全协议
3. **异常处理**：邮件发送可能失败，需要做好异常处理
4. **异步发送**：对于大量邮件发送，建议使用异步方式避免阻塞主线程