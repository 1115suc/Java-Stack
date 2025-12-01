# ShardingSphere-JDBC

## 使用

### 1. 前提条件
- Java 1.8+
- Maven

### 2. 引入依赖

#### 方法一：直接引入依赖
```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc</artifactId>
    <version>${latest.release.version}</version>
</dependency>
```

#### 方法二：使用BOM管理依赖版本

##### - 引入 BOM 依赖

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>shardingsphere-bom</artifactId>
            <version>${shardingsphere.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

##### - 导入 BOM 后，您可以声明 ShardingSphere 依赖而无需指定版本：

```xml
<dependencies>
    <!-- ShardingSphere JDBC 驱动 -->
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>shardingsphere-jdbc</artifactId>
    </dependency>

    <!-- MySQL SQL 解析器 -->
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>shardingsphere-parser-sql-engine-mysql</artifactId>
    </dependency>

    <!-- 数据源池实现 -->
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>shardingsphere-infra-data-source-pool-hikari</artifactId>
    </dependency>
</dependencies>
```

### 3. 创建 YAML 配置文件

#### 通用配置参数

| 名称 | 数据类型 | 说明 | 默认值 |
|------|----------|------|--------|
| sql-show (?) | boolean | 是否在日志中打印 SQL| false |
| sql-simple (?) | boolean | 是否在日志中打印简单风格的 SQL | false |
| kernel-executor-size (?) | int | 用于设置任务处理线程池的大小| infinite |
| max-connections-size-per-query (?) | int | 一次查询请求在每个数据库实例中所能使用的最大连接数 | 1 |
| check-table-metadata-enabled (?) | boolean | 在程序启动和更新时，是否检查分片元数据的结构一致性 | false |
| load-table-metadata-batch-size (?) | int | 在程序启动或刷新元数据时，单个批次加载表元数据的数量 | 1000 |

##### 配置示例

- 属性配置直接配置在 ShardingSphere-JDBC 所使用的配置文件中，格式如下：
```yaml
props:
    sql-show: true
```

#### yaml 配置参数

##### 模式配置

- 单机模式
```yaml
mode:
    type: Standalone
    repository:
        type: JDBC
```

- 集群模式
```yaml
mode:
  type: Cluster
  repository:
    type: ZooKeeper
    props:
      namespace: governance
      server-lists: localhost:2181
      retryIntervalMilliseconds: 500
      timeToLiveSeconds: 60
```
- 使用持久化仓库需要额外引入对应的 Maven 依赖，推荐使用：
```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-cluster-mode-repository-zookeeper</artifactId>
    <version>${shardingsphere.version}</version>
</dependency>
```
