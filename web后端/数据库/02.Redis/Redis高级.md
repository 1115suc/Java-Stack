# Redis高级 📚

## 🎯 1. 事务机制

### 📋 1.1 场景分析

以关注为例，在B站上黑马程序员关注了传智教育，同时传智教育也关注了黑马程序员，那么两者之间就形成了互相关注，互为粉丝的关系。

如果要通过Redis描述这种关系的话，最好的数据类型就是set， 每个用户定义两个set集合，分别用于描述我的关注和我的粉丝：

```yacas
#我的关注 user:用户id:follow 被关注的用户id
#我的粉丝 user:用户id:fans 粉丝用户id

#黑马程序员用户为1001 ， 传智教育用户id为1002
sadd user:1001:follow 1002
sadd user:1001:fans 1002

sadd user:1002:follow 1001
sadd user:1002:fans 1001
```

但是假设在执行四条指令的过程中，某一条指令出现错误运行失败，就会有可能出现A关注了B，而B的粉丝里没有A，以及相类似的情况。

### 🔧 1.2 事务介绍

Redis是支持事务的， 对于上述问题可以通过事务控制解决。

举一个事务的经典例子：转账

* A给B汇款，那么A账户会扣钱
* B账户会加钱

这两个步骤一定会存在于一个事务中，要么都成功，要么都失败。

Redis事务是基于队列实现的，创建一个事务队列，然后将事务操作都放入队列中，最后依次执行。

![image-20210302085407044](assets/image-20210302085407044.png)

#### 💻 1.2.1 安装redis

```shell
tar -zxvf redis-6.2.4.tar.gz

#安装gcc
yum install -y gcc-c++ autoconf automake
#centos7 默认的 gcc 默认是4.8.5,版本小于 5.3 无法编译,需要先安装gcc新版才能编译
gcc -v
#升级新版gcc，配置永久生效
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile 
#将资料中Redis的源码包上传  解压  并且  编译redis
cd redis-6.2.4
make
#安装到指定目录
mkdir -p /usr/local/redis
make PREFIX=/usr/local/redis install
sysctl vm.overcommit_memory=1
```

拷贝演示配置:

```shell
# 将演示用的配置文件 拷贝到指定文件夹
cp ~/redis-6.2.4/redis.conf /usr/local/redis
```

启动redis:

```shell
cd /usr/local/redis
# 运行master
bin/redis-server  /usr/local/redis/redis.conf
```

#### 🎮 1.2.2 演示事务效果

```yacas
#开启事务
multi

#添加命令
sadd user:1001:follow 1002
sadd user:1002:follow 1001
sadd user:1001:fans 1002
sadd user:1002:fans 1002

#执行事务
exec
# 取消事务
discard
```

![image-20210302085938868](assets/image-20210302085938868.png)

### ⚙️ 1.3 事务处理机制

假设这四条指令在事务执行中，某一条命令执行出错， 此时事务会如何处理呢？ 现在可能很多人会说，刚才不是已经说了嘛，要么都成功，要么都失败， 有一个出错，那就都失败呗。

这么想不能算错，但是Redis对于命令执行错误处理，有两种解决方式：

* 语法错误（编译）
* 执行错误（运行）

#### ❌ 1.3.1 语法错误

​	语法错误：**执行命令的语法不正确**。

```yacas
#开启事务
multi

#命令
set name zhangsan
set age
seterror sex male

#执行事务
exec

#获取正确指令数据
get name
```

![image-20210302091533022](assets/image-20210302091533022.png)

​	此时整个事务队列中，存在**一条正确指令，两条语法错误指令**， 当执行exec后，会直接返回错误，正确的命令也不会执行。

#### ⚠️ 1.3.2 执行错误

​	执行错误：**命令在运行过程中出现错误**。

```yacas
#开启事务
multi

#命令
set lesson java
rpush lesson eureka feign nacos
set lesson redis

#执行事务
exec

#获取数据
get lesson
```

![image-20210302094737093](assets/image-20210302094737093.png)

​	通过上面事务执行可以看到，语法本身是没有问题的，所以运行之前redis无法发现错误，但是在执行时出现了错误，因此只会错误的命令不执行， 而正确的命令仍然能够正常执行。

### 🚀 1.4 SpringBoot实现事务操作

1）修改RedisConfig配置类，开启事务控制

```java
//开启redis事务控制
redisTemplate.setEnableTransactionSupport(true);
```

2）自定义方法，测试事务效果

```java
@Test
public void multiTest(){
    //开启事务
    redisTemplate.multi();
    try{
        redisTemplate.opsForValue().set("lesson","java");
        redisTemplate.opsForSet().add("lesson","eureka","feign","gateway");
        redisTemplate.opsForValue().set("lesson","redis");
        System.out.println(redisTemplate.opsForValue().get("lesson"));
    }catch (Exception e){
        //回滚
        System.out.println("出现异常");
        redisTemplate.discard();
    }finally {
        redisTemplate.exec();
    }
}
```

## 💾 2. 持久化机制

### 📊 2.1 场景分析

Redis将数据保存在内存中。一旦服务器宕机重启，内存中的数据就会丢失。当出现这种情况后，为了能够让Redis进行数据恢复，因此Redis提供了持久化机制，将内存中的数据保存到磁盘中，避免数据意外丢失。

Redis提供了两种持久化机制：**RDB**、**AOF**。 根据不同的场景，可以选择只使用其中一种或一起使用。

### 📸 2.2 RDB快照

#### 📖 2.2.1 概述

RDB（Redis DataBase）是Redis默认存储方式。其基于快照思想，当符合一定条件（手动或自动触发）时，Redis会将这一刻的内存数据进行快照并保存在磁盘上，产生一个经过压缩的二进制文件，文件后缀名.rdb。

![image-20210301101319837](assets/image-20210301101319837.png)

![image-20210301101413832](assets/image-20210301101413832.png)

因为RDB文件是保存在磁盘上的，因此即使Redis进程退出，甚至服务器宕机重启。只要RDB文件存在，就可以利用它来还原Redis数据。

#### 🔫 2.2.2 RDB触发条件

##### ⏰ 2.2.2.1 符合配置文件中的快照规则

​	在redis.conf文件中配置了一些默认触发机制。

```yacas
save ""  # 不使用RDB存储  不能主从

# 记忆
save 3600 1     #表示1小时内至少1个键被更改则进行快照。
save 300 100    #表示5分钟（300秒）内至少100个键被更改则进行快照。
save 60 10000  #表示1分钟内至少10000个键被更改则进行快照。
```

![image-20210301105448003](assets/image-20210301105448003.png)

<font color="red">思考：是否会存在数据丢失问题？</font>

##### 🎯 2.2.2.2 手动执行save或bgsave命令

​	在redis客户端执行save或bgsave命令，手动触发RDB快照。

```yacas
#进入客户端
bin/redis-cli

#执行save命令(同步执行)
save

#执行bgsave命令(异步子线程执行)
bgsave
```

![image-20210301105802813](assets/image-20210301105802813.png)

那么这两个命令都会触发快照的话，他们两个又有什么区别呢？

* **save：**同步处理，阻塞Redis服务进程，服务器不会处理任何命令，直到RDB文件保存完毕。
* **bgsave：**会fork一个和主线程一致的子线程负责操作RDB文件，不会阻塞Redis服务进程，操作RDB文件的同时仍然可以处理命令。

<font color="red">Redis默认使用的是 bgsave 来保存快照数据。</font>



#### 🔄 2.2.3 执行过程

![image-20210303160927696](assets/image-20210303160927696.png)

1）Redis服务进程判断，当前是否有子线程在执行save或bgsave。

2）如果有，则直接返回，不做任何处理。

3）如果没有，则以阻塞式创建子线程，在创建子线程期间，Redis不处理任何命令。

4）创建完子线程后，取消阻塞，Redis服务继续响应其他命令。

5）同时基于子线程操作RDB文件，将此刻数据保存到磁盘。

#### 📈 2.2.4 优缺点

**优点：**

- 基于二进制文件完成数据备份，占用空间少，便于文件传输。
- 能够自定义规则，根据Redis繁忙状态进行数据备份。

**缺点：**

- 无法保证数据完整性，会丢失最后一次快照后的所有数据。
- bgsave执行每次执行都会阻塞Redis服务进程创建子线程，频繁执行影响系统吞吐率。

### 📝 2.3 AOF

#### 📖 2.3.1 概述

​	RDB方式会出现数据丢失的问题，对于这个问题，可以通过Redis中另外一种持久化方式解决：**AOF**。

​	AOF（append only file）是Redis提供了另外一种持久化机制。与RDB记录数据不同，当开启AOF持久化后，Redis会将客户端发送的所有更改数据的命令，记录到磁盘中的AOF文件。 这样的话，当Redis重启后，通过读取AOF文件，按顺序获取到记录的数据修改命令，即可完成数据恢复。

![image-20210301115444793](assets/image-20210301115444793.png)

​	举个例子，对Redis执行三条写命令：

```yacas
set name itheima

hset cart shop nike

sadd lesson java python hadoop
```

​	RDB会将name、cart、lesson三个键值对数据进行保存，而AOF会将set、hset、sadd三个命令保存到AOF文件中。

#### 🔧 2.3.2 基础使用

​	AOF方式需要手动开启，修改**redis.conf**

```yacas
# 是否开启AOF，默认为no
appendonly yes

#设置AOF文件名称
appendfilename  appendonly.aof
```

![image-20210301142045738](assets/image-20210301142045738.png)

当开启了AOF机制之后，Redis何时会向aof文件中记录内容呢？

对于AOF的触发方式有三种：**always**、**everysec**、**no**。 默认使用everysec。可以通过redis.conf中appendfsync属性进行配置。

那么这三种参数各自都代表什么意思呢？ 为什么默认使用everysec呢？这里我们做一个预留，因为涉及到一些其他知识，后面再给大家详细介绍。

开启AOF后，重启Redis，进入Redis客户端并执行多条写命令，这些命令会被保存到appendonly.aof文件中。

```yacas
set name zhangsan
set age 18
set sex male
get name
get age
get sex
```

此时查看**redis/data**目录，会新产生一个appendonly.aof文件。 查看文件内容

```yacas
*2
$6
SELECT
$1
0
*3
$3
set
$4
name
$8
zhangsan
*3
$3
set
$3
age
$2
18
*3
$3
set
$3
sex
$4
male
```

​	通过文件查看，可以看到，在aof文件中，记录了对redis操作的所有写命令，读命令并不会记录。

#### ⚙️ 2.3.3 执行原理

​	AOF功能实现的整个执行过程可以分为三个部分：**命令追加**、**文件写入**、**文件同步**。

![image-20210303212635762](assets/image-20210303212635762.png)

1）客户端向Redis发送写命令。

2）Redis将接收到的写命令保存到缓冲文件**aof_buf的末尾**。 **这个过程是命令追加。**

3）redis将缓冲区文件内容写入到AOF文件，**这个过程是文件写入**。

4）redis根据策略将AOF文件保存到磁盘，**这个过程是文件同步。**

5）何时将AOF文件同步到磁盘的策略依据就是**redis.conf**文件中**appendfsync**属性值：**always**、**everysec**、**no**

- **always**：每次执行写入命令都会将aof_buf缓冲区文件全部内容写入到AOF文件中，并将AOF文件同步到磁盘。该方式效率最低，安全性最高。
- **everysec**：每次执行写入命令都会将aof_buf缓冲区文件全部内容写入到AOF文件中。 并且每隔一秒会由子线程将AOF文件同步到磁盘。该方式兼备了效率与安全，即使出现宕机重启，也只会丢失不超过两秒的数据。
- **no**：每次执行写入命令都会将aof_buf缓冲区文件全部内容写入到AOF文件中，但并不对AOF文件进行同步磁盘。 同步操作交由操作系统完成（每30秒一次），该方式最快，但最不安全。

| 模式     | aof_buf写入到AOF是否阻塞 | AOF文件写入磁盘是否阻塞 | 宕机重启时丢失的数据量              | 效率 | 安全 |
| -------- | ------------------------ | ----------------------- | ----------------------------------- | ---- | ---- |
| always   | 阻塞                     | 阻塞                    | 最多只丢失一个命令的数据            | 低   | 高   |
| everysec | 阻塞                     | 不阻塞                  | 不超过两秒的数据                    | 中   | 中   |
| no       | 阻塞                     | 阻塞                    | 操作系统最后一次对AOF写入磁盘的数据 | 高   | 低   |

#### 🔄 2.3.4 AOF重写优化

##### 📖 2.3.4.1 概述

AOF会将对Redis操作的所有写命令都记录下来，随着服务器的运行，AOF文件内保存的内容会越来越多。这样就会造成两个比较严重的问题：**占用大量存储空间**、**数据还原花费的时间多**。

​	举个例子：

```yacas
sadd lessons java
sadd lessons python go
sadd lessons hive
sadd lessons hadoop rocketmq
sadd lessons redis
```

当这些命令执行完，AOF文件中记录5条命令。但是实际生产环境下，写命令会出现非常多，文件的体积也会非常庞大。

​	为了解决AOF文件巨大的问题，Redis提供了AOF文件重写功能。 当AOF文件体积超过阈值时，则会触发AOF文件重写，Redis会开启子线程创建一个新的AOF文件替代现有AOF文件。 新的AOF文件不会包含任何浪费空间的冗余命令，只存在恢复当前Redis状态的最小命令集合。

##### ⚙️ 2.3.4.2 触发配置

​	那么AOF文件达到多大时，会对其进行重写呢？   对于重写阈值的配置，可以通过修改redis.conf进行配置。

```yacas
#当前aof文件大小超过上一次aof文件大小的百分之多少时进行重写。如果之前没有重写过，以
启动时aof文件大小为准
auto-aof-rewrite-percentage 100

#限制允许重写最小aof文件大小，也就是文件大小小于64mb的时候，不需要进行优化
auto-aof-rewrite-min-size 64mb
```

​	除了让Redis自动执行重写外，也可以手动让其进行执行：`bgrewriteaof`

![image-20210301171637281](assets/image-20210301171637281.png)

### 📊 2.4 RDB与AOF对比

1. RDB默认开启，AOF需手动开启。
2. RDB性能优于AOF。
3. AOF安全性优于RDB。
4. AOF优先级高于RDB。
5. RDB存储某个时刻的数据快照，AOF存储**写**命令。
6. RDB在配置触发状态会丢失最后一次快照以后更改的所有数据，AOF默认使用everysec，每秒保存一次，最多丢失两秒以内的数据。

### 🏭 2.5 生产环境下持久化实践

​

1. 如当前只追求高性能，不关注数据安全性，则关闭RDB和AOF，如redis宕机重启，直接从数据源恢复数据。
2. 如需较高性能且关注数据安全性，则开启RDB，并定制触发规则。
3. 如更关注数据安全性，则开启AOF。
4. 黑马头条  选择 即开启AOF  也开启RDB

## 🔄 3. 高可用-主从复制

### ❓ 3.1 问题概述

通过持久化机制的学习， 可以发现，不管是RDB还是AOF，都并不能百分百的避免数据丢失。关键是现在只有一台服务器，持久化数据都是保存在这台服务器的磁盘上，假设这台服务器的磁盘损坏，数据仍然会全部丢失。 那这个问题该怎么解决呢？

那我们想一下，现在所有持久化数据只是保存在一台服务器上，能不能让它们同时保存在多台服务器上，这样即使一台服务器出现问题，仍然可以从其他服务器同步数据。

这样就需要当一台服务器中数据更新后，可以自动的将更新的数据同步到其他服务器上， 这就是所谓的复制。

![image-20210301182912507](assets/image-20210301182912507.png)

### 🔧 3.2 复制搭建&使用

​	![image-20210301183648921](assets/image-20210301183648921.png)

关键步骤：

（1）安装依赖环境及安装Redis

```yaml
tar -zxvf  redis压缩包

#安装gcc
yum install -y gcc-c++ autoconf automake

#centos7 默认的 gcc 默认是4.8.5,版本小于 5.3 无法编译,需要先安装gcc新版才能编译
gcc -v

#升级新版gcc，配置永久生效
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile 




#将资料中Redis的源码包上传  解压  并且  编译redis
cd redis-6.2.4
make

#安装到指定目录
mkdir -p /usr/local/redis

make PREFIX=/usr/local/redis install

sysctl vm.overcommit_memory=1
```

（2）创建目录

```sh
#日志 /usr/local/redis/log
#数据 /usr/local/redis/data
#配置文件 /usr/local/redis/conf
mkdir /usr/local/redis/logs -p
mkdir /usr/local/redis/data -p
mkdir /usr/local/redis/conf -p
```



将资料中配置文件复制到 `conf` 文件夹下



（3）启动Redis master节点

```sh
# master 节点， 进入到redis目录下
cd /usr/local/redis

# 运行master
bin/redis-server ./conf/master/redis-6379.conf
```

日志：`tail -f ./logs/redis_6379.log `

![image-20210624161948397](assets/image-20210624161948397.png)

通过 `bin/redis-cli -p 6379 -a 123456` 链接 redis服务

`info replication`

![image-20210624162407161](assets/image-20210624162407161.png)

（4）启动Redis slave1和slave2节点

slave1节点:

```sh
# slave1 节点， 进入到redis目录下
cd /usr/local/redis

# 运行slave1
bin/redis-server ./conf/slave1/redis-6380.conf
# 链接redis slave1
bin/redis-cli -p 6380 -a 123456

# info 查看信息
```

slave2节点:

```sh
# slave2 节点， 进入到redis目录下
cd /usr/local/redis

# 运行slave2
bin/redis-server ./conf/slave2/redis-6381.conf

# 链接redis slave2
bin/redis-cli -p 6381 -a 123456

# info 查看信息
```

（5）启动完成后查看Redis集群状态是否成功

![image-20210624162627648](assets/image-20210624162627648.png)

验证读写分离：<font color="red">只能主写从读</font>

![image-20210624162850479](assets/image-20210624162850479.png)

### 🏭 3.3 生产环境下主从复制实践

#### 📖 3.3.1 读写分离

在生产环境下，读请求会远远多于写请求，大概10:1的比率。 当单机无法应对大量读请求时，可以通过主从复制机制，实现读写分离。主节点只负责写请求，从节点只负责读请求。

![image-20210301193014441](assets/image-20210301193014441.png)

#### ⚙️ 3.3.2 持久化优化

现在如果master和所有的slave都开启持久化的话，性能相对来说比较低。该如何优化提升性能呢？

我们可以在**从节点上开启持久化、在主节点关闭持久化**。 但是这样的话，数据不会丢失吗？

在主从复制的结构下，无非要么主节点宕机，要么从节点宕机。

- 当从节点宕机重启后，主节点会自动的将数据同步到从节点上。所以不会出现数据丢失。
- 当主节点宕机后，可以将从节点提升为主节点(**slaveof no one**)，继续对外提供服务。 并且当原先的主节点重启后，使用slaveof命令将其设置为新主节点的从节点，即可完成数据同步。

```yacas
#中断端口为6379的redis服务进程
#将6380从节点提升为新的主节点
slaveof no one
#在6380节点添加数据
sadd lessons java redis rocketmq
#启动6379节点

#将6381节点作为从节点连接到新的主节点6380
slaveof 192.168.200.130 6380
#6381节点获取断开连接期间数据
smembers lessons
```

现在可能有人在想，那要是主节点和从节点同时宕机了呢？ 数据这不还是会丢吗？  首先服务器宕机的问题本身出现的几率就非常低，并且采用合理的部署方式，如异地部署。 主节点和从节点同时宕机的几率更是微乎其微的。

### 📋 3.4 主从复制总结

![image-20210624164716647](assets/image-20210624164716647.png)

具体步骤：

1、Slave服务启动，主动连接Master，并发送SYNC命令，请求初始化同步；

2、Master收到SYNC后，执行BGSAVE命令生成RDB文件，并缓存该时间段内的写命令；

3、Master完成RDB文件后，将其发送给所有Slave服务器；

4、Slave服务器接收到RDB文件后，删除内存中旧的缓存数据，并装载RDB文件；

5、Master在发送完RDB后，即刻向所有Slave服务器发送缓存中的写命令；

主从复制的作用：

* 读写分离：**主写从读**，提高服务器的读写负载能力
* 负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量

* 故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复

* 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式

* 高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案

## 🚨 4. 哨兵模式

### ❓ 4.1 问题概述

我们学习了主从库集群模式。在这个模式下，如果从库发生故障了，客户端可以继续向主库或其他从库发送请求，进行相关的操作，但是如果主库发生故障了，那就直接会影响到从库的同步，因为从库没有相应的主库可以进行数据复制操作了。

![image-20210621142937626](assets/image-20210621142937626.png)

如上图：

* 如果客户端发送的**读请求**，可以由从库继续提供服务

* 如果客户端发送的**写请求，**需要主库提供服务，但是主库挂了



无论是写服务中断，还是从库无法进行数据同步，都是不能接受的。所以，如果主库挂

了，我们就需要运行一个新主库，比如说把一个从库切换为主库，把它当成主库。

这就涉及到三个问题：

1. 主库真的挂了吗？

2. 该选择哪个从库作为主库？

3. 怎么把新主库的相关信息通知给从库和客户端呢？



这就要提到哨兵机制了。在 Redis 主从集群中，哨兵机制是实现主从库自动切换的关键机

制，它有效地解决了主从复制模式下故障转移的这三个问题。

哨兵(sentinel) 是一个分布式系统，用于对主从结构中的每台服务器进行**监控**，当出现故障时通过投票机制**选择**新的master并将所有slave连接到新的master。

![image-20210621144247782](assets/image-20210621144247782.png)

**注意：**

* 哨兵也是一台redis服务器，只是不提供数据服务

* 通常哨兵配置数量为单数

* 如果配置启用单节点哨兵，如果有哨兵实例在运行时发生了故障，主从库无法正常

  切换啦，所以我们需要搭建 **哨兵集群**


### 🔧 4.2 集群节点哨兵模式

#### 📖 4.2.1 哨兵模式介绍

![image-20210624165106048](assets/image-20210624165106048.png)

- Redis提供了哨兵的命令，是一个独立的进程

- 原理 哨兵通过发送命令给多个节点，等待Redis服务器响应，从而监控运行的多个Redis实例的运行情况

- 当哨兵监测到master宕机，会自动将slave切换成master，通过通知其他的从服务器，修改配置文件切换主机

**Sentinel三大工作任务**

- 监控（Monitoring）
    - Sentinel 会不断地检查你的主服务器和从服务器是否运作正常
- 提醒（Notification）
    - 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知
- 自动故障迁移（Automatic failover）
    - 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器
    - 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器

#### ⚖️ 4.2.2 客观下线和主观下线

- 主观下线（Subjectively Down， 简称 SDOWN）

    - **哨兵进程会使用 PING 命令检测它自己和主、从库的网络连接情况，用来判断实例的状**

      **态**。如果哨兵发现主库或从库对 PING 命令的响应超时了，那么，哨兵就会先把它标记

      为"主观下线"

    - 一个服务器没有在 down-after-milliseconds 选项所指定的时间内， 对向它发送 PING 命令的 Sentinel 返回一个有效回复（valid reply）， 那么 Sentinel 就会将这个服务器标记为主观下线

    - 如果检测的是从库，那么，哨兵简单地把它标记为"主观下线"就行了，因为从库的下线影响一般不太大，集群的对外服务不会间断。

    - 如果检测的是主库，那么，哨兵还不能简单地把它标记为"主观下线"，开启主从

      切换。因为很有可能存在这么一个情况：那就是哨兵**误判**了，其实主库并没有故障。可

      是，一旦启动了主从切换，后续的选主和通知操作都会带来额外的计算和通信开销。

我们要知道啥叫误判。很简单，就是主库实际并没有下线，但是哨兵误以为它下线了。误判一般会发生在集群网络压力较大、网络拥塞，或者是主库本身压力较大的情况下。

- 客观下线（Objectively Down， 简称 ODOWN）

  ![image-20210624170202319](assets/image-20210624170202319.png)

    - 指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断
    - 一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线
    - 客观下线条件只适用于主服务器

- 仲裁 qurum

    - Sentinel 在给定的时间范围内， 从其他 Sentinel 那里接收到了【足够数量】的主服务器下线报告， 那么 Sentinel 就会将主服务器的状态从主观下线改变为客观下线
    - 这个【足够数量】就是配置文件里面的值，一般是Sentinel个数的一半加1，比如3个Sentinel则就设置为2
    - down-after-milliseconds 是一个哨兵在超过规定时间依旧没有得到响应后，会自己认为主机不可用
    - 当拥有认为主观下线的哨兵达到sentinel monitor所配置的数量时，就会发起一次投票，进行failover

#### 🏗️ 4.2.3 一主二从三哨兵搭建

关键步骤：

- 基于搭建成功的一主二从Redis集群

- 配置3个哨兵，每个哨兵的配置都是一样的

- 启动顺序 先启动主再启动从，最后启动3个哨兵

- 哨兵端口是 【26379】


（1）配置文件

```sh
#不限制ip
bind 0.0.0.0

# 让sentinel服务后台运行
daemonize yes

# 配置监听的主服务器，mymaster代表服务器的名称，自定义，192.168.200.130 代表监控的主服务器，6379代表端口，
#2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
# 计算规则：哨兵个数/2 +1
sentinel monitor mymaster 192.168.200.130 6379 2

# sentinel auth-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
sentinel auth-pass mymaster 123456

#超过5秒master还没有连接上，则认为master已经停止
sentinel down-after-milliseconds mymaster 5000

#如果该时间内没完成failover操作，则认为本次failover失败
sentinel failover-timeout mymaster 30000
```

在`redis/conf/sentinel`目录下创建3个文件 sentinel-1.conf、sentinel-2.conf、sentinel-3.conf



（2）启动哨兵集群

```sh
#  进入到redis目录下
cd /usr/local/redis

# sentinel1
bin/redis-server ./conf/sentinel/sentinel1.conf --sentinel

# sentinel2
bin/redis-server ./conf/sentinel/sentinel2.conf --sentinel

# sentinel3
bin/redis-server ./conf/sentinel/sentinel3.conf --sentinel
```

（3）查看日志

```sh
tail -f ./logs/sentinel_26379.log
```

![image-20210624171752727](assets/image-20210624171752727.png)

```sh
tail -f ./logs/sentinel_26380.log
```

![image-20210624172110065](assets/image-20210624172110065.png)

```sh
tail -f ./logs/sentinel_26381.log
```

![image-20210624172253411](assets/image-20210624172253411.png)

查看Sentinel1 日志发现，其它两个Sentinel加入到集群中

![image-20210624172401577](assets/image-20210624172401577.png)

#### 🧪 4.2.4 哨兵模式测试

（1）关闭Redis master服务 `SHUTDOWN`

![image-20210624173830416](assets/image-20210624173830416.png)

（2）观察哨兵模式日志

哨兵日志：

![image-20210624180816418](assets/image-20210624180816418.png)

名称解释：

基于pub/sub的客户端事件通知
- 主库下线事件：
    - +sdown：实例进入主观下线状态
    - -sdown：实例退出主观下线状态
    - +odown：实例进入客观下线状态
    - -odown：实例退出客观下线状态
- 从库重新配置事件
    - +slave-reconf-sent：哨兵发送SLAVEOF命令重新配置从库
    - +slave-reconf-inpprog：从库配置了新主库，但尚未进行同步
    - +slave-reconf-done：从库配置了新主库，且和新主库完成同步
- 新主库切换：

    - +swith-master：主库地址发送变化


redis 6381 日志：

![image-20210624180748138](assets/image-20210624180748138.png)

redis 6380 日志：

![image-20210624180914779](assets/image-20210624180914779.png)

redis 6379 重启查看日志：

![image-20210624181045393](assets/image-20210624181045393.png)

### ⚙️ 4.3 集群节点哨兵模式工作原理

![image-20210624185214180](assets/image-20210624185214180.png)

* 哨兵集群中的多个实例共同判断，可以降低对**主库下线**的**误判率**
    * 哨兵集群组成: 基于 **pub/sub** 机制
        * 哨兵间发现：
            * 主库频道"\__sentinel__:hello"，不同哨兵通过它相互发现，实现互相通信
        * 哨兵发现从库
            * 向主库发送 INFO 命令

    * 基于 pub/sub 机制的客户端事件通知
        * 事件：主库下线事件
            * +sdown : 实例进入"主观下线"状态
            * -sdown : 实例退出"主观下线"状态
            * +odown : 实例进入"客观下线"状态
            * -odown : 实例退出"客观下线"状态
        * 事件：从库重新配置事件
            * +slave-reconf-sent : 哨兵发送 SLAVEOF 命令重新配置从库
            * +slave-reconf-inprog : 从库配置了新主库，但尚未进行同步
            * +slave-reconf-done : 从库配置了新主库，且完成同步
        * 事件：新主库切换
            * +switch-master : 主库地址发生变化

    * 由哪个哨兵执行主从切换？
        * 一个哨兵获得了仲裁**所需的赞成票数**后，就可以标记主库为"客观下线"
            * 所需的赞成票数 &lt;= quorum 配置项

      > 例如，现在有 5 个哨兵，quorum 配置的是 3，那么，一个哨兵需要 3 张赞成票，就可以标记主库为"客观下线"了。这 3 张赞成票包括哨兵自己的一张赞成票和另外两个哨兵的赞成票。

        * "Leader 选举"
            * 两个条件：
                * 拿到半数以上的赞成票
                * 拿到的票数&gt;= quorum
            * 如果未选出，则集群会等待一段时间（哨兵故障转移超时时间的 2 倍），再重新选举

    * 经验：要保证所有哨兵实例的**配置是一致**的

        * 尤其是主观下线的判断值 down-after-milliseconds

### 📋 4.4 总结

目前解决了什么问题：

* 主从集群间可以实现自动切换，可用性更高
* 数据更大限度的防止丢失
* 解决哨兵的集群高可用问题，减少误判率

目前还存在什么问题：

* 海量数据存储问题
* 高并发写的问题

## 🚀 5. 高可扩-Redis Cluster分片集群

### ❓ 5.1 问题概述

提出问题：要用 Redis 保存 5000 万个键值对，每个键值对大约是 512B，为了能快速部署并对外提供服务，我们采用云主机来运行 Redis 实例，那么，该如何选择云主机的内存容量呢？

思路分析：

* 所有的key占用内存预估是：5000 万 *512B = 25GB，所以redis的服务器申请 32G以上可以解决海量数据存储问题
* 为防止单点故障，必须还需要主从+哨兵实现故障自动转移和恢复数据
* 过程会发现一旦主节点宕机，会出现数据恢复时间会很长（秒级以上）

得出结论：如果只是单纯按照内存来申请redis服务器，这个方案是行不通的

那么如果想解决这个问题，就必须需要 Redis Cluster 分片集群，在Redis 3.0 之后开始支持。

### ⚙️ 5.2 分片集群原理

![image-20210624192532853](assets/image-20210624192532853.png)

Redis原理：

* 数据切片和实例的对应分布关系
    * Redis Cluster 方案：无中心化
        * 采用哈希槽（Hash Slot）来处理数据和实例之间的映射关系
        * 一个切片集群共有 16384 个哈希槽，**只给Master分配**

        * 具体的映射过程
            1. 根据键值对的 key，按照CRC16 算法计算一个 16 bit 的值；
            2. 再用这个 16bit 值对 16384 取模，得到 0~16383 范围内的模数，每个模数代表一个相应编号的哈希槽

    * 哈希槽映射到具体的 Redis 实例上
        * 用 cluster create 命令创建集群，Redis 会自动把这些槽平均分布在集群实例上
        * 也可以使用 cluster meet 命令手动建立实例间的连接，形成集群，再使用 cluster addslots 命令，指定每个实例上的哈希槽个数

          注意：需要把 16384 个槽都分配完，否则 Redis 集群无法正常工作

* 客户端如何定位数据
    * Redis 实例会把自己的哈希槽信息发给和它相连接的其它实例，来完成哈希槽分配信息的扩散
    * 客户端和集群实例建立连接后，实例就会把哈希槽的分配信息发给客户端
    * 客户端会把哈希槽信息缓存在本地。当请求键值对时，会先计算键所对应的哈希槽
    * 但集群中，实例和哈希槽的对应关系并不是一成不变的
        * 实例新增或删除
        * 负载均衡
    * 实例之间可以通过相互传递消息，获得最新的哈希槽分配信息，但客户端是无法主动感知这些变化

* **重定向机制**

    * 如果实例上没有该键值对映射的哈希槽，就会返回 MOVED 命令
        * 客户端会更新本地缓存
    * 在**迁移部分完成**情况下，返回ASK
        * 表明 Slot 数据还在迁移中
        * ASK 命令把客户端所请求数据的最新实例地址返回给客户端
        * 并不会更新客户端缓存的哈希槽分配信息
