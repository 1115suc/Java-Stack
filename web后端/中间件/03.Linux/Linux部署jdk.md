# Linux 部署 JDK

## 1. 下载 JDK

> https://repo.huaweicloud.com/java/jdk/

## 2. 解压 JDK

> 解压到 /opt/java 或 /usr/lib/jvm

```shell
tar -zxvf jdk-8u181-linux-x64.tar.gz
```

## 3. 配置JAVA_HOME和PATH环境变量

> 环境变量必须对所有用户（尤其是后台服务）生效，推荐写入系统级配置文件

- 编辑 /etc/environment 文件

```environment
JAVA_HOME="/opt/java/jdk1.8.0_181"
```

- 编辑 /etc/profile/java.sh 文件

```shell
export JAVA_HOME="/opt/java/jdk1.8.0_181"
export PATH=$JAVA_HOME/bin:$PATH
```

- 执行 java.sh 文件