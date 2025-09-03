# Git 仓库（Repository）完全指南

> Git 仓库是 Git 版本控制系统的核心，它包含了项目的完整历史记录和所有版本信息。理解 Git 仓库的结构和工作原理是掌握 Git 的关键。

## 什么是 Git 仓库？

Git 仓库（Repository，简称 Repo）是 Git 用来存储项目历史记录和元数据的数据库。每个 Git 仓库都完整包含了项目的所有文件和历史版本信息。

### 仓库类型

1. **本地仓库**：存储在开发者本地计算机上
2. **远程仓库**：存储在远程服务器上（如 GitHub、GitLab 等）

## 创建 Git 仓库

### 初始化新仓库

```bash
# 在当前目录初始化新仓库
git init

# 初始化并指定目录名
git init <项目目录>
```

### 克隆现有仓库

```bash
# 克隆远程仓库
git clone <远程仓库URL>

# 克隆并指定本地目录名
git clone <远程仓库URL> <本地目录名>

# 克隆特定分支
git clone -b <分支名> <远程仓库URL>
```

## 仓库结构

一个典型的 Git 仓库包含以下重要部分：

```
.git/               # Git 元数据目录
├── HEAD            # 当前检出的引用
├── config          # 仓库配置
├── description     # 仓库描述
├── hooks/          # 客户端钩子脚本
├── info/           # 全局排除文件
├── objects/        # 所有 Git 对象
├── refs/           # 分支和标签的指针
│   ├── heads/      # 本地分支
│   └── tags/       # 标签
├── index           # 暂存区信息
└── logs/           # 引用更改日志
```

## 仓库操作

### 查看仓库状态

```bash
git status
```

### 添加文件到暂存区

```bash
# 添加单个文件
git add <文件名>

# 添加所有更改
git add .

# 交互式添加
git add -p
```

### 提交更改

```bash
# 提交暂存区更改
git commit -m "提交消息"

# 添加并提交所有更改（跳过暂存区）
git commit -a -m "提交消息"

# 修改最后一次提交
git commit --amend
```

## 远程仓库管理

### 添加远程仓库

```bash
git remote add <远程名称> <远程仓库URL>
```

### 查看远程仓库

```bash
# 列出所有远程仓库
git remote -v

# 查看远程仓库详细信息
git remote show <远程名称>
```

### 推送更改

```bash
# 推送到默认远程仓库
git push

# 推送到特定远程和分支
git push <远程名称> <分支名>

# 强制推送（谨慎使用）
git push -f
```

### 拉取更新

```bash
# 拉取并合并远程更改
git pull

# 拉取但不自动合并
git fetch
```

## 分支管理

### 创建分支

```bash
# 创建新分支
git branch <分支名>

# 创建并切换到新分支
git checkout -b <分支名>
```

### 切换分支

```bash
git checkout <分支名>
```

### 合并分支

```bash
git merge <分支名>
```

### 删除分支

```bash
# 删除本地分支
git branch -d <分支名>

# 删除远程分支
git push <远程名称> --delete <分支名>
```

## 标签管理

### 创建标签

```bash
# 创建轻量标签
git tag <标签名>

# 创建带注释的标签
git tag -a <标签名> -m "标签消息"
```

### 推送标签

```bash
# 推送单个标签
git push <远程名称> <标签名>

# 推送所有标签
git push <远程名称> --tags
```

## 仓库配置

### 查看配置

```bash
# 查看所有配置
git config --list

# 查看特定配置
git config <配置项>
```

### 设置配置

```bash
# 设置用户名
git config user.name "你的名字"

# 设置邮箱
git config user.email "你的邮箱"

# 全局配置（对所有仓库生效）
git config --global <配置项> <值>
```

## 高级操作

### 重写历史

```bash
# 交互式重写提交历史
git rebase -i <基准提交>

# 修改多个提交的作者信息
git filter-branch --env-filter '
    OLD_EMAIL="旧邮箱"
    CORRECT_NAME="正确名字"
    CORRECT_EMAIL="正确邮箱"
    if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]; then
        export GIT_COMMITTER_NAME="$CORRECT_NAME"
        export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
    fi
    if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]; then
        export GIT_AUTHOR_NAME="$CORRECT_NAME"
        export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
    fi
' --tag-name-filter cat -- --branches --tags
```

### 子模块管理

```bash
# 添加子模块
git submodule add <仓库URL> <路径>

# 初始化和更新子模块
git submodule update --init --recursive
```

## 仓库维护

### 清理仓库

```bash
# 清理未跟踪文件
git clean -fd

# 交互式清理
git clean -i
```

### 垃圾回收

```bash
# 压缩仓库并清理无用对象
git gc --aggressive
```

## 常见问题解决

### 撤销更改

```bash
# 撤销工作区更改
git checkout -- <文件>

# 撤销暂存区更改
git reset HEAD <文件>

# 重置到特定提交
git reset --hard <提交哈希>
```

### 恢复删除的分支

```bash
# 查找删除的分支的提交哈希
git reflog

# 从提交哈希恢复分支
git branch <分支名> <提交哈希>
```

## 最佳实践

!> **重要提示**：遵循这些最佳实践可以保持仓库健康

1. **保持提交原子性**：每个提交应该只包含一个逻辑更改
2. **编写有意义的提交消息**：使用约定式提交格式
3. **定期同步远程仓库**：避免合并冲突
4. **使用分支策略**：如 Git Flow 或 GitHub Flow
5. **定期清理仓库**：删除无用分支和标签

## 总结

Git 仓库是版本控制的核心，掌握仓库的创建、配置和管理是高效使用 Git 的基础。通过合理使用分支、标签和远程仓库，可以构建高效的团队协作工作流。

> 提示：随着项目规模增长，考虑使用 `.gitignore` 文件排除不需要版本控制的文件，保持仓库整洁。