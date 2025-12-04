# 🐬 MySQL基础教程

## 1️⃣ 数据库基础概念 📚

### 🏗️ 什么是数据库
数据库就是存储数据的仓库，其本质是一个文件系统，数据按照特定的格式将数据存储起来，用户可以通过SQL对数据库中的数据进行增加，修改，删除及查询操作。

### 💾 数据存储方式演变
1. **原始阶段**：数据保存在内存中
2. **文件阶段**：数据保存在文件中
3. **数据库阶段**：数据保存在数据库中

### ✅ 数据库的优点
- 持久化存储
- 读写速度极高
- 保证数据的有效性
- 对程序支持性非常好，容易扩展

### 🌟 常见数据库
- **MySQL**: 开源免费，中小型数据库，已经被Oracle收购
- **Oracle**: 收费的大型数据库
- **DB2**: IBM公司的数据库，收费
- **SQLServer**: 微软的中型数据库
- **SQLite**: 嵌入式的小型数据库

## 2️⃣ MySQL安装与配置 🔧

### 📥 安装步骤
1. 解压安装包
2. 复制到指定目录
3. 配置环境变量
4. 初始化数据库

### 🔌 连接方式
#### 命令行连接
```bash
mysql -uroot -proot
```

#### 图形界面工具
- **DataGrip**: JetBrains出品的数据库管理工具
- **Navicat**: 流行的数据库管理工具

### 🗑️ 卸载方法
1. 停止MySQL服务
2. 卸载MySQL程序
3. 删除安装目录
4. 删除注册表信息

## 3️⃣ SQL语句分类 📝

### 🎯 SQL分类详解
| 分类 | 全称 | 说明 |
|------|------|------|
| DDL | Data Definition Language | 数据定义语言，定义数据库对象（数据库、表、字段） |
| DML | Data Manipulation Language | 数据操作语言，对数据库表中的数据进行增删改 |
| DQL | Data Query Language | 数据查询语言，查询数据库中表的记录 |
| DCL | Data Control Language | 数据控制语言，创建数据库用户、控制访问权限 |

### 🎨 SQL语法规则
- SQL语句可以单行或多行书写，以分号结尾
- 可使用空格和缩进来增强语句的可读性
- 不区分大小写，建议关键字大写

## 4️⃣ DDL数据库操作 🏗️

### 🗄️ 数据库操作
```sql
-- 创建数据库
CREATE DATABASE 数据库名;

-- 查看所有数据库
SHOW DATABASES;

-- 使用数据库
USE 数据库名;

-- 删除数据库
DROP DATABASE 数据库名;
```

### 🔍 数据库连接问题解决
如果遇到连接失败，检查：
1. MySQL服务是否启动
2. 用户名密码是否正确
3. 端口是否被占用

## 5️⃣ DDL表结构操作 📋

### 🏗️ 创建表
```sql
CREATE TABLE 表名 (
    字段名1 数据类型1,
    字段名2 数据类型2,
    ...
);
```

### 📊 数据类型选择
| 类型 | 描述 | 示例 |
|------|------|------|
| INT | 整数类型 | age INT |
| DOUBLE | 小数类型 | score DOUBLE(5,2) |
| VARCHAR | 可变长字符串 | name VARCHAR(20) |
| CHAR | 固定长度字符串 | sex CHAR(1) |
| DATE | 日期类型 | birthday DATE |

### 🔍 表结构查询
```sql
-- 查看所有表
SHOW TABLES;

-- 查看表结构
DESC 表名;

-- 查看建表语句
SHOW CREATE TABLE 表名;
```

### ✏️ 表结构修改
```sql
-- 添加列
ALTER TABLE 表名 ADD 字段名 类型;

-- 修改列类型
ALTER TABLE 表名 MODIFY 字段名 新的类型;

-- 修改列名和类型
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型;

-- 删除列
ALTER TABLE 表名 DROP 字段名;

-- 修改表名
RENAME TABLE 旧表名 TO 新表名;
```

### 🗑️ 删除表
```sql
-- 直接删除
DROP TABLE 表名;

-- 判断存在后删除
DROP TABLE IF EXISTS 表名;
```

## 6️⃣ DML数据操作 💾

### ➕ 插入数据
```sql
-- 插入全部字段
INSERT INTO 表名 (字段1, 字段2...) VALUES (值1, 值2...);
INSERT INTO 表名 VALUES (值1, 值2...);

-- 插入部分字段
INSERT INTO 表名 (字段1, 字段2...) VALUES (值1, 值2...);

-- 批量插入
INSERT INTO 表名 VALUES 
(值1, 值2...),
(值1, 值2...),
(值1, 值2...);
```

### ✏️ 更新数据
```sql
-- 不带条件修改
UPDATE 表名 SET 字段名=新值;

-- 带条件修改
UPDATE 表名 SET 字段名=新值 WHERE 条件;

-- 修改多个字段
UPDATE 表名 SET 字段1=新值1, 字段2=新值2 WHERE 条件;
```

### 🗑️ 删除数据
```sql
-- 带条件删除
DELETE FROM 表名 WHERE 条件;

-- 删除所有数据
DELETE FROM 表名;

-- 清空表（DDL）
TRUNCATE TABLE 表名;
```

## 7️⃣ DQL数据查询 🔍

### 👁️ 基础查询
```sql
-- 查询所有列
SELECT * FROM 表名;

-- 查询指定列
SELECT 字段1, 字段2 FROM 表名;

-- 别名查询
SELECT 字段1 AS 别名1, 字段2 AS 别名2 FROM 表名;

-- 去重查询
SELECT DISTINCT 字段 FROM 表名;
```

### 🔎 条件查询
```sql
-- 比较运算符
SELECT * FROM 表名 WHERE 字段 > 值;
SELECT * FROM 表名 WHERE 字段 = 值;
SELECT * FROM 表名 WHERE 字段 != 值;

-- 逻辑运算符
SELECT * FROM 表名 WHERE 条件1 AND 条件2;
SELECT * FROM 表名 WHERE 条件1 OR 条件2;

-- IN关键字
SELECT * FROM 表名 WHERE 字段 IN (值1, 值2, 值3);

-- BETWEEN范围
SELECT * FROM 表名 WHERE 字段 BETWEEN 值1 AND 值2;
```

### 🔍 模糊查询
```sql
-- %表示任意多个字符
SELECT * FROM 表名 WHERE 字段 LIKE '张%';  -- 查询姓张的
SELECT * FROM 表名 WHERE 字段 LIKE '%张%'; -- 包含张的

-- _表示一个字符
SELECT * FROM 表名 WHERE 字段 LIKE '张_';  -- 姓张且两个字
SELECT * FROM 表名 WHERE 字段 LIKE '_张_'; -- 中间是张
```

### 📊 排序查询
```sql
-- 单列排序
SELECT * FROM 表名 ORDER BY 字段 ASC;   -- 升序（默认）
SELECT * FROM 表名 ORDER BY 字段 DESC;  -- 降序

-- 组合排序
SELECT * FROM 表名 ORDER BY 字段1 DESC, 字段2 ASC;
```

### 🧮 聚合函数
```sql
-- 统计数量
SELECT COUNT(*) FROM 表名;

-- 求和
SELECT SUM(字段) FROM 表名;

-- 最大值
SELECT MAX(字段) FROM 表名;

-- 最小值
SELECT MIN(字段) FROM 表名;

-- 平均值
SELECT AVG(字段) FROM 表名;
```

### 👥 分组查询
```sql
-- 基本分组
SELECT 分组字段, 聚合函数 FROM 表名 GROUP BY 分组字段;

-- 分组后筛选
SELECT 分组字段, 聚合函数 FROM 表名 
GROUP BY 分组字段 
HAVING 聚合函数 > 值;
```

## 8️⃣ 高级查询技巧 🚀

### 🔄 蠕虫复制
```sql
-- 复制表数据
INSERT INTO 表名1 SELECT * FROM 表名2;

-- 复制部分字段
INSERT INTO 表名1(字段1, 字段2) SELECT 字段1, 字段2 FROM 表名2;
```

### 🎯 查询技巧总结
1. **select语句书写顺序**：
   ```
   SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY
   ```

2. **select语句执行顺序**：
   ```
   FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
   ```

3. **分组使用场景**：当需求中出现"每种"、"每个"等字眼时使用分组

## 9️⃣ 实战练习 🎯

### 📝 基础练习题
```sql
-- 1. 创建数据库
CREATE DATABASE demo1;

-- 2. 创建学生表
CREATE TABLE student (
    id INT,
    name VARCHAR(50),
    gender VARCHAR(2),
    address VARCHAR(50)
);

-- 3. 添加年龄字段
ALTER TABLE student ADD age INT;

-- 4. 插入测试数据
INSERT INTO student VALUES 
(1, '袁隆平', '男', '江西', 90),
(2, '钟南山', '男', '福建', 84),
(3, '邓稼先', '男', '安徽', 62);

-- 5. 更新数据
UPDATE student SET age = 100 WHERE id = 1;

-- 6. 删除数据
DELETE FROM student WHERE id = 3;
```

## 🔚 总结与展望

### 📌 学习要点回顾
1. 掌握DDL语句：数据库和表的创建、修改、删除
2. 掌握DML语句：数据的增删改操作
3. 掌握DQL语句：各种查询方式
4. 理解SQL语句的执行顺序
5. 熟练使用聚合函数和分组查询

### 🚀 下一步学习建议
- 深入学习多表查询（连接查询）
- 掌握事务处理
- 学习索引优化
- 了解数据库设计范式
- 实践项目应用

---
