## 📈 数据聚合详解

### DSL 实现聚合

#### 按品牌分组统计
```json
GET /hotel/_search
{
  "size": 0,  // 不返回具体文档，只返回聚合结果
  "aggs": {
    "brandAgg": {  // 聚合名称
      "terms": {
        "field": "brand",  // 按brand字段分组
        "size": 10         // 返回前10个分组
      }
    }
  }
}
```


#### 嵌套聚合（带指标）
```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 10
      },
      "aggs": {  // 嵌套聚合
        "avgPrice": {  // 计算平均价格
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```


### RestAPI 实现聚合
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

## 🔤 自动补全详解

### 定义自动补全映射
```json
PUT /hotel
{
  "mappings": {
    "properties": {
      "suggestion": {
        "type": "completion",      // completion类型用于自动补全
        "analyzer": "ik_max_word"  // 使用IK分词器
      }
      // 其他字段...
    }
  }
}
```


### 自动补全查询
```json
GET /hotel/_search
{
  "suggest": {
    "suggestions": {  // 建议名称
      "text": "如家",   // 用户输入的文本
      "completion": {
        "field": "suggestion",     // 自动补全字段
        "skip_duplicates": true,   // 跳过重复项
        "size": 10                 // 返回数量
      }
    }
  }
}
```



## 🔁 数据同步详解

### 监听数据库变更
使用消息队列实现数据库与 Elasticsearch 的数据同步：

```java
@Service
public class HotelDataService {

@Autowired
private ElasticsearchRestTemplate elasticsearchRestTemplate;

// 监听数据库插入操作
@RabbitListener(queues = "hotel.insert.queue")
public void handleHotelInsert(Long id) {
// 查询数据库获取最新数据
Hotel hotel = hotelService.getById(id);

// 转换为文档对象
HotelDoc hotelDoc = new HotelDoc(hotel);

// 写入 ES
elasticsearchRestTemplate.save(hotelDoc);
}

// 监听数据库更新操作
@RabbitListener(queues = "hotel.update.queue")
public void handleHotelUpdate(Long id) {
// 查询数据库获取最新数据
Hotel hotel = hotelService.getById(id);

// 转换为文档对象
HotelDoc hotelDoc = new HotelDoc(hotel);

// 更新 ES
elasticsearchRestTemplate.save(hotelDoc);
}

// 监听数据库删除操作
@RabbitListener(queues = "hotel.delete.queue")
public void handleHotelDelete(Long id) {
// 从 ES 删除
elasticsearchRestTemplate.delete(id.toString(), HotelDoc.class);
}
}
```



## 🏢 集群概念详解

### 集群基本概念说明
- **Cluster**: Elasticsearch 集群，包含多个节点，提供高可用性和扩展性
- **Node**: 集群中的一个 Elasticsearch 实例，可以是主节点、数据节点等角色
- **Shard**: 索引的分片，用于水平拆分数据，提高性能和扩展性
- **Replica**: 分片的副本，用于提供高可用性和读取性能

### 集群架构优势说明
1. **高可用性**: 数据有多份副本，当某个节点故障时，其他节点可以继续提供服务
2. **横向扩展**: 可以添加更多节点来处理更大的负载和数据量
3. **负载均衡**: 请求可以在集群中的节点之间分布，避免单点压力过大
4. **性能提升**: 并行处理查询和索引操作，提高整体性能

### 分片和副本说明
- **主分片**: 包含实际数据的分片，每个文档只存在于一个主分片中
- **副本分片**: 主分片的拷贝，提供冗余和查询性能
- 默认情况下，每个索引有 1 个主分片和 1 个副本分片，可以根据需求调整