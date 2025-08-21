# 扩散模型（Diffusion Models）的数学原理

扩散模型是近年来兴起的一类生成模型，通过逐步添加和去除噪声的过程实现数据生成。以下是扩散模型的完整数学框架：

## 1. 基础概念

扩散模型包含两个核心过程：
1. **前向过程（扩散过程）**：逐步向数据添加噪声
2. **反向过程（去噪过程）**：学习从噪声中恢复原始数据

## 2. 前向扩散过程

### 2.1 定义
前向过程是一个固定的马尔可夫链，定义如下：

$$
q(x_{1:T}|x_0) = \prod_{t=1}^T q(x_t|x_{t-1})
$$

其中每一步的扩散：

$$
q(x_t|x_{t-1}) = \mathcal{N}(x_t; \sqrt{1-\beta_t}x_{t-1}, \beta_t\mathbf{I})
$$

### 2.2 重参数化
可以直接从$x_0$计算任意时刻$t$的$x_t$：

$$
x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon
$$

其中：
- $\alpha_t = 1-\beta_t$
- $\bar{\alpha}_t = \prod_{s=1}^t \alpha_s$
- $\epsilon \sim \mathcal{N}(0,\mathbf{I})$

## 3. 反向去噪过程

### 3.1 定义
反向过程是一个参数化的马尔可夫链：

$$
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^T p_\theta(x_{t-1}|x_t)
$$

其中：

$$
p_\theta(x_{t-1}|x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t,t), \Sigma_\theta(x_t,t))
$$

### 3.2 目标函数
优化变分下界（ELBO）：

$$
\mathcal{L} = \mathbb{E}_q \left[ -\log p_\theta(x_0) + \sum_{t=2}^T D_{KL}(q(x_{t-1}|x_t,x_0) \| p_\theta(x_{t-1}|x_t)) + D_{KL}(q(x_T|x_0) \| p(x_T)) \right]
$$

### 3.3 简化目标
实际训练中优化噪声预测：

$$
\mathcal{L}_{\text{simple}} = \mathbb{E}_{t,x_0,\epsilon} \left[ \|\epsilon - \epsilon_\theta(x_t,t)\|^2 \right]
$$

## 4. 采样算法

### 4.1 标准采样
1. 采样$x_T \sim \mathcal{N}(0,\mathbf{I})$
2. 对于$t=T,...,1$:
   - 预测噪声$\epsilon_\theta(x_t,t)$
   - 计算$x_{t-1}$:
     $$
     x_{t-1} = \frac{1}{\sqrt{\alpha_t}} \left( x_t - \frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}} \epsilon_\theta(x_t,t) \right) + \sigma_t z
     $$
     其中$z \sim \mathcal{N}(0,\mathbf{I})$

### 4.2 DDIM采样
更高效的采样方式：

$$
x_{t-1} = \sqrt{\bar{\alpha}_{t-1}} \left( \frac{x_t - \sqrt{1-\bar{\alpha}_t}\epsilon_\theta(x_t,t)}{\sqrt{\bar{\alpha}_t}} \right) + \sqrt{1-\bar{\alpha}_{t-1}-\sigma_t^2}\epsilon_\theta(x_t,t) + \sigma_t z
$$

## 5. 噪声调度策略

### 5.1 线性调度
$$
\beta_t = \beta_{\text{min}} + (\beta_{\text{max}}-\beta_{\text{min}})\frac{t}{T-1}
$$

### 5.2 余弦调度
$$
\bar{\alpha}_t = \frac{\cos(\pi t/2T + s)}{1+s}, \quad s=0.008
$$

## 6. 条件扩散模型

对于条件生成$p(x|y)$，修改噪声预测网络：

$$
\epsilon_\theta(x_t,t,y)
$$

常见条件控制方法：
- Classifier guidance
- Classifier-free guidance

## 7. 改进与变体

### 7.1 Latent Diffusion Models
在隐空间进行扩散：
1. 编码器：$z = E(x)$
2. 在$z$空间进行扩散
3. 解码器：$x = D(z)$

### 7.2 Stable Diffusion
结合CLIP文本编码器的文本到图像扩散模型

## 8. 数学推导

### 8.1 后验分布
$$
q(x_{t-1}|x_t,x_0) = \mathcal{N}(x_{t-1}; \tilde{\mu}_t(x_t,x_0), \tilde{\beta}_t\mathbf{I})
$$

其中：
$$
\tilde{\mu}_t(x_t,x_0) = \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}x_0 + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}x_t
$$
$$
\tilde{\beta}_t = \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\beta_t
$$

### 8.2 参数化均值
学习目标可以参数化为：

$$
\mu_\theta(x_t,t) = \frac{1}{\sqrt{\alpha_t}} \left( x_t - \frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}} \epsilon_\theta(x_t,t) \right)
$$

扩散模型通过这种渐进式的噪声添加和去除机制，结合深度学习强大的函数逼近能力，能够生成高质量的样本，已成为当前最先进的生成模型之一。