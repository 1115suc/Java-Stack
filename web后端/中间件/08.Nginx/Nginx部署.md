# 🚀 Nginx 安装指南

## 🐳 基于 Docker 构建 Nginx

### 1. 拉取 Nginx 镜像
```shell
docker pull nginx
```


### 2. 创建 Nginx 配置文件目录
```shell
mkdir -p /usr/local/nginx_info/conf
mkdir -p /usr/local/nginx_info/log
mkdir -p /usr/local/nginx_info/html
```

- 拷贝默认配置文件
```shell
docker run --name nginx -p 80:80 -d nginx
docker cp nginx:/etc/nginx/nginx.conf /usr/local/nginx_info/conf/nginx.conf
docker cp nginx:/etc/nginx/conf.d /usr/local/nginx_info/conf/conf.d
docker cp nginx:/usr/share/nginx/html /usr/local/nginx_info/
```

### 3. 创建 Nginx 容器
```shell
docker run --name nginx \
    -v /usr/local/nginx_info/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /usr/local/nginx_info/conf/conf.d:/etc/nginx/conf.d \
    -v /usr/local/nginx_info/log:/var/log/nginx \
    -v /usr/local/nginx_info/html:/usr/share/nginx/html \
    -p 80:80 -d nginx
```


### 4. 防火墙配置（如需要）
```shell
# 关闭80端口防火墙
firewall-cmd --zone=public --remove-port=80/tcp --permanent
firewall-cmd --reload
```

---

## 💻 源码编译安装 Nginx

### 1. 安装依赖包
```shell
# 安装C编译环境及第三方依赖库
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```


### 2. 下载 Nginx 安装包
```shell
# 安装wget工具
yum install wget

# 下载Nginx源码包
wget https://nginx.org/download/nginx-1.16.1.tar.gz
```


> 📝 `wget` 命令说明：
> - 从指定URL下载文件
> - 网络不稳定时具有重试机制
> - 支持断点续传功能

### 3. 解压安装包
```shell
tar -zxvf nginx-1.16.1.tar.gz
```


### 4. 配置编译环境
```shell
# 创建安装目录
mkdir -p /SuChan/Tool/nginx

# 进入源码目录
cd nginx-1.16.1

# 配置安装路径
./configure --prefix=/SuChan/Tool/nginx
```


> 📝 配置参数说明：
> - `--prefix`：指定Nginx安装目录

### 5. 编译并安装
```shell
# 编译源码
make

# 安装到指定目录
make install
```
