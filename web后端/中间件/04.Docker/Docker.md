# 🐳 Docker 学习笔记

## 📦 Docker 常用命令

### 镜像相关操作
- `docker images`：列出本地镜像
- `docker pull <image>:tag`：拉取镜像
- `docker rmi <image_id>`：删除镜像
- `docker build -t <tag> .`：根据 Dockerfile 构建镜像
- `docker tag <source_image> <target_image>`：给镜像打标签
- `docker save -o <image_id> <file_name>.tar`: 保存镜像为 tar 文件
- `docker load -i <file_name>.tar`：从 tar 文件加载镜像

![image-20230105111738797.png](img/image-20230105111738797.png)

### 容器相关操作
- `docker run -d --name <container_name> <image>`：后台运行容器
- `docker run -it <image> /bin/bash`：交互式运行容器
- `docker ps`：查看正在运行的容器
- `docker ps -a`：查看所有容器（包括停止的）
- `docker stop <container_id>`：停止容器
- `docker pause <container_id>`: 暂停容器
- `docker unpause <container_id>`: 恢复已暂停的容器
- `docker start <container_id>`：启动已停止的容器
- `docker rm -f <container_id>`：删除容器(-f 强制删除)
- `docker logs -f <container_id>`：查看容器日志(-f 动态查询)
- `docker exec -it <container_id> /bin/bash`：进入容器内部

### 数据卷相关操作
- `docker volume create <volume_name>`：创建数据卷
- `docker volume inspect <volume_name>`：查看数据卷信息
- `docker volume rm <volume_name>`：删除数据卷
- `docker volume ls`：列出所有数据卷
- `docker volume prune <volume_name>`: 清理未使用的数据

### 其他常用命令
- `docker system df`：查看 Docker 磁盘使用情况
- `docker system prune`：清理未使用的数据

---

## 🌐 更换镜像源

由于默认的 Docker Hub 在国内访问较慢，建议配置国内镜像源：

### Windows/Mac (Docker Desktop)
1. 打开 Docker Desktop 设置
2. 进入 `Docker Engine` 选项卡
3. 添加以下配置：
   ```json
   {
     "registry-mirrors": [
       "https://hub-mirror.c.163.com",
       "https://mirror.baidubce.com"
     ]
   }
   ```

4. 点击 Apply & Restart

### Linux 系统
1. 创建或编辑 daemon 配置文件：
   ```bash
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": [
       "https://hub-mirror.c.163.com",
       "https://mirror.baidubce.com"
     ]
   }
   EOF
   ```

2. 重启 Docker 服务：
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```


---

## 📄 Dockerfile 编写指南

Dockerfile 是用来构建 Docker 镜像的文本文件，包含一系列指令：

### 基础指令
- `FROM <image>`：指定基础镜像
- `WORKDIR <path>`：设置工作目录
- `COPY <src> <dest>`：复制文件到镜像
- `ADD <src> <dest>`：复制文件，支持解压 tar 包
- `RUN <command>`：执行命令并创建新的镜像层
- `CMD ["executable","param1","param2"]`：容器启动时执行的命令
- `ENTRYPOINT ["executable", "param1", "param2"]`：容器入口点
- `EXPOSE <port>`：暴露端口
- `ENV <key>=<value>`：设置环境变量

### 示例 Java 应用 Dockerfile
```dockerfile
FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```


---

## 🏗️ 构建镜像

### 构建步骤
1. 编写 Dockerfile
2. 在 Dockerfile 所在目录执行构建命令：
   ```bash
   docker build -t myapp:latest .
   ```

3. 查看构建结果：
   ```bash
   docker images
   ```


### 最佳实践
- 使用 `.dockerignore` 文件排除不必要的文件
- 多阶段构建减小镜像体积
- 合理利用镜像缓存

---

## 🎼 Docker Compose

Docker Compose 是用于定义和运行多容器 Docker 应用程序的工具。

### 安装 Docker Compose
```bash
# Linux 安装
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```


### docker-compose.yml 示例
```yaml
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
    depends_on:
      - database

  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```


### 常用命令
- `docker-compose up`：启动所有服务
- `docker-compose up -d`：后台启动所有服务
- `docker-compose down`：停止并移除所有服务
- `docker-compose logs`：查看服务日志
- `docker-compose ps`：查看运行的服务

---

## 🚀 Docker 部署 Java 后端项目

### 后端 Spring Boot 项目部署
1. 构建 jar 包：
   ```bash
   mvn clean install
   ```


2. 创建 Dockerfile：
   ```dockerfile
   FROM openjdk:11-jre-slim
   WORKDIR /app
   COPY target/*.jar app.jar
   EXPOSE 8080
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```


3. 构建镜像：
   ```bash
   docker build -t springboot-app .
   ```


4. 运行容器：
   ```bash
   docker run -d -p 8080:8080 --name myapp springboot-app
   ```
