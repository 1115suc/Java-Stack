# 📦 Maven 下载 jar 包问题及解决方案

## ❗ Maven 下载问题概述

### 🚨 下载失败原因
Maven 在下载 jar 包时，可能会因为网络传输过程中数据丢失或中途断网而导致下载失败。

![7d5295d816ad4bfab21a2f5de6c61b5b.png](img/7d5295d816ad4bfab21a2f5de6c61b5b.png)

### ✅ Maven 正常下载流程
- 下载过程中：jar 包扩展名为 `xxx.jar.lastUpdated`
- 下载成功后：Maven 自动删除 `lastUpdated` 扩展名，恢复为 `xxx.jar`

---

## 🔧 下载失败情况一：`.lastUpdated` 文件残留

### 📋 问题描述
当网络连接中断时，未完成的 jar 包会保留 `xxx.jar.lastUpdated` 扩展名。Maven 重新下载时发现此扩展名就不再处理。

### 🛠️ 解决方案

#### 手动清理方式
手动删除所有以 `lastUpdated` 结尾的文件，然后让 Maven 重新下载。

#### 批量清理脚本
当 `lastUpdated` 文件过多时，使用 `clearLastUpdated.bat` 批处理文件：

**使用步骤：**
1. 将 `clearLastUpdated.bat` 复制到本地仓库根目录
2. 编辑文件，修改配置：
    - `SET CLEAR_PATH=` 设置本地仓库所在盘符
    - `SET CLEAR_DIR=` 设置本地仓库根目录路径
3. 双击运行脚本即可清理

**示例配置：**
```batch
SET CLEAR_PATH=D:
SET CLEAR_DIR=D:\maven-rep1026
```

---

## 🔍 下载失败情况二：jar 包损坏

### 🎯 问题特征
- jar 包表面完整（无 `.lastUpdated` 扩展名）
- 运行时报 `ClassNotFoundException` 等类找不到错误

### 🔐 SHA1 校验方法

#### 校验步骤
1. Win + R 打开命令行
2. 使用命令查看 jar 包 SHA1 值：
   ```cmd
   certutil -hashfile 文件名 SHA1
   ```

3. 对比 jar 包旁 `*.sha1` 文件中的 SHA1 值
4. **一致**：jar 包完好
5. **不一致**：jar 包损坏

### 📚 HASH 算法原理
- 输入数据不变 → 输出值不变
- 输入数据微变 → 输出值大变
- 不可逆性：无法从密文反推明文
- 固定长度：每种 HASH 算法输出长度固定

### 🔎 定位问题 jar 包

当出现 `ClassNotFoundException` 时，如：`org.springframework.expression.Expression`

**定位方法：**
1. 利用 package 名与 `groupId` 的相似性推测
2. 在 IDEA 中双击 Shift，使用全类名搜索
3. 找到 jar 包后右键 → `Show in Explorer`

### ⚠️ 为什么不建议删除整个库

1. **重新下载成本高**：需要重新下载整个库的所有 jar 包
2. **并行下载风险**：多个 jar 包同时下载更容易出现数据丢失

---

## 💡 重新触发 Maven 下载

清理完成后，可通过以下方式重新下载 jar 包：
```bash
mvn clean compile
mvn -U clean compile  # 强制更新依赖
```
