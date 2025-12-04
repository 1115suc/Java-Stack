# 🚀 MySQL高级索引全面指南

## 📚 学习资源推荐
~~~html
索引高级：
https://www.bilibili.com/video/BV1MS4y1T7uJ?from=search&seid=5032261320934971179&spm_id_from=333.337.0.0
hashmap：
https://www.bilibili.com/video/BV1nJ411J7AA?from=search&seid=5641536392359642884&spm_id_from=333.337.0.0
~~~

---

## 📖 第一部分：MySQL常用函数

### 1.1 日期函数 📅

| 函数名 | 描述 | 实例 | 重要性 |
|--------|------|------|--------|
| **NOW()** 和 SYSDATE() | 返回系统的当前日期和时间 | SELECT NOW(); 或 SELECT SYSDATE(); | ⭐️⭐️⭐️⭐️⭐️ |
| **CURDATE()** | 返回当前日期 | SELECT CURDATE(); | ⭐️⭐️⭐️⭐️⭐️ |
| CURTIME() | 返回当前系统时间 | SELECT CURTIME(); | ⭐️⭐️⭐️ |
| **YEAR(d)** | 返回d的中的年份 | SELECT YEAR(NOW()); | ⭐️⭐️⭐️⭐️ |
| MONTH(d) | 返回d的中的月份 | SELECT MONTH(NOW()); | ⭐️⭐️⭐️ |
| DAY(d) | 返回d中的日 | SELECT DAY(NOW()); | ⭐️⭐️⭐️ |
| WEEK(d) | 返回d为一年中的第几周 | SELECT WEEK(NOW()); | ⭐️⭐️ |

<div style='background-color:#42C0A3;'>示例: </div>

```sql
SELECT NOW(); 或 SELECT SYSDATE(); -- 返回系统的当前时间: 年-月-日 时:分:秒 
SELECT CURDATE(); -- 返回系统当前日期: 年-月-日
SELECT CURTIME(); -- 返回系统当前时间: 时:分:秒
SELECT YEAR(NOW()); -- 返回当前日期中的年份
SELECT MONTH(NOW()); -- 返回当前日期中的月份
SELECT DAY(NOW()); -- 返回当前日期中的日
SELECT WEEK(NOW()); -- 返回当前日期中属于一年的第几周
```

### 1.2 Case When判断函数 🤔

#### 基本概念
- **作用**：计算条件列表并返回多个可能结果表达式之一
- **类比**：类似于Java中的switch多分支语句

#### 两种格式

**📝 格式一：简单Case函数**
```sql
CASE 表达式1 
         WHEN 表达式2 THEN 表达式3 
         WHEN 表达式4 THEN 表达式5
         ......
ELSE 表达式6 
END 

-- 示例：
CASE sex 
         WHEN '1' THEN '男' 
         WHEN '2' THEN '女' 
ELSE '其他' 
END 
```

**📝 格式二：Case搜索函数**
```sql
CASE 
         WHEN 条件表达式1 THEN 表达式2 
         WHEN 条件表达式3 THEN 表达式4
         ......
ELSE 表达式5 
END 

-- 示例：
CASE 
	WHEN sex = '1' THEN '男' 
	WHEN sex = '2' THEN '女' 
ELSE '其他'
END
```

#### 与SELECT语句结合使用
```sql
-- 简单Case函数 
select 字段1,字段2,
		CASE 字段3
		WHEN 值1 THEN 返回新值
		WHEN 值2 THEN 返回新值
		ELSE '其他'
		END as 别名
from 表名;

-- Case搜索函数
select 字段1,字段2,
		CASE
		WHEN 条件表达式 THEN 返回新值
		WHEN 条件表达式 THEN 返回新值
		ELSE '其他'
		END as 别名
from 表名;
```

#### 实践练习
```sql
-- 准备测试数据
use day03;
create table user(
  id int primary key auto_increment,
  `name` varchar(30),
  age int,
  sex int
);

-- 插入数据 1 男 0 女
insert into user values(null,'张三',18,1),(null,'李四',28,1),(null,'王五',38,0),(null,'赵六',8,0);

-- 需求：将sex字段值转换为中文显示
select id,name,age,
       case sex
			  when 1 THEN '男'
			  when 0 THEN '女'
       else '其他'
       end as 性别
from user;
```

---

## ⚡ 第二部分：MySQL性能优化基础

### 2.1 数据库性能问题分析 🔍

#### 性能优化两种方式
1. **💪 硬优化**：购买更好的服务器硬件（不关注）
2. **🧠 软优化**：数据库操作和设计优化（**重点学习**）

### 2.2 SQL语句类型分析 📊

#### 语句分类
- **🔍 查询密集型**：查询频率较高（8:2比例）→ 适合使用索引优化
- **✏️ 修改密集型**：订单系统等频繁修改场景 → 可考虑ElasticSearch

#### 查看数据库类型
```sql
-- 查询累计插入和返回数据条数
show global status like 'Innodb_rows%';
```

![1588642111101](img\1588642111101.png)

### 2.3 SQL执行效率测试 🧪

#### 准备千万级测试数据
```sql
create database itcast01;
use itcast01;

-- 1. 准备表
CREATE TABLE user(
	id INT,
	username VARCHAR(32),
	password VARCHAR(32),
	sex VARCHAR(6),
	email VARCHAR(50)
);

-- 2. 创建存储过程，实现批量插入记录
DELIMITER $$ -- 声明存储过程的结束符号为$$
CREATE PROCEDURE auto_insert()
BEGIN
    DECLARE i INT DEFAULT 1;
	START TRANSACTION; -- 开启事务
    WHILE(i<=10000000)DO
        INSERT INTO user VALUES(i,CONCAT('jack',i),MD5(i),'male',CONCAT('jack',i,'@itcast.cn'));
        SET i=i+1;
    END WHILE;
	COMMIT; -- 提交
END$$ -- 声明结束
DELIMITER ; -- 重新声明分号为结束符号

-- 3. 查看存储过程
SHOW CREATE PROCEDURE auto_insert;

-- 4. 调用存储过程
CALL auto_insert();
```

**⏰ 预计执行时间**：5分钟左右（因电脑配置而异）

![image-20200702113746689](img\image-20200702113746689.png)

#### 无索引查询测试
```sql
-- 查询id是22的用户（无索引情况）
select * from user where id = 22;
```

![image-20200704091912695](img\image-20200704091912695.png)

**结果**：查询耗时约8秒，性能较差

---

## 🎯 第三部分：MySQL索引核心知识

### 3.1 什么是索引？📚

#### 官方定义
> 索引（index）是帮助MySQL高效获取数据的数据结构（有序）

**通俗理解**：将数据进行排序整理的过程，相当于书籍的目录

![1588643911985](img\1588643911985.png)

#### 索引的重要性
- **有索引**：兰博基尼 🏎️
- **无索引**：人力三轮车 🚲

![image-20200702231058157](img\image-20200702231058157.png)

### 3.2 MySQL索引分类 🗂️

| 索引类型 | 特点 | 应用场景 |
|----------|------|----------|
| **主键（约束）索引** | 主键约束 + 提高查询效率 | 表的主键字段 |
| **唯一（约束）索引** | 唯一约束 + 提高查询效率 | 需要唯一性的字段 |
| **普通索引** | 仅提高查询效率 | 经常查询的字段 |
| **组合（联合）索引** | 多个字段组成索引 | 多字段联合查询 |
| **全文索引** | 全文搜索 | solr、es替代 |
| **hash索引** | key-value高效查询 | 特定场景使用 |

> **注意**：创建表时指定主键和唯一约束，相当于自动添加了对应索引

---

## ⚙️ 第四部分：索引语法实践

### 4.1 创建索引的三种方式

#### 📝 方式一：在已有表上直接创建（了解）
```sql
-- 创建普通索引
create index 索引名 on 表名(字段);

-- 创建唯一索引
create unique index 索引名 on 表名(字段);

-- 创建组合索引
create index 索引名 on 表名(字段1,字段2,..);
```

**示例**：
```sql
create database day04;
use day04;
CREATE TABLE student(id INT, name VARCHAR(32), telephone VARCHAR(11));

-- 创建索引
CREATE INDEX name_idx ON student(name);
CREATE UNIQUE INDEX telephone_uni_idx ON student(telephone);
```

![image-20200703085004366](img\image-20200703085004366.png)

#### 📝 方式二：修改表时指定（了解）
```sql
-- 添加主键索引
alter table 表名 add primary key(字段);

-- 添加唯一索引
alter table 表名 add unique(字段);

-- 添加普通索引
alter table 表名 add index(字段);
```

#### 📝 方式三：创建表时指定（掌握）
```sql
-- 推荐方式：创建表时直接指定索引
CREATE TABLE student3(
 id INT PRIMARY KEY AUTO_INCREMENT, -- 主键索引
 name VARCHAR(32),
 telephone VARCHAR(11) UNIQUE, -- 唯一索引
 sex VARCHAR(5),
 birthday DATE,
 INDEX(name) -- 普通索引
);
```

![image-20200703090908671](img\image-20200703090908671.png)

### 4.2 查看和删除索引 🔍

#### 查看索引
```sql
show index from 表名;
```

![image-20200703091709085](img\image-20200703091709085.png)

#### 删除索引
```sql
-- 方式一：直接删除
drop index 索引名 on 表名;

-- 方式二：修改表时删除
alter table 表名 drop index 索引名;
```

---

## 🚀 第五部分：索引效果实战演示

### 5.1 无索引查询测试

![image-20200703102211151](img\image-20200703102211151.png)

```sql
-- 无索引情况下的查询
select * from user where id = 8888888;           -- 耗时较长
select * from user where username = 'jack1234567'; -- 耗时较长
```

![image-20200703102942834](img\image-20200703102942834.png)

### 5.2 添加索引

#### 空间占用对比
- **无索引前**：数据占用空间较小
- **添加索引后**：空间会增加（索引占用硬盘空间）

![image-20200703103305640](img\image-20200703103305640.png)

```sql
-- 添加索引
ALTER TABLE USER ADD PRIMARY KEY(id);
ALTER TABLE USER ADD INDEX(username);
```

![image-20200703104139995](img\image-20200703104139995.png)

### 5.3 有索引查询测试

```sql
-- 有索引情况下的查询
select * from user where id = 8888888;           -- 速度极快
select * from user where username = 'jack1234567'; -- 速度极快
```

![image-20200703104444965](img\image-20200703104444965.png)

**效果**：查询速度提升几十倍！⚡

---

## ⚖️ 第六部分：索引优缺点分析

### 6.1 优势 ✅

1. **📖 目录索引效果**：提高数据检索效率，降低IO成本
2. **📊 排序优化**：索引底层就是排序，降低数据排序的CPU消耗

### 6.2 劣势 ❌

1. **⏰ 创建维护成本**：建立和维护索引需要时间
2. **💾 存储空间**：索引占用物理存储空间
3. **🔄 维护开销**：数据增删改时需要动态维护索引

---

## 🎯 第七部分：索引创建原则

### 7.1 核心原则

1. **🎯 识别度原则**：字段内容可识别度 ≥ 70%，唯一值个数 ≥ 70%
    - 例：年龄 vs 性别 → 年龄更适合创建索引

2. **🔍 查询频率原则**：经常用于WHERE条件的字段优先创建索引

3. **🔗 连接优化原则**：经常用于表连接的字段创建索引

4. **📈 排序优化原则**：经常用于ORDER BY的字段创建索引

### 7.2 注意事项 ⚠️

> **不要过度创建索引**：索引的建立和维护需要时间成本，会影响数据库整体效率

---

## 🏗️ 第八部分：索引底层数据结构

### 8.1 索引的本质

**索引 = 帮助MySQL高效获取排好序的数据结构**

![1566372154562](img\1566372154562.png)

### 8.2 数据结构演进

#### 1. 🌳 二叉查找树
- **特点**：左子节点 < 父节点 < 右子节点
- **问题**：特殊数据情况下退化为链表，效率降低

![1588649871508](img\1588649871508.png)

#### 2. 🔴 红黑树（平衡二叉树）
- **优化**：通过左旋、右旋、变色保持平衡
- **问题**：树高度较高（1000万数据高度约23），IO次数多

![1588650208409](img\1588650208409.png)

#### 3. 🌲 BTree（多路平衡搜索树）
- **突破**：一个节点存储多个元素，增加宽度降低高度
- **计算**：1000万数据高度约6，大幅提升效率

![1588650730396](img\1588650730396.png)

#### 4. 🌟 B+Tree（MySQL实际使用）
- **优化BTree**：非叶子节点只存索引+指针，叶子节点存数据
- **特点**：更适合范围查询，IO次数更少

![1588651284679](img\1588651284679.png)

---

## 🎉 综合总结

### 📊 核心知识点回顾
1. **索引本质**：有序数据结构，提高查询效率
2. **索引分类**：主键、唯一、普通、组合等6种类型
3. **创建方式**：三种创建方式，推荐创建表时指定
4. **性能对比**：有索引 vs 无索引，性能差异巨大
5. **数据结构**：B+Tree是MySQL索引的底层实现

### 💪 最佳实践建议
- ✅ 根据查询模式合理创建索引
- ✅ 遵循索引创建原则，避免过度索引
- ✅ 理解底层原理，优化数据库设计
- ✅ 定期监控索引性能，适时调整

### 🔮 扩展学习方向
- 索引覆盖优化
- 索引失效场景分析
- 复合索引最左前缀原则
- 索引在分布式系统中的应用