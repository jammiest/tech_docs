# GIT

## 参考文档

- 中文：<https://git-scm.com/book/zh/v2>
- 英文：<https://git-scm.com/book/en/v2>


## Git 提交类型

在使用 Git 进行代码管理时，规范的提交信息（commit message）可以提高代码的可读性和维护性。以下是常见的 Git 提交类型及其含义：

## 常见提交类型

- `init`: 初始化项目。
- `feat`: 新功能（feature），用于提交新功能。例如：feat: 增加用户注册功能
- `fix`: 修复 bug，用于提交 bug 修复。例如：fix: 修复登录页面崩溃的问题
- `docs`: 文档变更，用于提交仅文档相关的修改。例如：docs: 更新 README 文件
- `style`: 代码风格变动（不影响代码逻辑），用于提交仅格式化、标点符号、空白等不影响代码运行的变更。例如：style: 删除多余的空行
- `refactor`: 代码重构（既不是新增功能也不是修复 bug 的代码更改），用于提交代码重构。例如：refactor: 重构用户验证逻辑
- `perf`: 性能优化，用于提交提升性能的代码修改。例如：perf: 优化图片加载速度
- `test`: 添加或修改测试，用于提交测试相关的内容。例如：test: 增加用户模块的单元测试
- `chore`: 杂项（构建过程或辅助工具的变动），用于提交构建过程、辅助工具等相关的内容修改。例如：chore: 更新依赖库
- `build`: 构建系统或外部依赖项的变更，用于提交影响构建系统的更改。例如：build: 升级 webpack 到版本
- `ci`: 持续集成配置的变更，用于提交 CI 配置文件和脚本的修改。例如：ci: 修改 GitHub Actions 配置文件
- `revert`: 回滚，用于提交回滚之前的提交。例如：revert: 回滚 feat: 增加用户注册功能

## 提交信息格式

规范的提交信息格式如下：

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```
- `type`: 提交的类别，必需字段。
- `scope`: 提交影响的范围，可选字段。
- `subject`: 提交目的的简短描述，必需字段，不超过 50 个字符。
- `body`: 提交的详细描述，可选字段。
- `footer`: 仅用于不兼容变动或关闭 Issue，可选字段

> 通过使用规范的提交信息，可以让项目更加模块化、易于维护和理解，同时也便于自动化工具（如发布工具或 Changelog 生成器）解析和处理提交记录


## 常见Git命令一览

```shell
## 基础操作 -----------
git init                # 初始化仓库

git clone <仓库URL>     # 克隆远程仓库

git status

git add <文件名>        # 添加单个文件
git add .              # 添加所有修改
git add -A             # 添加所有变更（包括删除）

git commit -m "提交信息"
git commit -am "提交信息"  # 跳过`git add`（仅对已跟踪文件有效）

git reset HEAD <文件>    # 取消暂存
git checkout -- <文件>   # 丢弃工作区修改（危险！不可恢复）
git commit --amend       # 修改最后一次提交

git branch              # 本地分支
git branch -a           # 所有分支（包括远程）

git branch <分支名>      # 创建分支
git checkout <分支名>    # 切换分支
git checkout -b <分支名> # 创建并切换

git merge <分支名>       # 合并到当前分支
git rebase <分支名>      # 变基（谨慎使用）

git branch -d <分支名>   # 删除本地分支
git push origin --delete <远程分支名>  # 删除远程分支

## 远程仓库 -----------
git remote add origin <仓库URL>         #关联远程仓库
git push -u origin <分支名>         # 首次推送
git push                             # 后续推送

git pull                     # 拉取并合并（= fetch + merge）
git fetch                    # 仅拉取不合并

git remote -v                # 查看远程仓库地址

## 日志与对比 ----------
git log                      # 详细日志
git log --oneline --graph    # 简洁可视化日志

git diff                     # 工作区与暂存区差异
git diff --staged            # 暂存区与最新提交差异
git diff <分支1> <分支2>      # 分支间差异

git stash                    # 保存工作现场
git stash pop                # 恢复工作现场

git tag v1.0                 # 创建标签
git push origin v1.0         # 推送标签到远程

git reset --hard <commit-id> # 强制回退（危险！）
git revert <commit-id>       # 生成新提交以撤销某次提交

git submodule add <仓库URL>   # 添加子模块
git submodule update --init   # 初始化子模块

## 实用技巧 ----------
git config --global alias.co checkout  # 别名配置：用`git co`代替`git checkout`
```