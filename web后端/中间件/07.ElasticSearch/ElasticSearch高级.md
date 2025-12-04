# 🚀 ElasticSearch 高级指南

## 📈 数据聚合详解

**[聚合（aggregations）](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)** 可以让我们极其方便地实现对数据的统计、分析、运算。例如：

- 什么品牌的手机最受欢迎？
- 这些手机的平均价格、最高价格、最低价格？
- 这些手机每月的销售情况如何？

实现这些统计功能比数据库的 SQL 要方便得多，而且查询速度非常快，可以实现近实时搜索效果。

### 🧱 DSL 实现聚合

#### 🪣 Bucket 聚合语法

```json
GET /hotel/_search
{
  "size": 0,  // 设置size为0，结果中不包含文档，只包含聚合结果
  "aggs": { // 定义聚合
    "brandAgg": { //给聚合起个名字
      "terms": { // 聚合的类型，按照品牌值聚合，所以选择term
        "field": "brand", // 参与聚合的字段
        "size": 20 // 希望获取的聚合结果数量
      }
    }
  }
}
```


![image-20210723171948228.png](img/image-20210723171948228.png)

#### 📊 聚合结果排序

```json
GET /hotel/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "order": {
          "_count": "asc" // 按照_count升序排列
        },
        "size": 20
      }
    }
  }
}
```


#### 🔍 限定聚合范围

默认情况下，Bucket 聚合是对索引库的所有文档做聚合，但真实场景下，用户会输入搜索条件，因此聚合必须是对搜索结果聚合。那么聚合必须添加限定条件。

我们可以限定要聚合的文档范围，只要添加 query 条件即可：

```json
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "lte": 200 
      }
    }
  }, 
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20
      }
    }
  }
}
```


![image-20210723172404836.png](img/image-20210723172404836.png)

#### 📐 Metric 聚合语法

```json
GET /hotel/_search
{
  "size": 0, // 不返回文档数据
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20
      },
      "aggs": { // 是brands聚合的子聚合，也就是分组后对每组分别计算
        "score_stats": { // 聚合名称
          "stats": { // 聚合类型，这里stats可以计算min、max、avg等
            "field": "score" // 聚合字段，这里是score
          }
        }
      }
    }
  }
}
```


![image-20210723172917636.png](img/image-20210723172917636.png)

### 💻 RestAPI 实现聚合

![image-20210723173057733.png](img/image-20210723173057733.png)

聚合的结果也与查询结果不同，API也比较特殊。不过同样是 JSON 逐层解析：

![image-20210723173215728.png](img/image-20210723173215728.png)

代码实现：

```java
public void aggregationSearch() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    
    request.source().size(0);  // 不返回文档数据
    request.source().aggregation(
        AggregationBuilders
            .terms("brandAgg")  // 创建terms聚合
            .field("brand")     // 按brand字段聚合
            .size(10)           // 返回前10个
    );
    
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    
    Aggregations aggregations = response.getAggregations();
    Terms brandAgg = aggregations.get("brandAgg");  // 获取聚合结果
    
    // 遍历聚合桶
    for (Terms.Bucket bucket : brandAgg.getBuckets()) {
        String brand = bucket.getKeyAsString();  // 品牌名称
        long count = bucket.getDocCount();       // 该品牌的酒店数量
        System.out.println(brand + ": " + count);
    }
}
```


---

## 🔤 自动补全功能

### ⚙️ 自定义分词器

默认的拼音分词器会将每个汉字单独分为拼音，而我们希望的是每个词条形成一组拼音，需要对拼音分词器做个性化定制，形成自定义分词器。

Elasticsearch 中分词器（analyzer）的组成包含三部分：

- character filters：在 tokenizer 之前对文本进行处理。例如删除字符、替换字符
- tokenizer：将文本按照一定的规则切割成词条（term）。例如 keyword，就是不分词；还有 ik_smart
- tokenizer filter：将 tokenizer 输出的词条做进一步处理。例如大小写转换、同义词处理、拼音处理等

文档分词时会依次由这三部分来处理文档：

![image-20210723210427878.png](img/image-20210723210427878.png)

声明自定义分词器的语法如下：

```json
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { // 自定义分词器
        "my_analyzer": {  // 分词器名称
          "tokenizer": "ik_max_word",
          "filter": "py"
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", 
          "keep_full_pinyin": false, 
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer", # 保存文档内容时，使用自定义分词器-》写
        "search_analyzer": "ik_smart" # 搜索时使用id_smart ---》读
      }
    }
  }
}
```

| 参数名 | 说明 | 默认值 | 示例 |
|--------|------|--------|------|
| `keep_first_letter` | 将词的第一个字母全部拼起来 | true | 刘德华 → ldh |
| `keep_separate_first_letter` | 将第一个字母一个个分开 | false | 刘德华 → l,d,h |
| `limit_first_letter_length` | 设置最大 `keep_first_letter` 结果的长度 | 16 | - |
| `keep_full_pinyin` | 保存词的全拼，并按字分开保存 | true | 刘德华 → [liu,de,hua] |
| `keep_joined_full_pinyin` | 保存词的全拼（不分开） | false | 刘德华 → [liudehua] |
| `keep_none_chinese` | 将非中文字母或数字保留在结果中 | true | - |
| `keep_none_chinese_together` | 保证非中文在一起 | true | DJ音乐家 → DJ,yin,yue,jia |
| `keep_none_chinese_in_first_letter` | 将非中文字母保留在首字母中 | true | 刘德华AT2016 → ldhat2016 |
| `keep_none_chinese_in_joined_full_pinyin` | 将非中文字母保留为完整拼音 | false | 刘德华2016 → liudehua2016 |
| `none_chinese_pinyin_tokenize` | 如果是非中文，切分成单独的拼音项 | true | liudehuaalibaba13zhuanghan → liu,de,hua,a,li,ba,ba,13,zhuang,han |
| `keep_original` | 是否保持原词 | false | - |
| `lowercase` | 小写非中文字母 | true | - |
| `trim_whitespace` | 去掉空格 | true | - |
| `remove_duplicated_term` | 保存索引时删除重复的词语 | false | de的 → de |
| `ignore_pinyin_offset` | 忽略偏移量，允许使用重叠标记 | true | - |
### 🔍 自动补全查询

Elasticsearch 提供了 Completion Suggester 查询来实现自动补全功能。这个查询会匹配以用户输入内容开头的词条并返回。为了提高补全查询的效率，对于文档中字段的类型有一些约束：

- 参与补全查询的字段必须是 completion 类型。
- 字段的内容一般是用来补全的多个词条形成的数组。

```json
PUT test
{
  "mappings": {
    "properties": {
      "title":{
        "type": "completion"
      }
    }
  }
}
```


插入下面的数据：

```json
POST test/_doc
{
  "title": ["Sony", "WH-1000XM3"]
}
POST test/_doc
{
  "title": ["SK-II", "PITERA"]
}
POST test/_doc
{
  "title": ["Nintendo", "switch"]
}
```


查询的 DSL 语句如下：

```json
// 自动补全查询
GET /test/_search
{
  "suggest": {
    "title_suggest": {
      "text": "s", // 关键字
      "completion": {
        "field": "title", // 补全查询的字段
        "skip_duplicates": true, // 跳过重复的
        "size": 10 // 获取前10条结果
      }
    }
  }
}
```


### 💻 自动补全查询的 Java API

![image-20210723213759922.png](img/image-20210723213759922.png)
![image-20210723213917524.png](img/image-20210723213917524.png)

#### `Controller` 层添加接口

```java
@GetMapping("suggestion")
public List<String> getSuggestions(@RequestParam("key") String prefix) {
    return hotelService.getSuggestions(prefix);
}
```


#### `Service` 层添加业务逻辑

```java
    List<String> getSuggestions(String prefix);
```


#### `Service` 层实现业务逻辑

```java
@Override
public List<String> getSuggestions(String prefix) {
    try {
        // 1.准备Request
        SearchRequest request = new SearchRequest("hotel");
        // 2.准备DSL
        request.source().suggest(new SuggestBuilder().addSuggestion(
            "suggestions",
            SuggestBuilders.completionSuggestion("suggestion")
            .prefix(prefix)
            .skipDuplicates(true)
            .size(10)
        ));
        // 3.发起请求
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4.解析结果
        Suggest suggest = response.getSuggest();
        // 4.1.根据补全查询名称，获取补全结果
        CompletionSuggestion suggestions = suggest.getSuggestion("suggestions");
        // 4.2.获取options
        List<CompletionSuggestion.Entry.Option> options = suggestions.getOptions();
        // 4.3.遍历
        List<String> list = new ArrayList<>(options.size());
        for (CompletionSuggestion.Entry.Option option : options) {
            String text = option.getText().toString();
            list.add(text);
        }
        return list;
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```


---

## 🔁 数据同步机制

常见的数据同步方案有三种：

- 同步调用
- 异步通知
- 监听 binlog

### 🔄 同步调用

![image-20210723214931869.png](img/image-20210723214931869.png)

- 优点：实现简单，粗暴
- 缺点：业务耦合度高

### 📨 异步通知

![image-20210723215140735.png](img/image-20210723215140735.png)

- 优点：低耦合，实现难度一般
- 缺点：依赖 mq 的可靠性

### 📋 监听 binlog

![image-20210723215518541.png](img/image-20210723215518541.png)

流程如下：

- 给 mysql 开启 binlog 功能
- mysql 完成增、删、改操作都会记录在 binlog 中
- hotel-demo 基于 canal 监听 binlog 变化，实时更新 elasticsearch 中的内容

- 优点：完全解除服务间耦合
- 缺点：开启 binlog 增加数据库负担、实现复杂度高

### 🛠️ 实现数据同步

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```


#### 声明队列交换机名称

```java
public class MqConstants {
    /**
     * 交换机
     */
    public final static String HOTEL_EXCHANGE = "hotel.topic";
    /**
     * 监听新增和修改的队列
     */
    public final static String HOTEL_INSERT_QUEUE = "hotel.insert.queue";
    /**
     * 监听删除的队列
     */
    public final static String HOTEL_DELETE_QUEUE = "hotel.delete.queue";
    /**
     * 新增或修改的RoutingKey
     */
    public final static String HOTEL_INSERT_KEY = "hotel.insert";
    /**
     * 删除的RoutingKey
     */
    public final static String HOTEL_DELETE_KEY = "hotel.delete";
}
```


#### 声明队列交换机

```java
@Configuration
public class MqConfig {
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange(MqConstants.HOTEL_EXCHANGE, true, false);
    }

    @Bean
    public Queue insertQueue(){
        return new Queue(MqConstants.HOTEL_INSERT_QUEUE, true);
    }

    @Bean
    public Queue deleteQueue(){
        return new Queue(MqConstants.HOTEL_DELETE_QUEUE, true);
    }

    @Bean
    public Binding insertQueueBinding(){
        return BindingBuilder.bind(insertQueue()).to(topicExchange()).with(MqConstants.HOTEL_INSERT_KEY);
    }

    @Bean
    public Binding deleteQueueBinding(){
        return BindingBuilder.bind(deleteQueue()).to(topicExchange()).with(MqConstants.HOTEL_DELETE_KEY);
    }
}
```


#### 配置 MQ

```yaml
spring:
  rabbitmq:
    virtual-host: /
    port: 5672
    host: 192.168.88.128
    username: Qing
    password: 24364726
```


#### 发送 MQ 消息

![image-20210723221843816.png](img/image-20210723221843816.png)

#### 接收 MQ 消息

hotel-demo 接收到 MQ 消息要做的事情包括：

- 新增消息：根据传递的 hotel 的 id 查询 hotel 信息，然后新增一条数据到索引库
- 删除消息：根据传递的 hotel 的 id 删除索引库中的一条数据

1. `IHotelService` 中新增新增、删除业务

```java
void deleteById(Long id);

void insertById(Long id);
```


2. HotelService 中实现业务：

```java
@Override
public void deleteById(Long id) {
    try {
        // 1.准备Request
        DeleteRequest request = new DeleteRequest("hotel", id.toString());
        // 2.发送请求
        client.delete(request, RequestOptions.DEFAULT);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

@Override
public void insertById(Long id) {
    try {
        // 0.根据id查询酒店数据
        Hotel hotel = getById(id);
        // 转换为文档类型
        HotelDoc hotelDoc = new HotelDoc(hotel);

        // 1.准备Request对象
        IndexRequest request = new IndexRequest("hotel").id(hotel.getId().toString());
        // 2.准备Json文档
        request.source(JSON.toJSONString(hotelDoc), XContentType.JSON);
        // 3.发送请求
        client.index(request, RequestOptions.DEFAULT);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```


3. 编写监听器

```java
@Component
public class HotelListener {

    @Autowired
    private IHotelService hotelService;

    /**
     * 监听酒店新增或修改的业务
     * @param id 酒店id
     */
    @RabbitListener(queues = MqConstants.HOTEL_INSERT_QUEUE)
    public void listenHotelInsertOrUpdate(Long id){
        hotelService.insertById(id);
    }

    /**
     * 监听酒店删除的业务
     * @param id 酒店id
     */
    @RabbitListener(queues = MqConstants.HOTEL_DELETE_QUEUE)
    public void listenHotelDelete(Long id){
        hotelService.deleteById(id);
    }
}
```


---

## 🏢 集群概念详解

单机的 elasticsearch 做数据存储，必然面临两个问题：海量数据存储问题、单点故障问题。

- 海量数据存储问题：将索引库从逻辑上拆分为 N 个分片（shard），存储到多个节点
- 单点故障问题：将分片数据在不同节点备份（replica ）

**ES 集群相关概念**:

- 集群（cluster）：一组拥有共同的 cluster name 的 节点。
- <font color="red">节点（node)</font>：集群中的一个 Elasticearch 实例
- <font color="red">分片（shard）</font>：索引可以被拆分为不同的部分进行存储，称为分片。在集群环境下，一个索引的不同分片可以拆分到不同的节点中

  解决问题：数据量太大，单点存储量有限的问题。

![image-20200104124440086-5602723.png](img/image-20200104124440086-5602723.png)

- 主分片（Primary shard）：相对于副本分片的定义。
- 副本分片（Replica shard）每个主分片可以有一个或者多个副本，数据和主分片一样。

数据备份可以保证高可用，但是每个分片备份一份，所需要的节点数量就会翻一倍，成本实在是太高了！

为了在高可用和成本间寻求平衡，我们可以这样做：

- 首先对数据分片，存储到不同节点
- 然后对每个分片进行备份，放到对方节点，完成互相备份

这样可以大大减少所需要的服务节点数量，如图，我们以 3 分片，每个分片备份一份为例：

![image-20200104124551912.png](img/image-20200104124551912.png)

现在，每个分片都有 1 个备份，存储在 3 个节点：

- node0：保存了分片 0 和 1
- node1：保存了分片 0 和 2
- node2：保存了分片 1 和 2

### 🧠 集群脑裂问题

#### 🎯 集群职责划分

![image-20210723223008967.png](img/image-20210723223008967.png)

默认情况下，集群中的任何一个节点都同时具备上述四种角色。

但是真实的集群一定要将集群职责分离：

- master 节点：对 CPU 要求高，但是内存要求低
- data 节点：对 CPU 和内存要求都高
- coordinating 节点：对网络带宽、CPU 要求高

职责分离可以让我们根据不同节点的需求分配不同的硬件去部署。而且避免业务之间的互相干扰。

一个典型的 es 集群职责划分如图：

![image-20210723223629142.png](img/image-20210723223629142.png)

#### 🧩 脑裂问题

脑裂是因为集群中的节点失联导致的。

例如一个集群中，主节点与其它节点失联：

![image-20210723223804995.png](img/image-20210723223804995.png)

此时，node2 和 node3 认为 node1 宕机，就会重新选主：

![image-20210723223845754.png](img/image-20210723223845754.png)

当 node3 当选后，集群继续对外提供服务，node2 和 node3 自成集群，node1 自成集群，两个集群数据不同步，出现数据差异。

当网络恢复后，因为集群中有两个 master 节点，集群状态的不一致，出现脑裂的情况：

![image-20210723224000555.png](img/image-20210723224000555.png)

解决脑裂的方案是，要求选票超过 ( eligible 节点数量 + 1 ）/ 2 才能当选为主，因此 eligible 节点数量最好是奇数。对应配置项是 discovery.zen.minimum_master_nodes，在 es7.0 以后，已经成为默认配置，因此一般不会发生脑裂问题。

例如：3 个节点形成的集群，选票必须超过 （3 + 1） / 2 ，也就是 2 票。node3 得到 node2 和 node3 的选票，当选为主。node1 只有自己 1 票，没有当选。集群中依然只有 1 个主节点，没有出现脑裂。

### 🗃️ 集群分布式存储

#### 🧮 分片存储原理

- elasticsearch 会通过 hash 算法来计算文档应该存储到哪个分片

![image-20210723224354904.png](img/image-20210723224354904.png)

说明：

- _routing 默认是文档的 id
- 算法与分片数量有关，因此索引库一旦创建，分片数量不能修改！

新增文档的流程如下：

![image-20210723225436084.png](img/image-20210723225436084.png)

- 1）新增一个 id=1 的文档
- 2）对 id 做 hash 运算，假如得到的是 2，则应该存储到 shard-2
- 3）shard-2 的主分片在 node3 节点，将数据路由到 node3
- 4）保存文档
- 5）同步给 shard-2 的副本 replica-2，在 node2 节点
- 6）返回结果给 coordinating-node 节点

> 集群写入时，会先随机选取一个节点（node），该节点可以称之为"协调节点"。
>
> 新文档写入前，es 会对其 id 做 hash 取模，来确定该文档会分布在哪个分片上。
>
> 当分片位置确定好后，es 会判断当前"协调节点"上是否有该主分片。如果有，直接写；如果没有，则会将数据路由到包含该主分片的节点上。
>
> 整个写入过程是，es 会将文档先写入主分片上（如 p0），写完后再将数据同步一份到副本上（如 r0）
>
> 待副本数据也写完后，副本节点会通知协调节点，最后协调节点告知客户端，文档写入结束。

### 🔍 集群分布式查询

elasticsearch 的查询分成两个阶段：

- scatter phase：分散阶段，coordinating node 会把请求分发到每一个分片
- gather phase：聚集阶段，coordinating node 汇总 data node 的搜索结果，并处理为最终结果集返回给用户

![image-20210723225809848.png](img/image-20210723225809848.png)

### ⚠️ 集群故障转移

集群的 master 节点会监控集群中的节点状态，如果发现有节点宕机，会立即将宕机节点的分片数据迁移到其它节点，确保数据安全，这个叫做故障转移。

1. 例如一个集群结构如图：

![image-20210723225945963](img/image-20210723225945963.png)

现在，node1 是主节点，其它两个节点是从节点。

2. 突然，node1 发生了故障：

![image-20210723230020574](img/image-20210723230020574.png)

宕机后的第一件事，需要重新选主，例如选中了 node2：

![image-20210723230055974](img/image-20210723230055974.png)

node2 成为主节点后，会检测集群监控状态，发现：shard-1、shard-0 没有副本节点。因此需要将 node1 上的数据迁移到 node2、node3：

![image-20210723230216642](img/image-20210723230216642.png)