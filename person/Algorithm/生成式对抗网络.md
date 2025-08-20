# 生成式对抗网络（GAN）的数学原理

## 1. 基本框架

生成式对抗网络（Generative Adversarial Network, GAN）由两个主要组件构成：

- **生成器（Generator）**：$G(z;\theta_g)$，将随机噪声$z$映射到数据空间
- **判别器（Discriminator）**：$D(x;\theta_d)$，判断输入$x$是真实数据还是生成数据

目标函数为极小极大博弈：

$$
\min_G \max_D V(D,G) = \mathbb{E}_{x\sim p_{\text{data}}(x)}[\log D(x)] + \mathbb{E}_{z\sim p_z(z)}[\log(1-D(G(z)))]
$$

## 2. 优化目标分解

### 2.1 判别器目标
固定$G$，优化$D$：

$$
\max_D V(D,G) = \mathbb{E}_{x\sim p_{\text{data}}}[\log D(x)] + \mathbb{E}_{z\sim p_z}[\log(1-D(G(z)))]
$$

### 2.2 生成器目标
固定$D$，优化$G$：

$$
\min_G V(D,G) = \mathbb{E}_{z\sim p_z}[\log(1-D(G(z)))]
$$

实践中常用改进目标：

$$
\max_G \mathbb{E}_{z\sim p_z}[\log D(G(z))]
$$

## 3. 最优解分析

### 3.1 最优判别器
对于固定生成器$G$，最优判别器：

$$
D^*(x) = \frac{p_{\text{data}}(x)}{p_{\text{data}}(x) + p_g(x)}
$$

### 3.2 全局最优解
当且仅当$p_g = p_{\text{data}}$时达到纳什均衡，此时：

$$
D^*(x) = \frac{1}{2}, \quad V(D^*,G) = -\log 4
$$

## 4. 训练算法

### 4.1 基本训练流程
```python
for epoch in range(num_epochs):
    # 更新判别器
    real_data = next(data_loader)
    z = torch.randn(batch_size, latent_dim)
    fake_data = generator(z)
    
    d_loss = -torch.mean(torch.log(discriminator(real_data)) + 
                         torch.log(1 - discriminator(fake_data)))
    
    # 更新生成器
    z = torch.randn(batch_size, latent_dim)
    g_loss = -torch.mean(torch.log(discriminator(generator(z))))
```

### 4.2 改进训练技巧
- **特征匹配**：生成器匹配真实数据的统计特征
- **小批量判别**：防止模式崩溃
- **标签平滑**：防止判别器过度自信
- **历史平均**：稳定训练

## 5. GAN变体

### 5.1 DCGAN（深度卷积GAN）
- 使用卷积神经网络
- 架构设计准则：
  - 使用步长卷积代替池化层
  - 在生成器和判别器中使用批归一化
  - 去除全连接层
  - 使用ReLU（生成器）和LeakyReLU（判别器）

### 5.2 WGAN（Wasserstein GAN）
使用Wasserstein距离：

$$
W(p_{\text{data}}, p_g) = \sup_{\|D\|_L \leq 1} \mathbb{E}_{x\sim p_{\text{data}}}[D(x)] - \mathbb{E}_{z\sim p_z}[D(G(z))]
$$

关键改进：
- 权重裁剪保证Lipschitz连续性
- 使用线性激活值（去除了sigmoid）
- 损失函数直接反映生成质量

### 5.3 Conditional GAN
加入条件信息$y$：

$$
\min_G \max_D V(D,G) = \mathbb{E}_{x\sim p_{\text{data}}}[\log D(x|y)] + \mathbb{E}_{z\sim p_z}[\log(1-D(G(z|y)))]
$$

## 6. 评估指标

### 6.1 Inception Score (IS)
$$
IS(G) = \exp(\mathbb{E}_{x\sim p_g} KL(p(y|x) \| p(y)))
$$

### 6.2 Fréchet Inception Distance (FID)
计算真实数据和生成数据在特征空间的统计距离：

$$
FID = \|\mu_r - \mu_g\|^2 + Tr(\Sigma_r + \Sigma_g - 2(\Sigma_r \Sigma_g)^{1/2})
$$

## 7. 数学推导

### 7.1 判别器梯度
$$
\nabla_{\theta_d} V = \mathbb{E}_{x\sim p_{\text{data}}}\left[\frac{\nabla_{\theta_d} D(x)}{D(x)}\right] + \mathbb{E}_{z\sim p_z}\left[\frac{-\nabla_{\theta_d} D(G(z))}{1-D(G(z))}\right]
$$

### 7.2 生成器梯度
$$
\nabla_{\theta_g} V = \mathbb{E}_{z\sim p_z}\left[\frac{-\nabla_{\theta_g} D(G(z))}{1-D(G(z))}\right]
$$

## 8. 理论分析

### 8.1 目标函数与JS散度的关系
原始GAN目标等价于最小化：

$$
2JS(p_{\text{data}}\|p_g) - \log 4
$$

### 8.2 模式崩溃问题
当生成器只生成部分模式时，判别器无法提供有效梯度

### 8.3 梯度消失
当判别器过于优秀时，生成器梯度趋近于零

GAN通过这种对抗训练机制，能够学习到复杂的数据分布，在图像生成、风格迁移等领域取得了显著成功。