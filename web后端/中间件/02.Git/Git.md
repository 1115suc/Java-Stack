# 📌 Git 学习笔记

## 🔗 Git 官方下载地址

👉 [https://git-scm.com/download](https://git-scm.com/download)

---

## 🛠️ 基础操作命令

### 初始化 & 状态查看
- `git init`：初始化本地仓库 ✅
- `git status`：查看工作区文件状态

### 文件添加到暂存区
- `git add <file>`：添加指定文件
- `git add .`：添加当前目录所有文件（不含删除）
- `git add -A`：添加所有变更（新增/修改/删除）
- `git add -u`：添加已跟踪文件的修改和删除

### 提交到本地仓库
- `git commit -m "message"`：提交暂存区内容
- 首次提交前需配置用户信息：
  ```bash
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
  ```

### 比较差异
- `git diff <file>`：工作区 vs 暂存区
- `git diff --cached <file>`：暂存区 vs 本地仓库
- `git diff HEAD <file>`：工作区 vs 本地仓库

---

## ⏪ 版本控制与回退

### 版本回退
- `git reset --hard HEAD^`：回退至上一版本
- `git reset --hard <commit_id>`：回退到指定版本
- 使用 `HEAD~n` 可快速表示前 n 个版本

### 查看提交记录
- `git log`：查看详细提交历史
- `git log --decorate`：显示分支指向
- `git reflog`：查看所有操作记录（含版本号）

### 撤销操作
- `git checkout -- <file>`：撤销工作区修改
- `git reset HEAD <file>`：将文件从暂存区撤回

---

## 🌿 分支管理

- `git branch <branch_name>`：创建分支
- `git branch`：查看所有分支
- `git checkout <branch_name>`：切换分支
- `git merge <branch_name>`：合并某分支到当前分支
- `git branch -d <branch_name>`：删除分支（需先合并）

📌 **注意**：合并前必须在目标分支上完成 commit。

---

## ☁️ 远程仓库交互

### 关联远程仓库
```bash
git remote add origin <remote_repo_url>
```

### 推送与拉取
- 首次推送：
  ```bash
  git push -u origin main
  ```

- 后续推送：
  ```bash
  git push
  ```

- 拉取远程更新：
    - 首次拉取（允许无关历史）：
      ```bash
      git pull origin master --allow-unrelated-histories
      ```

    - 日常拉取：
      ```bash
      git pull
      ```

### 克隆远程仓库
```bash
git clone <remote_repo_url>
```

🔍 **区别说明**
| 命令       | 场景                         |
|------------|------------------------------|
| `git clone` | 本地无仓库，完整复制远程仓库 |
| `git pull`  | 本地已有仓库，获取并合并更新 |

---

## 💼 常用技巧

### 忽略文件 `.gitignore`
常见规则：
```
*.log
/node_modules
/build
.env
```

### 查看谁修改了某一行代码
```bash
git blame <file>
```

### 查找历史中的特定内容
```bash
git log -S "keyword"  # 查找添加或删除 keyword 的提交
```

### 打标签（发布版本）
```bash
git tag v1.0
git push origin --tags
```

### 暂存当前更改（临时保存）
```bash
git stash          # 暂存当前未提交的更改
git stash pop      # 恢复最近一次暂存
```

### 修改最后一次提交
```bash
git commit --amend
```
