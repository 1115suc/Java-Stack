# 📊 Elasticsearch 进阶学习笔记

## 🔍 DSL 查询文档详解

### 📚 DSL 查询分类说明
DSL（Domain Specific Language）是 Elasticsearch 提供的 JSON 风格查询语言。

- **查询所有**：查询出所有数据，一般测试用。例如：`match_all`
- **全文检索（full text）查询**：利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如：
    - `match_query`
    - `multi_match_query`
- **精确查询**：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段。例如：
    - `ids`
    - `range`
    - `term`
- **地理（geo）查询**：根据经纬度查询。例如：
    - `geo_distance`
    - `geo_bounding_box`
- **复合（compound）查询**：复合查询可以将上述各种查询条件组合起来，合并查询条件。例如：
    - `bool`
    - `function_score`

### 🔤 全文检索查询

#### match 查询
用于全文检索，会对查询内容进行分词后再匹配：

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "all": "外滩如家"  // 会分词为"外滩"和"如家"分别匹配
    }
  }
}
```


#### multi_match 查询
在多个字段中进行全文检索：

```json
GET /hotel/_search
{
  "query": {
    "multi_match": {
      "query": "外滩如家",
      "fields": ["name", "business"]  // 在name和business字段中搜索
    }
  }
}
```


### 🎯 精准查询

#### term 查询
用于精确匹配，不对查询内容分词：

```json
GET /hotel/_search
{
  "query": {
    "term": {
      "city": {
        "value": "上海"  // 精确匹配"上海"，不会匹配"上海市"
      }
    }
  }
}
```


#### range 查询
用于范围查询：

```json
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100,  // 大于等于100
        "lte": 300   // 小于等于300
      }
    }
  }
}
```


### 🌍 地理坐标查询

#### 矩形范围查询
![DKV9HZbVS6.gif](img/DKV9HZbVS6.gif)

```json
GET hotel/_search
{
  "query":{
    "geo_bounding_box":{
      "location":{
        "top_left": {
          "lat": 31.1,
          "lon": 121.5
        },
        "bottom_right":{
          "lat": 30.9,
          "lon": 121.7
        }
      }
    }
  }
}
```


#### 附近查询
![vZrdKAh19C.gif](img/vZrdKAh19C.gif)

```json
GET /hotel/_search
{
  "query": {
    "geo_distance": { 
      "distance": "15km", // 半径
      "location": "31.21,121.5" // 中心点
    }
  }
}
```


### 🔗 复合查询

#### bool 查询
布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有：
 
- `must`：必须匹配每个子查询，类似"与"
- `should`：选择性匹配子查询，类似"或"
- `must_not`：必须不匹配，**不参与算分**，类似"非"
- `filter`：必须匹配，**不参与算分**

```json
GET /hotel/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"city": "上海" }}
      ],
      "should": [
        {"term": {"brand": "皇冠假日" }},
        {"term": {"brand": "华美达" }}
      ],
      "must_not": [
        { "range": { "price": { "lte": 500 } }}
      ],
      "filter": [
        { "range": {"score": { "gte": 45 } }}
      ]
    }
  }
}
```


#### 📊 相关性算分

当我们利用match查询时，文档结果会根据与搜索词条的关联度打分（`_score`），返回结果时按照分值降序排列。

```json
[
  {
    "_score" : 17.850193,
    "_source" : {
      "name" : "虹桥如家酒店真不错",
    }
  },
  {
    "_score" : 12.259849,
    "_source" : {
      "name" : "外滩如家酒店真不错",
    }
  },
  {
    "_score" : 11.91091,
    "_source" : {
      "name" : "迪士尼如家酒店真不错",
    }
  }
]
```


在Elasticsearch中，早期使用的打分算法是TF-IDF算法，公式如下：
![image-20210721190152134.png](img/image-20210721190152134.png)

在后来的5.1版本升级中，Elasticsearch将算法改进为BM25算法，公式如下：
![image-20210721190416214.png](img/image-20210721190416214.png)

TF-IDF算法有一个缺陷，就是词条频率越高，文档得分也会越高，单个词条对文档影响较大。而BM25则会让单个词条的算分有一个上限，曲线更加平滑：
![image-20210721190907320.png](img/image-20210721190907320.png)

#### 算分函数查询
![image-20210721191544750.png](img/image-20210721191544750.png)

`function_score` 查询中包含四部分内容：

- **原始查询**条件：`query`部分，基于这个条件搜索文档，并且基于BM25算法给文档打分，**原始算分**（`query score`)
- **过滤条件**：`filter`部分，符合该条件的文档才会重新算分
- **算分函数**：符合`filter`条件的文档要根据这个函数做运算，得到的**函数算分**（`function score`），有四种函数
    - `weight`：函数结果是常量
    - `field_value_factor`：以文档中的某个字段值作为函数结果
    - `random_score`：以随机数作为函数结果
    - `script_score`：自定义算分函数算法
- **运算模式**：算分函数的结果、原始查询的相关性算分，两者之间的运算方式，包括：
    - `multiply`：相乘
    - `replace`：用function score替换query score
    - 其它，例如：`sum`、`avg`、`max`、`min`

`function_score`的运行流程如下：

1. 根据**原始条件**查询搜索文档，并且计算相关性算分，称为**原始算分**（`query score`）
2. 根据**过滤条件**，过滤文档
3. 符合**过滤条件**的文档，基于**算分函数**运算，得到**函数算分**（`function score`）
4. 将**原始算分**（`query score`）和**函数算分**（`function score`）基于**运算模式**做运算，得到最终结果，作为相关性算分。

因此，其中的关键点是：

- 过滤条件：决定哪些文档的算分被修改
- 算分函数：决定函数算分的算法
- 运算模式：决定最终算分结果

##### 示例

需求：给"如家"这个品牌的酒店排名靠前一些

翻译一下这个需求，转换为之前说的四个要点：

- 原始条件：不确定，可以任意变化
- 过滤条件：`brand = "如家"`
- 算分函数：可以简单粗暴，直接给固定的算分结果，`weight`
- 运算模式：比如求和

因此最终的DSL语句如下：

```json
GET /hotel/_search
{
  "query": {
    "function_score": {
      "query": {  .... }, // 原始查询，可以是任意条件
      "functions": [ // 算分函数
        {
          "filter": { // 满足的条件，品牌必须是如家
            "term": {
              "brand": "如家"
            }
          },
          "weight": 2 // 算分权重为2
        }
      ],
      "boost_mode": "sum" // 加权模式，求和
    }
  }
}
```


## 📊 搜索结果处理

### 📈 排序
```json
GET /hotel/_search
{
  "query": {
    "match_all": {}  // 匹配所有文档
  },
  "sort": [
    {
      "price": {
        "order": "asc"  // 排序字段、排序方式ASC、DESC
      },
      "score": {
        "order": "desc"
      }
    }
  ]
}
```
```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance" : {
          "FIELD" : "纬度，经度", // 文档中geo_point类型的字段名、目标坐标点
          "order" : "asc", // 排序方式
          "unit" : "km" // 排序的距离单位
      }
    }
  ]
}
```


### 📄 分页
```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,    // 起始位置（从0开始）
  "size": 10    // 返回条数
}
```


#### 深度分页问题

现在，我要查询990~1000的数据，查询逻辑要这么写：
```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "from": 990, // 分页开始的位置，默认为0
  "size": 10, // 期望获取的文档总数
  "sort": [
    {"price": "asc"}
  ]
}
```


这里是查询990开始的数据，也就是 第990~第1000条 数据。

不过，elasticsearch内部分页时，必须先查询 0~1000条，然后截取其中的990 ~ 1000的这10条：
![image-20210721200643029.png](img/image-20210721200643029.png)
查询TOP1000，如果es是单点模式，这并无太大影响。

但是elasticsearch将来一定是集群，例如我集群有5个节点，我要查询TOP1000的数据，并不是每个节点查询200条就可以了。

因为节点A的TOP200，在另一个节点可能排到10000名以外了。

因此要想获取整个集群的TOP1000，必须先查询出每个节点的TOP1000，汇总结果后，重新排名，重新截取TOP1000。
![image-20210721201003229.png](img/image-20210721201003229.png)

那如果我要查询9900~10000的数据呢？是不是要先查询TOP10000呢？那每个节点都要查询10000条？汇总到内存中？

当查询分页深度较大时，汇总数据过多，对内存和CPU会产生非常大的压力，因此elasticsearch会禁止from+ size 超过10000的请求。

```json
GET hotel/_search
{
  "query": {
    "match": {
      "all": "外滩如家"
    }
  },
  "size": 3, 
  "search_after": [379, "433576"],
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    },
    {
      "id": {
        "order": "asc"
      }
    }
  ]
}
```


针对深度分页，ES提供了两种解决方案，[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)：

- `search after`：分页时需要排序，原理是从上一次的排序值开始，查询下一页数据。官方推荐使用的方式。
- `scroll`：原理将排序后的文档id形成快照，保存在内存。官方已经不推荐使用。

### ✨ 高亮显示
```json
GET /hotel/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT" // 查询条件，高亮一定要使用全文检索查询
    }
  },
  "highlight": {
    "fields": { // 指定要高亮的字段
      "FIELD": {
        "pre_tags": "<em>",  // 用来标记高亮字段的前置标签
        "post_tags": "</em>" // 用来标记高亮字段的后置标签
      }
    }
  }
}
```


**注意：**

- 高亮是对关键字高亮，因此**搜索条件必须带有关键字**，而不能是范围这样的查询。
- 默认情况下，**高亮的字段，必须与搜索指定的字段一致**，否则无法高亮
- 如果要对非搜索字段高亮，则需要添加一个属性：`required_field_match=false`

### 📝 总结

查询的DSL是一个大的JSON对象，包含下列属性：

- `query`：查询条件
- `from`和`size`：分页条件
- `sort`：排序条件
- `highlight`：高亮条件

![image-20210721203657850.png](img/image-20210721203657850.png)

## 🧰 RestClient 查询文档

### 📦 响应结果结构说明
elasticsearch返回的结果是一个JSON字符串，结构包含：

- `hits`：命中的结果
    - `total`：总条数，其中的value是具体的总条数值
    - `max_score`：所有结果中得分最高的文档的相关性算分
    - `hits`：搜索结果的文档数组，其中的每个文档都是一个json对象
        - `_source`：文档中的原始数据，也是json对象

因此，我们解析响应结果，就是逐层解析JSON字符串，流程如下：

- `SearchHits`：通过`response.getHits()`获取，就是JSON中的最外层的`hits`，代表命中的结果
    - `SearchHits#getTotalHits().value`：获取总条数信息
    - `SearchHits#getHits()`：获取`SearchHit`数组，也就是文档数组
        - `SearchHit#getSourceAsString()`：获取文档结果中的`_source`，也就是原始的json文档数据

### 📦 添加依赖说明
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>7.12.1</version>
</dependency>
```


### 🔍 基本查询示例详解
```java
@Test
void testMatchAll() throws IOException {
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    request.source()
            .query(QueryBuilders.matchAllQuery());
    // 3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    // 4.解析响应
    handleResponse(response);
}

private void handleResponse(SearchResponse response) {
    // 4.解析响应
    SearchHits searchHits = response.getHits();
    // 4.1.获取总条数
    long total = searchHits.getTotalHits().value;
    System.out.println("共搜索到" + total + "条数据");
    // 4.2.文档数组
    SearchHit[] hits = searchHits.getHits();
    // 4.3.遍历
    for (SearchHit hit : hits) {
        // 获取文档source
        String json = hit.getSourceAsString();
        // 反序列化
        HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
        System.out.println("hotelDoc = " + hotelDoc);
    }
}
```


### 🔤 match查询
```java
@Test
void testMatch() throws IOException {
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    request.source()
        .query(QueryBuilders.matchQuery("all", "如家"));
    // 3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    handleResponse(response);
}
```


### 🎯 精确查询
精确查询主要是两者：

- term：词条精确匹配
- range：范围查询

与之前的查询相比，差异同样在查询条件，其它都一样。

查询条件构造的API如下：
![image-20210721220305140.png](img/image-20210721220305140.png)

### 🔗 布尔查询

布尔查询是用must、must_not、filter等方式组合其它查询，代码示例如下：
![image-20210721220927286.png](img/image-20210721220927286.png)

```java
@Test
void testBool() throws IOException {
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    // 2.1.准备BooleanQuery
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
    // 2.2.添加term
    boolQuery.must(QueryBuilders.termQuery("city", "杭州"));
    // 2.3.添加range
    boolQuery.filter(QueryBuilders.rangeQuery("price").lte(250));

    request.source().query(boolQuery);
    // 3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    handleResponse(response);
}
```


### 📊 排序、分页
搜索结果的排序和分页是与query同级的参数，因此同样是使用`request.source()`来设置。

对应的API如下：
![image-20210721221121266.png](img/image-20210721221121266.png)

```java
@Test
void testPageAndSort() throws IOException {
    // 页码，每页大小
    int page = 1, size = 5;
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    // 2.1.query
    request.source().query(QueryBuilders.matchAllQuery());
    // 2.2.排序 sort
    request.source().sort("price", SortOrder.ASC);
    // 2.3.分页 from、size
    request.source().from((page - 1) * size).size(5);
    // 3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    handleResponse(response);
}
```


### ✨ 高亮

高亮的代码与之前代码差异较大，有两点：

- 查询的DSL：其中除了查询条件，还需要添加高亮条件，同样是与query同级。
- 结果解析：结果除了要解析`_source`文档数据，还要解析高亮结果

高亮请求的构建API如下：
![image-20210721221744883.png](img/image-20210721221744883.png)

```java
@Test
void testHighlight() throws IOException {
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    // 2.1.query
    request.source().query(QueryBuilders.matchQuery("all", "如家"));
    // 2.2.高亮
    request.source().highlighter(new HighlightBuilder().field("name").requireFieldMatch(false));
    // 3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    handleResponse(response);
}
```


#### 高亮结果解析

高亮的结果与查询的文档结果默认是分离的，并不在一起。

因此解析高亮的代码需要额外处理：
![image-20210721222057212.png](img/image-20210721222057212.png)

代码解读：

- 第一步：从结果中获取source。`hit.getSourceAsString()`，这部分是非高亮结果，json字符串。还需要反序列为`HotelDoc`对象
- 第二步：获取高亮结果。`hit.getHighlightFields()`，返回值是一个Map，key是高亮字段名称，值是`HighlightField`对象，代表高亮值
- 第三步：从map中根据高亮字段名称，获取高亮字段值对象`HighlightField`
- 第四步：从`HighlightField`中获取`Fragments`，并且转为字符串。这部分就是真正的高亮字符串了
- 第五步：用高亮的结果替换`HotelDoc`中的非高亮结果

```java
private void handleResponse(SearchResponse response) {
    // 4.解析响应
    SearchHits searchHits = response.getHits();
    // 4.1.获取总条数
    long total = searchHits.getTotalHits().value;
    System.out.println("共搜索到" + total + "条数据");
    // 4.2.文档数组
    SearchHit[] hits = searchHits.getHits();
    // 4.3.遍历
    for (SearchHit hit : hits) {
        // 获取文档source
        String json = hit.getSourceAsString();
        // 反序列化
        HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
        // 获取高亮结果
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        if (!CollectionUtils.isEmpty(highlightFields)) {
            // 根据字段名获取高亮结果
            HighlightField highlightField = highlightFields.get("name");
            if (highlightField != null) {
                // 获取高亮值
                String name = highlightField.getFragments()[0].string();
                // 覆盖非高亮结果
                hotelDoc.setName(name);
            }
        }
        System.out.println("hotelDoc = " + hotelDoc);
    }
}
```