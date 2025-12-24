# Docker 容器部署 🐳


- Docker 部署 MySQL
```shell
docker run -d -p 3306:3306 --privileged=true \
      -v /SuChan/docker/volume/mysql/log:/var/log/mysql \
      -v /SuChan/docker/volume/mysql/data:/var/lib/mysql \
      -v /SuChan/docker/volume/mysql/conf:/etc/mysql/conf.d \
      -v /SuChan/docker/volume/mysql/mysql-files:/var/lib/mysql-files/ \
      -e TZ=Asia/Shanghai \
      -e MYSQL_CHARACTER_SET_SERVER=utf8mb4 \
      -e MYSQL_COLLATION_SERVER=utf8mb4_unicode_ci \
      -e MYSQL_DEFAULT_TIME_ZONE='+8:00' \
      -e MYSQL_ROOT_PASSWORD=24364726 \
      --name mysql mysql:5.7
```

- Docker 部署 MongoDB
```shell
docker run -d \
      --name MongoDB \
      -v /SuChan/docker/volume/mongodb/data:/data/db \
      -v /SuChan/docker/volume/mongodb/log:/data/log \
      -p 27017:27017 \
      --privileged=true \  
      -e MONGO_INITDB_ROOT_USERNAME=root \
      -e MONGO_INITDB_ROOT_PASSWORD=24364726 \
      -e TZ=Asia/Shanghai \
      mongo
```

> --privileged=true \ 当使用--privileged=true选项运行容器时，Docker会赋予容器几乎与主机相同的权限4。具体来说，这个选项做了以下两件事情：         
> 1.给容器添加了所有的capabilities         
> 2.允许容器访问主机的所有设备


- Docker 部署 RabbitMQ
```shell
docker run \
      -e RABBITMQ_DEFAULT_USER=root \
      -e RABBITMQ_DEFAULT_PASS=24364726 \
      -v mq-plugins:/plugins \
      --name RabbitMQ \
      --hostname mq \
      -p 15672:15672 \
      -p 5672:5672 \
      -d rabbitmq:3.8-management
```

- Docker 部署 Elasticsearch
```shell
docker run -d \
      --name elasticsearch \
      -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
      -e "discovery.type=single-node" \
      -v es-data:/usr/share/elasticsearch/data \
      -v es-plugins:/usr/share/elasticsearch/plugins \
      --privileged \
      --network es-net \
      -p 9200:9200 \
      -p 9300:9300 \
      elasticsearch:7.17.5
```

- Docker 部署 Kibana
```shell
docker run -d \
      --name kibana \
      -e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 \
      --network=es-net \
      -p 5601:5601 \
      kibana:7.17.5
```

- Docker 部署 xxl-job-admin (需要创建数据库 xxl_job)
```shell
docker run -d \
      -e PARAMS="--spring.datasource.url=jdbc:mysql://192.168.88.128:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=UTC --spring.datasource.username=root --spring.datasource.password=24364726" 
      -p 8093:8080  
      -v xxldata:/data/applogs 
      --name=xxl-job-admin 
      xuxueli/xxl-job-admin:2.3.0
```


- Docker 部署 Sentinel
```shell
docker run -d \
      --name sentinel \
      -p 8858:8858 \ 
      bladex/sentinel-dashboard:1.8.0
```


- Docker 部署 MinIO
```shell
docker run -p 9000:9000 --name minio -d \
       --restart=always \
       -e "MINIO_ACCESS_KEY=minio" \
       -e "MINIO_SECRET_KEY=24364726" \
       -v /home/data:/data \
       -v /home/config:/root/.minio minio/minio \
       server /data
```

