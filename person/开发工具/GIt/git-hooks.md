# Git 钩子：自动化你的开发工作流的隐形引擎

> Git 钩子是 Git 版本控制系统中的强大功能，允许开发者在特定的 Git 操作前后自动执行自定义脚本，从而实现开发工作流的自动化。

## 什么是 Git 钩子？

Git 钩子（Git Hooks）是存储在 Git 仓库 `.git/hooks` 目录下的可执行脚本。当 Git 执行特定操作（如提交、推送、合并等）时，会自动触发相应的钩子脚本。

### 钩子类型

Git 提供了两种类型的钩子：

- **客户端钩子**：在本地仓库操作时触发
- **服务器端钩子**：在远程仓库操作时触发

## 常用客户端钩子

### pre-commit
> 在提交消息被输入前运行，用于检查代码质量

```bash
#!/bin/sh
# 运行代码检查
npm run lint
# 运行测试
npm test
```

### prepare-commit-msg
> 在默认提交消息准备好后运行，可用于修改提交消息

### commit-msg
> 在用户输入提交消息后运行，用于验证提交消息格式

```bash
#!/bin/sh
# 检查提交消息是否符合约定式提交规范
if ! grep -qE "^(feat|fix|docs|style|refactor|test|chore): " "$1"; then
    echo "错误：提交消息必须以 feat/fix/docs/style/refactor/test/chore 开头"
    exit 1
fi
```

### post-commit
> 在提交完成后运行，可用于通知或其他后续操作

### pre-push
> 在推送到远程仓库前运行，用于确保代码质量

```bash
#!/bin/sh
# 确保所有测试通过
npm test
# 确保构建成功
npm run build
```

## 常用服务器端钩子

### pre-receive
> 在接收推送前运行，用于检查推送的提交

### update
> 类似于 pre-receive，但按分支运行

### post-receive
> 在接收推送后运行，可用于自动部署

```bash
#!/bin/sh
# 自动部署到生产环境
cd /path/to/production
git pull origin main
npm install
npm run build
```

## 实际应用示例

### 1. 自动代码检查

```bash
#!/bin/sh
# pre-commit 钩子示例
echo "运行代码检查..."

# 检查 ESLint
if ! npx eslint --ext .js,.ts,.vue src/; then
    echo "ESLint 检查失败，请修复错误后再提交"
    exit 1
fi

# 检查样式
if ! npx stylelint "**/*.{css,scss,vue}"; then
    echo "Stylelint 检查失败"
    exit 1
fi

echo "代码检查通过！"
```

### 2. 提交消息规范验证

```bash
#!/bin/sh
# commit-msg 钩子示例
COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# 验证提交消息格式
if ! echo "$COMMIT_MSG" | grep -qE "^(feat|fix|docs|style|refactor|test|chore|perf|build|ci|revert)(\(.+\))?: .{1,}"; then
    echo "错误：提交消息格式不正确"
    echo "格式: <类型>(<范围>): <描述>"
    echo "示例: feat(auth): 添加用户登录功能"
    exit 1
fi

# 验证描述长度
if [ ${#COMMIT_MSG} -gt 100 ]; then
    echo "错误：提交消息描述过长（最大100字符）"
    exit 1
fi
```

### 3. 自动运行测试

```bash
#!/bin/sh
# pre-push 钩子示例
echo "运行测试套件..."

if ! npm test; then
    echo "测试失败，请修复测试后再推送"
    exit 1
fi

echo "所有测试通过！"
```

## 安装和管理 Git 钩子

### 手动安装
```bash
# 将脚本复制到 .git/hooks 目录
cp pre-commit.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

### 使用工具管理
推荐使用以下工具管理 Git 钩子：

- **Husky** - 现代化的 Git 钩子管理工具
- **pre-commit** - Python 项目的钩子管理
- **Lefthook** - 快速灵活的 Git 钩子管理器

#### Husky 示例配置
```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
      "pre-push": "npm test"
    }
  }
}
```

## 最佳实践

!> **重要提示**：遵循这些最佳实践可以确保 Git 钩子的有效性和可靠性

1. **保持钩子轻量**：避免在钩子中执行耗时操作
2. **提供清晰的错误信息**：当钩子失败时，告诉用户如何修复
3. **支持跳过机制**：允许用户通过环境变量跳过某些检查
4. **版本控制钩子脚本**：将钩子脚本纳入版本控制
5. **测试你的钩子**：确保钩子脚本在各种情况下都能正常工作

## 跳过钩子

在某些情况下，可能需要跳过钩子检查：

```bash
# 使用 --no-verify 跳过 pre-commit 和 commit-msg 钩子
git commit -m "紧急修复" --no-verify

# 跳过 pre-push 钩子
git push --no-verify
```

## 常见问题解决

### 钩子不执行
- 检查脚本是否有执行权限：`chmod +x .git/hooks/hook-name`
- 检查脚本的 shebang（如 `#!/bin/sh`）是否正确

### 性能问题
- 避免在钩子中执行重型操作
- 考虑使用缓存或增量检查

## 总结

Git 钩子是提升开发工作流自动化水平的强大工具。通过合理配置和使用钩子，可以：

- ✅ 确保代码质量
- ✅ 规范提交消息
- ✅ 自动运行测试
- ✅ 实现自动部署
- ✅ 提高团队协作效率

> 提示：开始使用 Git 钩子时，建议从简单的检查开始，逐步增加更复杂的自动化流程。