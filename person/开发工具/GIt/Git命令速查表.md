# Git 命令速查表

## 📋 配置与帮助
```bash
# 用户配置
git config --global user.name "Your Name"          # 设置全局用户名
git config --global user.email "your@email.com"    # 设置全局邮箱
git config --list                                  # 查看所有配置
git config --global core.editor "code --wait"      # 设置默认编辑器

# 别名设置
git config --global alias.co checkout              # 创建别名
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status

# 帮助信息
git help <command>                                 # 查看命令帮助
git <command> --help                               # 查看命令帮助
```

## 🏗️ 仓库操作
```bash
# 初始化与克隆
git init                                          # 初始化新仓库
git clone <url>                                   # 克隆远程仓库
git clone <url> <directory>                       # 克隆到指定目录
git clone --depth 1 <url>                         # 浅克隆（只下载最新版本）

# 远程仓库管理
git remote -v                                     # 查看远程仓库
git remote add <name> <url>                       # 添加远程仓库
git remote remove <name>                          # 删除远程仓库
git remote rename <old> <new>                     # 重命名远程仓库
git remote show origin                            # 查看远程仓库详情
```

## 📊 状态与日志
```bash
# 状态查看
git status                                        # 查看工作区状态
git status -s                                     # 简洁状态显示
git status --ignored                              # 显示被忽略的文件

# 提交历史
git log                                           # 查看提交历史
git log --oneline                                 # 简洁提交历史
git log --graph --oneline --all                   # 图形化提交历史
git log -p                                        # 显示详细差异
git log --stat                                    # 显示统计信息
git log --since="2023-01-01"                      # 按时间筛选
git log --author="name"                           # 按作者筛选
git log --grep="keyword"                          # 搜索提交信息
git log -n 5                                      # 显示最近5条提交
```

## 📁 文件操作
```bash
# 添加文件
git add <file>                                    # 添加特定文件
git add .                                         # 添加所有修改
git add -A                                        # 添加所有变化
git add -u                                        # 添加已跟踪文件的修改
git add -p                                        # 交互式添加

# 提交更改
git commit -m "commit message"                    # 提交更改
git commit -am "commit message"                   # 添加并提交已跟踪文件
git commit --amend                                # 修改最新提交
git commit --amend --no-edit                      # 修改提交但不改信息

# 撤销操作
git restore <file>                                # 撤销工作区修改（Git 2.23+）
git restore --staged <file>                       # 撤销暂存区修改
git checkout -- <file>                            # 撤销工作区修改（旧版本）
git reset HEAD <file>                             # 撤销暂存区修改（旧版本）
git clean -fd                                     # 删除未跟踪文件
```

## 🌿 分支管理
```bash
# 分支操作
git branch                                        # 查看本地分支
git branch -a                                     # 查看所有分支（包括远程）
git branch -r                                     # 查看远程分支
git branch <branch-name>                          # 创建新分支
git checkout <branch-name>                        # 切换到分支
git checkout -b <branch-name>                     # 创建并切换到新分支
git switch <branch-name>                          # 切换到分支（Git 2.23+）
git switch -c <branch-name>                       # 创建并切换到新分支
git branch -d <branch-name>                       # 删除分支
git branch -D <branch-name>                       # 强制删除分支
git branch -m <new-name>                          # 重命名当前分支

# 分支合并
git merge <branch-name>                           # 合并分支
git merge --no-ff <branch-name>                   # 禁用快进合并
git merge --squash <branch-name>                  # 压缩合并
git mergetool                                     # 使用合并工具解决冲突

# 变基操作
git rebase <branch-name>                          # 变基分支
git rebase -i HEAD~3                              # 交互式变基
git rebase --continue                             # 继续变基
git rebase --skip                                 # 跳过当前提交
git rebase --abort                                # 中止变基
```

## 🔄 远程操作
```bash
# 推送与拉取
git push origin <branch-name>                     # 推送到远程分支
git push -u origin <branch-name>                  # 推送到远程并建立跟踪
git push --force                                  # 强制推送（谨慎使用）
git push --force-with-lease                       # 安全强制推送
git pull origin <branch-name>                     # 拉取远程分支
git pull --rebase                                 # 变基方式拉取
git fetch origin                                  # 获取远程更新
git fetch --prune                                 # 获取并清理远程分支

# 跟踪分支
git branch --set-upstream-to=origin/<branch> <branch> # 设置跟踪分支
git push -d origin <branch-name>                  # 删除远程分支
```

## 🏷️ 标签管理
```bash
git tag                                           # 查看所有标签
git tag <tag-name>                                # 创建轻量标签
git tag -a <tag-name> -m "message"                # 创建附注标签
git tag -a v1.0 <commit-hash>                     # 给特定提交打标签
git show <tag-name>                               # 查看标签详情
git push origin <tag-name>                        # 推送标签到远程
git push origin --tags                            # 推送所有标签
git tag -d <tag-name>                             # 删除本地标签
git push origin --delete <tag-name>               # 删除远程标签
```

## 💾 暂存与储藏
```bash
git stash                                         # 储藏当前修改
git stash save "message"                          # 储藏并添加消息
git stash list                                    # 查看储藏列表
git stash apply                                   # 应用最新储藏
git stash apply stash@{n}                         # 应用指定储藏
git stash pop                                     # 应用并删除最新储藏
git stash drop stash@{n}                          # 删除指定储藏
git stash clear                                   # 清空所有储藏
git stash branch <branch-name>                    # 从储藏创建新分支
```

## 🔍 比较与差异
```bash
git diff                                          # 比较工作区和暂存区
git diff --staged                                 # 比较暂存区和最新提交
git diff HEAD                                     # 比较工作区和最新提交
git diff <commit1> <commit2>                      # 比较两个提交
git diff <branch1>..<branch2>                     # 比较两个分支
git diff --name-only                              # 只显示文件名
git diff --stat                                   # 显示统计信息
```

## ⏪ 撤销与重置
```bash
# 重置操作
git reset --soft HEAD^                            # 撤销提交，保留修改到暂存区
git reset --mixed HEAD^                           # 撤销提交，保留修改到工作区
git reset --hard HEAD^                            # 彻底撤销提交（谨慎使用）
git reset --hard <commit-hash>                    # 重置到指定提交
git reset HEAD~1                                  # 重置到上一个提交

# 撤销提交
git revert <commit-hash>                          # 创建新的提交来撤销指定提交
git revert HEAD                                   # 撤销最新提交
git revert --no-commit <commit-hash>              # 撤销但不自动提交
```

## 🍒 Cherry-pick
```bash
git cherry-pick <commit-hash>                     # 应用指定提交到当前分支
git cherry-pick <commit1> <commit2>               # 应用多个提交
git cherry-pick <start-commit>^..<end-commit>     # 应用一系列提交
git cherry-pick --no-commit <commit-hash>         # 应用提交但不自动提交
git cherry-pick -x <commit-hash>                  # 在提交信息中添加来源信息
git cherry-pick --continue                        # 解决冲突后继续
git cherry-pick --abort                           # 中止cherry-pick操作
```

## 📦 子模块
```bash
git submodule add <url> <path>                    # 添加子模块
git submodule init                                # 初始化子模块
git submodule update                              # 更新子模块
git submodule update --init --recursive           # 初始化并递归更新
git submodule foreach <command>                   # 在每个子模块执行命令
```

## 🧹 清理与优化
```bash
git gc                                            # 垃圾回收优化仓库
git fsck                                          # 检查仓库完整性
git prune                                         # 清理不可达对象
git reflog                                        # 查看所有操作记录
git reflog expire --expire=now --all              # 清理reflog
```

## 🎯 高级操作
```bash
# 二分查找
git bisect start                                  # 开始二分查找
git bisect bad                                    # 标记当前为错误提交
git bisect good <commit>                          # 标记已知好的提交
git bisect reset                                  # 结束二分查找

# 工作区管理
git worktree add ../new-dir feature-branch        # 添加工作树
git worktree list                                 # 列出工作树
git worktree remove ../new-dir                    # 移除工作树

# 归档导出
git archive --format=zip HEAD > project.zip       # 导出项目为zip
git archive --format=tar.gz HEAD > project.tar.gz # 导出为tar.gz
```

## 📝 忽略文件
```bash
# .gitignore 示例
*.log
node_modules/
.DS_Store
.env
dist/
build/

# 检查忽略
git check-ignore -v <file>                        # 检查忽略规则
```

## 🔧 实用技巧
```bash
# 查看文件历史
git blame <file>                                  # 查看文件的修改历史
git show <commit>:<file>                          # 查看特定提交的文件内容

# 搜索
git grep "search term"                            # 在代码中搜索
git grep -n "search term"                         # 显示行号
git grep --name-only "search term"                # 只显示文件名

# 统计
git shortlog -sn                                  # 贡献者统计
git log --pretty=format:%h -n 10 | xargs git show --stat # 最近10个提交的统计
```

## 🚀 常用工作流
```bash
# 功能开发流程
git checkout -b feature/new-feature               # 创建功能分支
git add .                                         # 添加修改
git commit -m "实现新功能"                         # 提交更改
git push -u origin feature/new-feature            # 推送到远程

# 代码审查后
git checkout main                                 # 切换到主分支
git pull origin main                              # 拉取最新代码
git merge --no-ff feature/new-feature             # 合并功能分支
git push origin main                              # 推送到主分支
git branch -d feature/new-feature                 # 删除功能分支

# 紧急修复流程
git checkout main                                 
git pull origin main                              
git checkout -b hotfix/urgent-fix                 # 创建修复分支
# 进行修复...
git add .
git commit -m "修复紧急问题"
git checkout main
git merge hotfix/urgent-fix                       
git push origin main                              
```

这个速查表涵盖了Git的各个方面，从基础操作到高级技巧，适合日常开发中使用。建议收藏以备查阅！