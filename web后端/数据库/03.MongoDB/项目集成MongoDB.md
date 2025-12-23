# 快速集成MongoDB 🚀

## 1.引入依赖 📦

```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```


## 2.配置相关属性 ⚙️

```yaml
spring:
  data:
    mongodb:
      host: 192.168.200.130
      username: root
      password: <PASSWORD>
      # 数据库
      database: articledb
      # 默认端口是27017
      port: 27017
      #也可以使用uri连接
      #uri: mongodb://192.168.40.134:27017/articledb
```


## 3.实体类的编写 📝

```java
//把一个java类声明为mongodb的文档，可以通过collection参数指定这个类对应的文档。
//@Document(collection="mongodb 对应 collection 名")
// 若未加 @Document ，该 bean save 到 mongo 的 comment collection
// 若添加 @Document ，则 save 到 comment collection
@Document(collection="comment")//可以省略，如果省略，则默认使用类名小写映射集合
//复合索引
// @CompoundIndex( def = "{'userid': 1, 'nickname': -1}")
@Data
public class Comment implements Serializable {
    @Id
    private String id;//主键
    //该属性对应mongodb的字段的名字，如果一致，则无需该注解
    @Field("content")
    private String content;//吐槽内容
    private Date publishtime;//发布日期
    //添加了一个单字段的索引
    @Indexed
    private String userid;//发布人ID
    private String nickname;//昵称
    private LocalDateTime createdatetime;//评论的日期时间
    private Integer likenum;//点赞数
    private Integer replynum;//回复数
    private String state;//状态
    private String parentid;//上级ID
    private String articleid;
}
```


- 单字段索引注解 `@Indexed`
  - `org.springframework.data.mongodb.core.index.Indexed.class`
  - 声明该字段需要索引，建索引可以大大的提高查询效率。
    ```sql
    db.comment.createIndex(
        {"userid":1}
    )
    ```


- 复合索引注解 `@CompoundIndex`
  - `org.springframework.data.mongodb.core.index.CompoundIndex.class`
  - 复合索引的声明，建复合索引可以有效地提高多字段的查询效率。
  ```sql
  db.comment.createIndex(
    {"userid":1,"nickname":-1}
  )
  ```



## 4.持久层接口类编写 🗃️

```java
@Repository
public interface CommentRepository extends MongoRepository<Comment, String> {
    
}
```


## 5.创建业务逻辑类 🏗️

```java
@Service
public class CommentService {
    
    @Autowired
    private CommentRepository commentRepository;
    
    public void saveComment(Comment comment){
        //如果需要自定义主键，可以在这里指定主键；如果不指定主键，MongoDB会自动生成主键
        //设置一些默认初始值。。。
        commentRepository.save(comment);
    }

    public void updateComment(Comment comment){
        //调用dao
        commentRepository.save(comment);
    }
    
    public void deleteCommentById(String id){
        //调用dao
        commentRepository.deleteById(id);
    }
    
    public List<Comment> findCommentList(){
        //调用dao
        return commentRepository.findAll();
    }

    public Comment findCommentById(String id){
        return commentRepository.findById(id).get();
    }
}
```


--- 

## 根据上级ID查询文章评论的分页列表 📄

### `CommentRepository` 新增方法定义

```java
    //根据父id，查询子评论的分页列表
    Page<Comment> findByParentid(String parentid, Pageable pageable);
```


### `CommentService` 新增方法

```java
  public Page<Comment> findCommentListPageByParentid(String parentid,int page ,int size){
        return commentRepository.findByParentid(parentid, PageRequest.of(page-1,size));
  }
```


### junit测试用例 🧪

```java
  @Test
  public void testFindCommentListPageByParentid(){
      Page<Comment> pageResponse = commentService.findCommentListPageByParentid("3", 1, 2);
      System.out.println("----总记录数："+pageResponse.getTotalElements());
      System.out.println("----当前页数据："+pageResponse.getContent());
  }
```


## MongoTemplate 实现评论点赞 👍

```java
    @Autowired
    private MongoTemplate mongoTemplate;
  
    public void updateCommentLikenum(String id) {
        Query query = Query.query(Criteria.where("_id").is(id));
        Update update = new Update();
        //局部更新，相当于$set
        update.set(key, value)
        //递增$inc
        update.inc("likenum", 1);
        update.inc("likenum");

        //参数1：查询对象
        //参数2：更新对象
        //参数3：集合的名字或实体类的类型Comment.class
        mongoTemplate.updateFirst(query, update, "comment");
    }
```
