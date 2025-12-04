# 📊 Elasticsearch 基础学习笔记

## 🔍 Elasticsearch 简介

### 🎯 什么是 Elasticsearch
- Elasticsearch 是一个开源的分布式搜索引擎，可以用来实现搜索、日志统计、分析、系统监控等功能
- Elastic Stack(ELK): 是以 `elasticsearch` 为核心的技术栈，包括 `beats`、`Logstash`、`kibana`、`elasticsearch`
- Lucene: 是 Apache 的开源搜索引擎类库，提供了搜索引擎的核心 API

## 📚 核心概念详解

### 🔢 正向索引 vs 倒排索引

#### 正向索引
例如下表（`tb_goods`）中的 `id` 创建索引：
![image-20210720195531539.png](img/image-20210720195531539.png)

- 如果是根据 `id` 查询，那么直接走索引，查询速度非常快
- 但如果是基于 `title` 做模糊查询，只能是逐行扫描数据，流程如下：
  1. 用户搜索数据，条件是 `title` 符合 `"%手机%"`
  2. 逐行获取数据，比如 `id` 为1的数据
  3. 判断数据中的 `title` 是否符合用户搜索条件
  4. 如果符合则放入结果集，不符合则丢弃。回到步骤1

#### 倒排索引 ✨
倒排索引中有两个非常重要的概念:

- **文档（`Document`）**：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息
- **词条（`Term`）**：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条

创建倒排索引是对正向索引的一种特殊处理，流程如下：
- 将每一个文档的数据利用算法分词，得到一个个词条
- 创建表，每行数据包括词条、词条所在文档 `id`、位置等信息
- 因为词条唯一性，可以给词条创建索引，例如 hash 表结构索引

![image-20210720200457207.png](img/image-20210720200457207.png)

![image-20210720201115192.png](img/image-20210720201115192.png)

## 🏗️ Elasticsearch 核心组件

### 📄 文档和字段
- Elasticsearch 是面向文档（`Document`）存储的，可以是数据库中的一条商品数据，一个订单信息
- 文档数据会被序列化为 json 格式后存储在 Elasticsearch 中
- 而 Json 文档中往往包含很多的**字段（`Field`）**，类似于数据库中的列

![image-20210720202707797.png](img/image-20210720202707797.png)

### 🗃️ 索引和映射
- **索引（`Index`）**: 就是相同类型的文档的集合，因此，我们可以把索引当做是数据库中的表
- 数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息
- 因此，索引库中就有映射（`mapping`），是索引中文档的字段约束信息，类似表的结构约束

![image-20210720203022172.png](img/image-20210720203022172.png)

## 🔄 MySQL 与 Elasticsearch 对比

| **MySQL** | **Elasticsearch** | **说明** |
|-----------|-------------------|----------|
| `Table` | `Index` | 索引(`index`)，就是文档的集合，类似数据库的表(`table`) |
| `Row` | `Document` | 文档（`Document`），就是一条条的数据，类似数据库中的行（`Row`），文档都是 JSON 格式 |
| `Column` | `Field` | 字段（`Field`），就是 JSON 文档中的字段，类似数据库中的列（`Column`） |
| `Schema` | `Mapping` | `Mapping`（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（`Schema`） |
| `SQL` | `DSL` | `DSL` 是 elasticsearch 提供的 JSON 风格的请求语句，用来操作 elasticsearch，实现 CRUD |

![image-20210720203534945.png](img/image-20210720203534945.png)

## 🗃️ 索引库操作详解

### 创建索引库
索引库是 Elasticsearch 中存储文档的地方，相当于数据库中的表。

mapping是对索引库中文档的约束，常见的mapping属性包括：

- type：字段数据类型，常见的简单类型有：
  - 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
  - 数值：long、integer、short、byte、double、float、
  - 布尔：boolean
  - 日期：date
  - 对象：object
- index：是否创建索引，默认为true
- analyzer：使用哪种分词器
- properties：该字段的子字段

```json
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"  // keyword类型：精确匹配，不分词
      },
      "name": {
        "type": "text",    // text类型：全文检索，会分词
        "analyzer": "ik_max_word",  // 使用IK分词器进行分词
        "copy_to": "all"   // 将内容复制到all字段，用于综合搜索
      },
      "address": {
        "type": "keyword",
        "index": false     // 不建立索引，节省空间，但无法搜索
      },
      "price": {
        "type": "integer"  // 数值类型
      },
      "score": {
        "type": "integer"
      },
      "brand": {
        "type": "keyword"  // 品牌字段，用于精确匹配和聚合
      },
      "city": {
        "type": "keyword"  // 城市字段，用于筛选
      },
      "starName": {
        "type": "keyword"  // 星级字段
      },
      "business": {
        "type": "keyword"  // 商圈字段
      },
      "location": {
        "type": "geo_point"  // 地理位置类型，支持距离查询
      },
      "pic": {
        "type": "keyword",
        "index": false     // 图片链接不建立索引
      },
      "all": {
        "type": "text",    // 综合搜索字段
        "analyzer": "ik_smart" // 使用ik分词器进行分词
      }
    }
  }
}
```


### 查看索引库
```bash
GET /hotel  # 查看hotel索引的详细信息
GET /_cat/indices  # 查看所有索引
```


### 删除索引库
```bash
DELETE /hotel  # 删除hotel索引及其所有数据
```


删除索引库会同时删除：
- 索引的 `mapping` 配置
- 索引中的所有 `document`
- 索引相关的分片和副本

### 修改索引库

#### 修改索引库设置
```bash
PUT /hotel/_settings
{
  "number_of_replicas": 2  // 修改副本数量为2
}
```


#### 添加新的字段映射
```bash
PUT /hotel/_mapping
{
  "properties": {
    "new_field": {
      "type": "text",
      "analyzer": "ik_max_word"
    }
  }
}
```


#### 注意事项
- Elasticsearch 中的 `mapping` 一旦创建，已有的字段类型不能修改
- 可以添加新的字段映射
- 可以修改索引的设置（如副本数量、刷新间隔等）
- 主分片数量在索引创建后不能修改，需要重新创建索引

## 📄 文档操作

### 新增文档

文档是 Elasticsearch 中的基本数据单元，相当于数据库中的一行记录。Elasticsearch 提供了多种方式来新增文档：

#### 自动分配ID
```json
POST /hotel/_doc
{
  "id": 61001,
  "name": "如家酒店",
  "address": "北京市朝阳区",
  "price": 200,
  "score": 4,
  "brand": "如家",
  "city": "北京",
  "starName": "二星级",
  "business": "国贸商圈",
  "location": "39.908611,116.397222",
  "pic": "http://example.com/hotel.jpg",
  "all": "如家酒店 北京市朝阳区 如家 国贸商圈"
}
```


#### 指定文档ID
```json
POST /hotel/_doc/61001
{
  "id": 61001,
  "name": "如家酒店",
  "address": "北京市朝阳区",
  "price": 200,
  "score": 4,
  "brand": "如家",
  "city": "北京",
  "starName": "二星级",
  "business": "国贸商圈",
  "location": "39.908611,116.397222",
  "pic": "http://example.com/hotel.jpg",
  "all": "如家酒店 北京市朝阳区 如家 国贸商圈"
}
```


#### 使用 `_create` 端点（仅当文档不存在时创建）
```json
PUT /hotel/_create/61001
{
  "id": 61001,
  "name": "如家酒店",
  "address": "北京市朝阳区",
  "price": 200,
  "score": 4,
  "brand": "如家",
  "city": "北京",
  "starName": "二星级",
  "business": "国贸商圈",
  "location": "39.908611,116.397222",
  "pic": "http://example.com/hotel.jpg",
  "all": "如家酒店 北京市朝阳区 如家 国贸商圈"
}
```


### 查看文档

#### 基本查询
```bash
GET /hotel/_doc/61001  # 根据ID查看指定文档
```


#### 只返回_source字段
```bash
GET /hotel/_source/61001
```


#### 检查文档是否存在
```bash
HEAD /hotel/_doc/61001  # 返回200表示存在，404表示不存在
```


#### 指定返回字段
```json
GET /hotel/_doc/61001?_source_includes=name,price,brand
```


### 删除文档

#### 基本删除
```bash
DELETE /hotel/_doc/61001  # 根据ID删除指定文档
```


#### 带条件删除（使用更新API）
```json
POST /hotel/_delete_by_query
{
  "query": {
    "match": {
      "brand": "如家"
    }
  }
}
```


### 修改文档

#### 全量更新
全量更新会替换整个文档内容：
```bash
PUT /hotel/_doc/61001
{
  "id": 61001,
  "name": "如家精选酒店",  // 修改了名称
  "address": "北京市朝阳区",
  "price": 250,          // 修改了价格
  "score": 5,            // 修改了评分
  "brand": "如家",
  "city": "北京",
  "starName": "三星级",   // 修改了星级
  "business": "国贸商圈",
  "location": "39.908611,116.397222",
  "pic": "http://example.com/hotel.jpg",
  "all": "如家精选酒店 北京市朝阳区 如家 国贸商圈"
}
```


#### 局部更新
局部更新只会修改指定的字段，其他字段保持不变：

##### 基本局部更新
```bash
POST /hotel/_update/61001
{
  "doc": {
    "price": 280,  // 只更新价格字段
    "score": 5     // 只更新评分字段
  }
}
```


#### 批量操作
```json
POST /_bulk
{ "index" : { "_index" : "hotel", "_id" : "61003" } }
{ "name": "7天酒店", "price": 180, "brand": "7天" }
{ "update" : { "_index" : "hotel", "_id" : "61001" } }
{ "doc" : { "price" : 300 } }
{ "delete" : { "_index" : "hotel", "_id" : "61002" } }
```


### 文档版本控制

Elasticsearch 使用 `_version` 字段来控制并发更新：

```json
POST /hotel/_update/61001?version=2
{
  "doc": {
    "price": 320
  }
}
```


### 响应状态说明

文档操作的常见响应状态码：
- `200 OK`: 操作成功
- `201 Created`: 文档创建成功
- `404 Not Found`: 文档不存在
- `409 Conflict`: 版本冲突

## 🌐 RestAPI 操作详解

### 常用 HTTP 方法说明
- `GET`: 查询数据，安全操作
- `POST`: 新增数据，非幂等操作
- `PUT`: 修改数据（全量更新），幂等操作
- `DELETE`: 删除数据，幂等操作

### RestClient 操作索引库

#### 引入依赖
```xml
<dependency>
  <groupId>org.elasticsearch.client</groupId>
  <artifactId>elasticsearch-rest-high-level-client</artifactId>
  <version>7.12.1</version>
</dependency>
```


#### 创建 RestHighLevelClient
```java
public class HotelIndexTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.150.101:9200")
        ));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}
```


#### 创建索引库
```java
public class HotelConstants {
    public static final String MAPPING_TEMPLATE = "{\n" +
        ...
    "}";
}
```
```java
    @Test
    void createHotelIndex() throws IOException {
        // 1.创建Request对象
        CreateIndexRequest request = new CreateIndexRequest("hotel");
        // 2.准备请求的参数：DSL语句
        request.source(MAPPING_TEMPLATE, XContentType.JSON);
        // 3.发送请求
        client.indices().create(request, RequestOptions.DEFAULT);
    }
```


#### 删除索引库
```java
    @Test
    void testDeleteHotelIndex() throws IOException {
        // 1.创建Request对象
        DeleteIndexRequest request = new DeleteIndexRequest("hotel");
        // 2.发送请求
        client.indices().delete(request, RequestOptions.DEFAULT);
    }
```


#### 判断索引库是否存在
```java
    @Test
    void testExistsHotelIndex() throws IOException {
        // 1.创建Request对象
        GetIndexRequest request = new GetIndexRequest("hotel");
        // 2.发送请求
        boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
        // 3.输出
        System.err.println(exists ? "索引库已经存在！" : "索引库不存在！");
    }
```


### RestClient 操作文档

#### 初始化 RestHighLevelClient
```java
@SpringBootTest
public class HotelDocumentTest {
    @Autowired
    private IHotelService hotelService;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.88.128:9200")
        ));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}
```


#### 新增文档
```java
    @Test
    void testAddDocument() throws IOException {
        // 1.根据id查询酒店数据
        Hotel hotel = hotelService.getById(61083L);
        // 2.转换为文档类型
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 3.将HotelDoc转json
        String json = JSON.toJSONString(hotelDoc);
    
        // 1.准备Request对象
        IndexRequest request = new IndexRequest("hotel").id(hotelDoc.getId().toString());
        // 2.准备Json文档
        request.source(json, XContentType.JSON);
        // 3.发送请求
        client.index(request, RequestOptions.DEFAULT);
    }
```


#### 查询文档
```java
    @Test
    void testGetDocumentById() throws IOException {
        // 1.准备Request
        GetRequest request = new GetRequest("hotel", "61082");
        // 2.发送请求，得到响应
        GetResponse response = client.get(request, RequestOptions.DEFAULT);
        // 3.解析响应结果
        String json = response.getSourceAsString();
    
        HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
        System.out.println(hotelDoc);
    }
```


#### 删除文档
```java
    @Test
    void testDeleteDocument() throws IOException {
        // 1.准备Request
        DeleteRequest request = new DeleteRequest("hotel", "61083");
        // 2.发送请求
        client.delete(request, RequestOptions.DEFAULT);
    }
```


#### 修改文档
- 全量修改：本质是先根据id删除，再新增
- 增量修改：修改文档中的指定字段值

在RestClient的API中，全量修改与新增的API完全一致，判断依据是ID：
- 如果新增时，ID已经存在，则修改
- 如果新增时，ID不存在，则新增

```java
    @Test
    void testUpdateDocument() throws IOException {
        // 1.准备Request
        UpdateRequest request = new UpdateRequest("hotel", "61083");
        // 2.准备请求参数
        request.doc(
            "price", "952",
            "starName", "四钻"
        );
        // 3.发送请求
        client.update(request, RequestOptions.DEFAULT);
    }
```


#### 批量导入文档
```java
    @Test
    void testBulkRequest() throws IOException {
        // 批量查询酒店数据
        List<Hotel> hotels = hotelService.list();
    
        // 1.创建Request
        BulkRequest request = new BulkRequest();
        // 2.准备参数，添加多个新增的Request
        for (Hotel hotel : hotels) {
            // 2.1.转换为文档类型HotelDoc
            HotelDoc hotelDoc = new HotelDoc(hotel);
            // 2.2.创建新增文档的Request对象
            request.add(new IndexRequest("hotel")
                        .id(hotelDoc.getId().toString())
                        .source(JSON.toJSONString(hotelDoc), XContentType.JSON));
        }
        // 3.发送请求
        client.bulk(request, RequestOptions.DEFAULT);
    }
```
