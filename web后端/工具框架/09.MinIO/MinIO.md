# 📦 MinIO 对象存储

## 🔍 MinIO 介绍

- MinIO基于Apache License v2.0开源协议的对象存储服务，可以做为云存储的解决方案用来保存海量的图片，视频，文档。由于采用Golang实现，服务端可以工作在Windows,Linux, OS X和FreeBSD上。配置简单，基本是复制可执行程序，单行命令可以运行起来。
- MinIO兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。

### 🌐 S3 （ Simple Storage Service简单存储服务）
- bucket – 类比于文件系统的目录
- Object – 类比文件系统的文件
- Keys – 类比文件名

## ⭐ MinIO 特点
- **数据保护**
  - Minio使用Minio Erasure Code（纠删码）来防止硬件故障。即便损坏一半以上的driver，但是仍然可以从中恢复。

- **高性能**
  - 作为高性能对象存储，在标准硬件条件下它能达到55GB/s的读、35GB/s的写速率

- **可扩容**
  - 不同MinIO集群可以组成联邦，并形成一个全局的命名空间，并跨越多个数据中心

- **SDK支持**
  - 基于Minio轻量的特点，它得到类似Java、Python或Go等语言的sdk支持

- **有操作页面**
  - 面向用户友好的简单操作界面，非常方便的管理Bucket及里面的文件资源

- **功能简单**
  - 这一设计原则让MinIO不容易出错、更快启动

- **丰富的API**
  - 支持文件资源的分享连接及分享链接的过期策略、存储桶操作、文件列表访问及文件上传下载的基本功能等。

- **文件变化主动通知**
  - 存储桶（Bucket）如果发生改变,比如上传对象和删除对象，可以使用存储桶事件通知机制进行监控，并通过以下方式发布出去:AMQP、MQTT、Elasticsearch、Redis、NATS、MySQL、Kafka、Webhooks等。

## 🚀 MinIO 安装

### 1️⃣ Docker 部署镜像
```shell
docker run -p 9000:9000 --name minio -d \
       --restart=always \
       -e "MINIO_ACCESS_KEY=minio" \
       -e "MINIO_SECRET_KEY=24364726" \
       -v /home/data:/data \
       -v /home/config:/root/.minio minio/minio \
       server /data
```


> 通过访问 http://127.0.0.1:9000 访问控制台

![image-20210417102204739](img/image-20210417102204739.png)

![image-20210417102356582](img/image-20210417102356582.png)

### 2️⃣ 引入依赖

```xml
<dependency>
   <groupId>io.minio</groupId>
   <artifactId>minio</artifactId>
   <version>7.1.0</version>
</dependency>
```


### 3️⃣ 创建测试类
```java
public class MinIOTest {
    public static void main(String[] args) {
        FileInputStream fileInputStream = null;
        try {
            fileInputStream =  new FileInputStream("D:\\list.html");;

            //1.创建minio链接客户端
            MinioClient minioClient = MinioClient.builder()
                    .credentials("minio", "24364726")
                    .endpoint("http://192.168.200.130:9000")
                    .build();
            //2.上传
            PutObjectArgs putObjectArgs = PutObjectArgs.builder()
                    .object("list.html")//文件名
                    .contentType("text/html")//文件类型
                    .bucket("leadnews")//桶名词  与minio创建的名词一致
                    .stream(fileInputStream, fileInputStream.available(), -1) //文件流
                    .build();
            minioClient.putObject(putObjectArgs);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```


## 🔧 封装 MinIO 为 starter

### 1️⃣ 创建模块 MinIO-spring-boot--starter

### 2️⃣ 导入依赖
```xml
<dependencies>
    <dependency>
        <groupId>io.minio</groupId>
        <artifactId>minio</artifactId>
        <version>7.1.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```


### 3️⃣ 创建配置类 MinIOConfigProperties 和 MinIOConfig
```java
@Data
@ConfigurationProperties(prefix = "minio")  // 文件上传 配置前缀file.oss
public class MinIOConfigProperties implements Serializable {
    private String accessKey;
    private String secretKey;
    private String bucket;
    private String endpoint;
    private String readPath;
}
```
```java
@Data
@Configuration
@EnableConfigurationProperties({MinIOConfigProperties.class})
//当引入FileStorageService接口时
@ConditionalOnClass(FileStorageService.class)
public class MinIOConfig {
   @Autowired
   private MinIOConfigProperties minIOConfigProperties;

    @Bean
    public MinioClient buildMinioClient(){
        return MinioClient
                .builder()
                .credentials(minIOConfigProperties.getAccessKey(), minIOConfigProperties.getSecretKey())
                .endpoint(minIOConfigProperties.getEndpoint()) // 设置MinIO服务的访问地址
                .build();
    }
}
```


### 4️⃣ 封装操作类 MinIO
```java
public interface FileStorageService {
    
    public String uploadImgFile(String prefix, String filename,InputStream inputStream);
    
    public String uploadHtmlFile(String prefix, String filename,InputStream inputStream);
    
    public void delete(String pathUrl);
    
    public byte[]  downLoadFile(String pathUrl);
}
```
```java
@Slf4j
@EnableConfigurationProperties(MinIOConfigProperties.class)
@Import(MinIOConfig.class)
public class MinIOFileStorageService implements FileStorageService {

    @Autowired
    private MinioClient minioClient;

    @Autowired
    private MinIOConfigProperties minIOConfigProperties;

    private final static String separator = "/";

    /**
     * @param dirPath
     * @param filename  yyyy/mm/dd/file.jpg
     * @return
     */
    public String builderFilePath(String dirPath,String filename) {
        StringBuilder stringBuilder = new StringBuilder(50);
        if(!StringUtils.isEmpty(dirPath)){
            stringBuilder.append(dirPath).append(separator);
        }
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
        String todayStr = sdf.format(new Date());
        stringBuilder.append(todayStr).append(separator);
        stringBuilder.append(filename);
        return stringBuilder.toString();
    }

    /**
     *  上传图片文件
     * @param prefix  文件前缀
     * @param filename  文件名
     * @param inputStream 文件流
     * @return  文件全路径
     */
    @Override
    public String uploadImgFile(String prefix, String filename,InputStream inputStream) {
        String filePath = builderFilePath(prefix, filename);
        try {
            PutObjectArgs putObjectArgs = PutObjectArgs.builder()
                    .object(filePath)
                    .contentType("image/jpg")
                    .bucket(minIOConfigProperties.getBucket()).stream(inputStream,inputStream.available(),-1)
                    .build();
            minioClient.putObject(putObjectArgs);
            StringBuilder urlPath = new StringBuilder(minIOConfigProperties.getReadPath());
            urlPath.append(separator+minIOConfigProperties.getBucket());
            urlPath.append(separator);
            urlPath.append(filePath);
            return urlPath.toString();
        }catch (Exception ex){
            log.error("minio put file error.",ex);
            throw new RuntimeException("上传文件失败");
        }
    }

    /**
     *  上传html文件
     * @param prefix  文件前缀
     * @param filename   文件名
     * @param inputStream  文件流
     * @return  文件全路径
     */
    @Override
    public String uploadHtmlFile(String prefix, String filename,InputStream inputStream) {
        String filePath = builderFilePath(prefix, filename);
        try {
            PutObjectArgs putObjectArgs = PutObjectArgs.builder()
                    .object(filePath)
                    .contentType("text/html")
                    .bucket(minIOConfigProperties.getBucket()).stream(inputStream,inputStream.available(),-1)
                    .build();
            minioClient.putObject(putObjectArgs);
            StringBuilder urlPath = new StringBuilder(minIOConfigProperties.getReadPath());
            urlPath.append(separator+minIOConfigProperties.getBucket());
            urlPath.append(separator);
            urlPath.append(filePath);
            return urlPath.toString();
        }catch (Exception ex){
            log.error("minio put file error.",ex);
            ex.printStackTrace();
            throw new RuntimeException("上传文件失败");
        }
    }

    /**
     * 删除文件
     * @param pathUrl  文件全路径
     */
    @Override
    public void delete(String pathUrl) {
        String key = pathUrl.replace(minIOConfigProperties.getEndpoint()+"/","");
        int index = key.indexOf(separator);
        String bucket = key.substring(0,index);
        String filePath = key.substring(index+1);
        // 删除Objects
        RemoveObjectArgs removeObjectArgs = RemoveObjectArgs.builder().bucket(bucket).object(filePath).build();
        try {
            minioClient.removeObject(removeObjectArgs);
        } catch (Exception e) {
            log.error("minio remove file error.  pathUrl:{}",pathUrl);
            e.printStackTrace();
        }
    }
    
    /**
     * 下载文件
     * @param pathUrl  文件全路径
     * @return  文件流
     *
     */
    @Override
    public byte[] downLoadFile(String pathUrl)  {
        String key = pathUrl.replace(minIOConfigProperties.getEndpoint()+"/","");
        int index = key.indexOf(separator);
        String bucket = key.substring(0,index);
        String filePath = key.substring(index+1);
        InputStream inputStream = null;
        try {
            inputStream = minioClient.getObject(GetObjectArgs.builder().bucket(minIOConfigProperties.getBucket()).object(filePath).build());
        } catch (Exception e) {
            log.error("minio down file error.  pathUrl:{}",pathUrl);
            e.printStackTrace();
        }

        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        byte[] buff = new byte[100];
        int rc = 0;
        while (true) {
            try {
                if (!((rc = inputStream.read(buff, 0, 100)) > 0)) break;
            } catch (IOException e) {
                e.printStackTrace();
            }
            byteArrayOutputStream.write(buff, 0, rc);
        }
        return byteArrayOutputStream.toByteArray();
    }
}
```


### 5️⃣ 对外加入自动配置
- 在resources下创建`META-INF/spring.factories`

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.file.service.impl.MinIOFileStorageService
```


### 6️⃣ 其他微服务使用

- 引入依赖
```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>MinIO-spring-boot--starter</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```


- 配置
```yaml
minio:
  accessKey: minio
  secretKey: 24364726
  bucket: example-name
  endpoint: http://192.168.200.130:9000
  readPath: http://192.168.200.130:9000
```


- 使用
```java
@SpringBootTest(classes = MinioApplication.class)
@RunWith(SpringRunner.class)
public class MinioTest {

  @Autowired
  private FileStorageService fileStorageService;

  @Test
  public void testUpdateImgFile() {
    try {
      FileInputStream fileInputStream = new FileInputStream("E:\\temp\\test.jpg");
      String filePath = fileStorageService.uploadImgFile("", "test.jpg", fileInputStream);
      System.out.println(filePath);
    } catch (FileNotFoundException e) {
      e.printStackTrace();
    }
  }
}
```
