## 🚀 Elasticsearch 总结

### 📚 索引库(Index)操作

#### 基本操作命令
- **创建索引库**：`PUT /索引库名`
- **查询索引库**：`GET /索引库名`
- **删除索引库**：`DELETE /索引库名`
- **添加字段**：`PUT /索引库名/_mapping`

#### 操作基本步骤
1. 初始化 `RestHighLevelClient`
2. 创建 `XxxIndexRequest`（`Xxx` 为 `Create`、`Get`、`Delete`）
3. 准备 DSL（仅 `Create` 需要，其他无参）
4. 发送请求：调用 `RestHighLevelClient.indices().xxx()` 方法
    - `xxx` 对应 `create`、`exists`、`delete`

### 📄 文档(Document)操作

#### 基本操作命令
- **创建文档**：`POST /{索引库名}/_doc/文档id { json文档 }`
- **查询文档**：`GET /{索引库名}/_doc/文档id`
- **删除文档**：`DELETE /{索引库名}/_doc/文档id`
- **修改文档**：
    - *全量修改*：`PUT /{索引库名}/_doc/文档id { json文档 }`
    - *增量修改*：`POST /{索引库名}/_update/文档id { "doc": {字段}}`

#### 操作基本步骤
1. 初始化 `RestHighLevelClient`
2. 创建 `XxxRequest`（`Xxx` 为 `Index`、`Get`、`Update`、`Delete`、`Bulk`）
3. 准备参数（`Index`、`Update`、`Bulk` 时需要）
4. 发送请求：调用 `RestHighLevelClient.xxx()` 方法
5. 解析结果（`Get` 时需要）

> 💡 **提示**：注意区分索引库操作和文档操作使用的客户端方法不同哦！


### 📝 总结

查询的DSL是一个大的JSON对象，包含下列属性：

- `query`：查询条件
- `from`和`size`：分页条件
- `sort`：排序条件
- `highlight`：高亮条件

![image-20210721203657850.png](img/image-20210721203657850.png)
