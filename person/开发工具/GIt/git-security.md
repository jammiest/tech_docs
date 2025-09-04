# Git 安全双翼：详解 SSH 与 GPG 的使命与配置

## 🛡️ 安全概述

在 Git 协作生态中，**SSH** 和 **GPG** 如同安全的双翼，分别承担着不同的安全使命：

| 安全组件 | 使命担当 | 应用场景 |
|---------|---------|---------|
| **SSH** | 身份认证与传输加密 | 远程仓库访问、数据传输 |
| **GPG** | 代码签名与完整性验证 | 提交签名、标签签名、来源验证 |

## 🔐 SSH：安全通信的守护者

### 使命解析
SSH（Secure Shell）为 Git 提供：
- **身份认证**：证明你是仓库的合法访问者
- **传输加密**：保护代码在传输过程中不被窃听
- **完整性保护**：确保数据在传输中不被篡改

### 配置实战

#### 1. 密钥生成策略

**ED25519（推荐）**：
```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_github
```

**RSA 4096（兼容性强）**：
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_github
```

!> **安全提示**：为不同平台使用不同密钥，避免一钥多用

#### 2. 多平台配置

创建 `~/.ssh/config`：

```bash
# GitHub 配置
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
    AddKeysToAgent yes

# GitLab 配置  
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab
    IdentitiesOnly yes

# 公司内部仓库
Host internal.git.company.com
    HostName git.company.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    Port 2222
```

#### 3. 代理管理自动化

在 `~/.zshrc` 或 `~/.bashrc` 中添加：

```bash
# SSH 代理自动启动
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)" > /dev/null
    ssh-add ~/.ssh/id_ed25519_github 2>/dev/null
    ssh-add ~/.ssh/id_ed25519_gitlab 2>/dev/null
fi
```

## 📝 GPG：代码完整性的卫士

### 使命解析
GPG（GNU Privacy Guard）为 Git 提供：
- **身份证明**：确保提交来自可信的开发者
- **完整性验证**：保证代码在提交后未被修改
- **不可否认性**：签名者无法否认自己的提交

### 配置实战

#### 1. 密钥生成最佳实践

```bash
# 生成主密钥（仅用于签名）
gpg --full-generate-key
```
选择选项：
- 密钥类型：`RSA and RSA`
- 密钥长度：`4096`
- 过期时间：`2y`（建议设置过期时间）
- 用户信息：真实姓名和邮箱

#### 2. 子密钥管理（推荐）

```bash
# 生成签名子密钥
gpg --edit-key YOUR_KEY_ID
addkey
# 选择密钥类型：RSA (sign only)
# 设置长度：4096
# 设置过期时间：1y
save
```

#### 3. Git 集成配置

```bash
# 告诉 Git 使用哪个密钥
git config --global user.signingkey YOUR_KEY_ID

# 全局开启提交签名
git config --global commit.gpgsign true

# 配置 GPG 程序路径（如需要）
git config --global gpg.program $(which gpg)

# 设置环境变量（解决终端问题）
export GPG_TTY=$(tty)
```

#### 4. 多设备同步方案

```bash
# 导出密钥（在源设备）
gpg --export-secret-keys --armor YOUR_KEY_ID > private.key
gpg --export --armor YOUR_KEY_ID > public.key

# 导入密钥（在目标设备）
gpg --import public.key
gpg --import private.key

# 信任密钥
gpg --edit-key YOUR_KEY_ID
trust
# 选择信任级别：5 = I trust ultimately
save
```

## 🚀 实战演练

### 场景一：新设备初始化

```bash
# 1. 生成 SSH 密钥
ssh-keygen -t ed25519 -C "dev@company.com" -f ~/.ssh/id_ed25519_work

# 2. 添加 SSH 配置
echo 'Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes' >> ~/.ssh/config

# 3. 导入 GPG 密钥
gpg --import private.key
gpg --import public.key

# 4. 配置 Git
git config --global user.name "Your Name"
git config --global user.email "dev@company.com"
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
```

### 场景二：提交签名验证

```bash
# 创建签名提交
git commit -S -m "feat: add security features"

# 验证提交签名
git log --show-signature -1

# 验证标签签名
git tag -s v1.0.0 -m "Release version 1.0.0"
git tag -v v1.0.0
```

## 🛠️ 故障排除

### SSH 常见问题

**问题：Permission denied (publickey)**
```bash
# 解决方案：
ssh -T git@github.com -v  # 查看详细错误
ssh-add -l               # 检查密钥是否加载
chmod 600 ~/.ssh/*       # 检查文件权限
```

### GPG 常见问题

**问题：gpg: signing failed: Inappropriate ioctl for device**
```bash
# 解决方案：
export GPG_TTY=$(tty)
echo "pinentry-program /usr/bin/pinentry-tty" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

## 📊 安全检查清单

- [ ] SSH 密钥使用 ED25519 或 RSA 4096
- [ ] 为不同服务使用不同密钥
- [ ] SSH 密钥设置了强密码
- [ ] GPG 密钥设置了合理的过期时间
- [ ] 提交签名全局开启
- [ ] 备份了 SSH 和 GPG 密钥
- [ ] 配置了密钥吊销证书

## 🔗 扩展资源

- https://docs.github.com/en/authentication/connecting-to-github-with-ssh
- https://docs.github.com/en/authentication/managing-commit-signature-verification
- https://riseup.net/en/security/message-security/openpgp/best-practices

&gt; **提示**：安全是一个持续的过程。定期轮换密钥、检查权限、更新配置，才能确保你的代码仓库始终处于安全保护之下。

通过正确配置和使用 SSH 与 GPG，你不仅保护了自己的代码，也为整个开源社区的安全做出了贡献。🛡️