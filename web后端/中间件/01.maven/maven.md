# 📦 Maven 

作为一名 Java 后端开发工程师，Maven 是我们日常开发中不可或缺的构建工具。它帮助我们管理项目的依赖、构建、文档生成等

## 🚀 Maven 基础概念

### 什么是 Maven？
Maven 是一个项目管理和构建自动化工具，主要基于项目对象模型（POM - Project Object Model）的概念。

### 核心特性
- **依赖管理**：自动下载和管理项目所需的依赖库
- **标准化目录结构**：统一的项目布局规范
- **生命周期管理**：预定义的构建生命周期
- **插件机制**：丰富的插件生态系统

---

## 📁 Maven 项目标准目录结构

```
my-app/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   ├── resources/
│   │   └── webapp/ (Web项目)
│   └── test/
│       ├── java/
│       └── resources/
└── target/
```


---

## 🛠️ Maven 核心命令

### 项目创建和初始化
```bash
# 创建 Maven 项目原型
mvn archetype:generate

# 快速创建简单项目
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```


### 生命周期命令
```bash
# 清理项目（删除 target 目录）
mvn clean

# 编译源代码
mvn compile

# 编译测试代码
mvn test-compile

# 运行测试
mvn test

# 打包项目（jar/war）
mvn package

# 安装到本地仓库
mvn install

# 部署到远程仓库
mvn deploy
```


### 常用组合命令
```bash
# 清理并编译
mvn clean compile

# 清理并测试
mvn clean test

# 清理并打包
mvn clean package

# 清理并安装到本地仓库
mvn clean install
```


---

## 📄 pom.xml 配置详解

`pom.xml` 是 Maven 项目的核心配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <!-- 基本信息 -->
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <!-- 项目属性 -->
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <!-- 依赖管理 -->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
</project>
```


### 重要配置项说明

#### 项目坐标
- `groupId`：组织标识符（通常是域名反写）
- `artifactId`：项目唯一标识符
- `version`：项目版本号
- `packaging`：打包方式（jar/war/pom）

#### 依赖范围（Scope）
- `compile`：编译范围（默认），对编译、测试、运行都有效
- `test`：测试范围，只对测试有效
- `provided`：已提供范围，编译和测试时需要，运行时由容器提供
- `runtime`：运行时范围，测试和运行时需要，编译不需要
- `system`：系统范围，需指定本地系统路径

---

## 📚 依赖管理

### 添加依赖
```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.7.0</version>
    </dependency>
    
    <!-- MySQL Connector -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```


### 依赖传递和冲突解决
```bash
# 查看依赖树
mvn dependency:tree

# 分析依赖冲突
mvn dependency:analyze

# 排除传递依赖
<exclusions>
    <exclusion>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
    </exclusion>
</exclusions>
```


---

## 🏗️ 构建配置

### 插件配置
```xml
<build>
    <plugins>
        <!-- 编译插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
        
        <!-- 打包插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.example.Main</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```


### 资源过滤
```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```


---

## 🌐 仓库管理

### 本地仓库
默认位置：`~/.m2/repository/`

### 远程仓库配置
```xml
<!-- settings.xml 中配置 -->
<mirrors>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云公共仓库</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
</mirrors>
```


### 私服配置
```xml
<repositories>
    <repository>
        <id>nexus</id>
        <name>Nexus Repository</name>
        <url>http://localhost:8081/repository/maven-public/</url>
    </repository>
</repositories>
```


---

## 💼 开发实战技巧

### 多模块项目
```xml
<!-- 父项目 pom.xml -->
<packaging>pom</packaging>
<modules>
    <module>service</module>
    <module>web</module>
    <module>common</module>
</modules>
```


### Profile 配置
```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```


激活 Profile：
```bash
mvn clean package -P prod
```


### 常用插件推荐
- `maven-surefire-plugin`：测试执行插件
- `maven-failsafe-plugin`：集成测试插件
- `maven-shade-plugin`：创建 uber-jar
- `spring-boot-maven-plugin`：Spring Boot 应用打包

---

## 🔧 故障排查和优化

### 常见问题解决
```bash
# 跳过测试
mvn clean package -DskipTests

# 强制更新依赖
mvn clean compile -U

# 清理本地仓库缓存
mvn dependency:purge-local-repository

# 查看有效 POM
mvn help:effective-pom
```


### 性能优化
- 合理使用依赖范围
- 排除不必要的传递依赖
- 使用私服加速依赖下载
- 并行构建：`mvn -T 4 clean package`

